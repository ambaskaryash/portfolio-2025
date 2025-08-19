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

          mkdir -p "${WEB_ROOT}"

          # Copy only HTML/CSS and common static assets
          find . -type f \\( \
            -name '*.html' -o -name '*.htm' -o -name '*.css' -o \
            -name '*.jpg'  -o -name '*.jpeg' -o -name '*.png' -o -name '*.gif' -o -name '*.svg' -o -name '*.webp' -o \
            -name '*.ico' \
          \\) -exec cp --parents -t "${WEB_ROOT}" {} +

          echo "Deployment finished to ${WEB_ROOT}"
        '''
      }
    }
  }

  post {
    success { echo "Success: cloned and deployed HTML/CSS to ${env.WEB_ROOT}" }
    failure { echo "Failed: check console output for details." }
  }
}
