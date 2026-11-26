pipeline {
    agent {
        label 'build-server-agent'
    }

    environment {
        DOCKER_IMAGE = 'company-website'
        DOCKER_TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/dilnamusthafa81-coder/Company-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Verify Image') {
            steps {
                sh '''
                    docker images
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credential',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} \
                            $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker push $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            agent {
                label 'build-server-agent'
            }

            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credential',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker pull $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker stop company-website || true
                        docker rm company-website || true

                        docker run -d \
                            --name company-website \
                            -p 9090:80 \
                            $DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }
    }
}
