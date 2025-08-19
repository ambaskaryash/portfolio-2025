pipeline {
  agent any

  environment {
    REPO_URL    = 'https://github.com/ambaskaryash/portfolio-2025.git'
    REPO_BRANCH = 'main'
    WEB_ROOT    = '/var/www/html'
  }

  stages {
    stage('Clone') {
      steps {
        git branch: "${env.REPO_BRANCH}", url: "${env.REPO_URL}"
        echo "Repository cloned."
      }
    }

    stage('Deploy') {
      steps {
        sh '''
          set -e
          echo "Deploying static files to ${WEB_ROOT}..."

          # Create target directory if missing (may require sudo depending on permissions)
          if [ ! -d "${WEB_ROOT}" ]; then
            mkdir -p "${WEB_ROOT}" || sudo mkdir -p "${WEB_ROOT}"
          fi

          # Copy only common static web files: html, css, images, fonts, and minimal assets
          # Adjust patterns as needed for your repo structure.
          rsync -av --delete \
            --include '*/' \
            --include '*.html' \
            --include '*.htm' \
            --include '*.css' \
            --include '*.jpg' --include '*.jpeg' --include '*.png' --include '*.gif' --include '*.svg' --include '*.webp' \
            --include '*.ico' \
            --include 'fonts/***' \
            --exclude '.git/' \
            --exclude 'Jenkinsfile' \
            --exclude '*' \
            ./ "${WEB_ROOT}/" || sudo rsync -av --delete \
              --include '*/' \
              --include '*.html' \
              --include '*.htm' \
              --include '*.css' \
              --include '*.jpg' --include '*.jpeg' --include '*.png' --include '*.gif' --include '*.svg' --include '*.webp' \
              --include '*.ico' \
              --include 'fonts/***' \
              --exclude '.git/' \
              --exclude 'Jenkinsfile' \
              --exclude '*' \
              ./ "${WEB_ROOT}/"

          echo "Deployment finished."
        '''
      }
    }
  }

  post {
    success {
      echo "Success: cloned and deployed HTML/CSS to ${env.WEB_ROOT}"
    }
    failure {
      echo "Failed: check console output for details."
    }
  }
}
