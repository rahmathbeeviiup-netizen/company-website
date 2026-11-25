pipeline {
    agent {
        label 'build-server-agent'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t company-website:latest .
                '''
            }
        }

        stage('Verify Image') {
            steps {
                sh '''
                    echo "Verifying Docker image..."
                    docker images
                    docker image inspect company-website:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image built successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
