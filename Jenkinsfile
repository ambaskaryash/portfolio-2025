pipeline {
  agent any

  options {
    // Fail fast on errors inside sh blocks
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timeout(time: 15, unit: 'MINUTES')
  }

  environment {
    REPO_URL       = 'https://github.com/ambaskaryash/portfolio-2025.git'
    REPO_BRANCH    = 'main'
    WEB_ROOT       = '/var/www/html'
    BUILD_ZIP_NAME = 'portfolio.zip'
  }

  stages {
    stage('Prepare Linux Dependencies') {
      steps {
        sh '''
          set -euo pipefail
          echo "Checking required tools on Linux agent..."
          if ! command -v zip >/dev/null 2>&1; then
            echo "zip not found. Attempting to install..."
            # Detect common package managers
            if command -v apt-get >/dev/null 2>&1; then
              sudo apt-get update -y
              sudo apt-get install -y zip rsync
            elif command -v dnf >/dev/null 2>&1; then
              sudo dnf install -y zip rsync
            elif command -v yum >/dev/null 2>&1; then
              sudo yum install -y zip rsync
            elif command -v apk >/dev/null 2>&1; then
              sudo apk add --no-cache zip rsync
            elif command -v pacman >/dev/null 2>&1; then
              sudo pacman -Sy --noconfirm zip rsync
            else
              echo "No supported package manager found. Please pre-install zip and rsync." >&2
              exit 1
            fi
          else
            echo "zip found: $(command -v zip)"
          fi

          if ! command -v rsync >/dev/null 2>&1; then
            echo "rsync not found. Installing..."
            if command -v apt-get >/dev/null 2>&1; then
              sudo apt-get update -y
              sudo apt-get install -y rsync
            elif command -v dnf >/dev/null 2>&1; then
              sudo dnf install -y rsync
            elif command -v yum >/dev/null 2>&1; then
              sudo yum install -y rsync
            elif command -v apk >/dev/null 2>&1; then
              sudo apk add --no-cache rsync
            elif command -v pacman >/dev/null 2>&1; then
              sudo pacman -Sy --noconfirm rsync
            else
              echo "No supported package manager found. Please pre-install rsync." >&2
              exit 1
            fi
          fi
        '''
      }
    }

    stage('Clone Repository') {
      steps {
        checkout([$class: 'GitSCM',
          branches: [[name: "*/${env.REPO_BRANCH}"]],
          userRemoteConfigs: [[url: env.REPO_URL]],
          extensions: [[$class: 'CleanCheckout']]
        ])
      }
    }

    stage('Build Artifact') {
      steps {
        sh '''
          set -euo pipefail
          echo "Creating build artifact..."
          rm -f "${BUILD_ZIP_NAME}"
          # Exclude VCS and Jenkins files from the archive
          zip -r "${BUILD_ZIP_NAME}" ./ \
            -x ".git/*" \
               ".gitignore" \
               "Jenkinsfile" \
               "**/node_modules/*" \
               "**/.DS_Store"
          echo "Artifact created: ${BUILD_ZIP_NAME}"
          ls -lh "${BUILD_ZIP_NAME}"
        '''
        archiveArtifacts artifacts: "${BUILD_ZIP_NAME}", fingerprint: true
      }
    }

    stage('Deploy to Local Server') {
      when {
        expression { return fileExists(env.BUILD_ZIP_NAME) }
      }
      steps {
        sh '''
          set -euo pipefail
          echo "Preparing deploy directory: ${WEB_ROOT}"

          # Ensure web root exists and is writable by the current user or use sudo with controlled permissions.
          if [ ! -d "${WEB_ROOT}" ]; then
            echo "Web root does not exist. Creating..."
            sudo mkdir -p "${WEB_ROOT}"
          fi

          # Option A (Recommended): Use rsync to deploy excluding VCS and build files.
          echo "Deploying files with rsync to ${WEB_ROOT}..."
          rsync -av --delete \
            --exclude ".git/" \
            --exclude "Jenkinsfile" \
            --exclude "${BUILD_ZIP_NAME}" \
            --exclude "node_modules/" \
            ./ "${WEB_ROOT}/"

          # Optionally set ownership to web server user/group (adjust as needed, e.g., www-data:nogroup, apache:apache, nginx:nginx)
          if id -u www-data >/dev/null 2>&1; then
            echo "Setting ownership to www-data:www-data..."
            sudo chown -R www-data:www-data "${WEB_ROOT}"
          elif id -u nginx >/dev/null 2>&1; then
            echo "Setting ownership to nginx:nginx..."
            sudo chown -R nginx:nginx "${WEB_ROOT}"
          fi

          # Set safe permissions
          sudo find "${WEB_ROOT}" -type d -exec chmod 755 {} +
          sudo find "${WEB_ROOT}" -type f -exec chmod 644 {} +

          echo "Deployment completed to ${WEB_ROOT}"
        '''
      }
    }
  }

  post {
    success {
      echo "Portfolio deployed successfully on Linux!"
    }
    failure {
      echo "Build failed. Please check logs."
    }
    always {
      // Optional: clean workspace for Linux agents
      cleanWs(deleteDirs: true, notFailBuild: true)
    }
  }
}
