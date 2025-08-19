pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ambaskaryash/portfolio-2025.git'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    echo "Zipping portfolio files..."
                    zip -r portfolio.zip ./*
                '''
            }
        }

        stage('Deploy to Local Server') {
            steps {
                sh '''
                    echo "Deploying to /var/www/html..."
                    sudo cp -r * /var/www/html/
                '''
            }
        }
    }

    post {
        success {
            echo "Portfolio deployed successfully!"
        }
        failure {
            echo "Build failed. Please check logs."
        }
    }
}
