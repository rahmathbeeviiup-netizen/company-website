
pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_IMAGE = 'company-website'
        DOCKER_TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            agent {
                label 'build-server-agent'
            }
            steps {
                git branch: 'main',
                    url: 'https://github.com/rahmathbeeviiup-netizen/company-website.git'
            }
        }

        stage('Check Docker Permissions') {
            agent {
                label 'build-server-agent'
            }
            steps {
                sh '''
                    echo "===== USER ====="
                    whoami

                    echo "===== GROUPS ====="
                    id

                    echo "===== DOCKER SOCKET ====="
                    ls -l /var/run/docker.sock

                    echo "===== DOCKER TEST ====="
                    docker ps
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                label 'build-server-agent'
            }
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                '''
            }
        }

        stage('Verify Image') {
            agent {
                label 'build-server-agent'
            }
            steps {
                sh '''
                    docker images ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Push to Docker Hub') {
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
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} \
                            "$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}"

                        docker push "$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Server') {
            agent {
                label 'docker-deploy-agent'
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
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker pull "$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}"

                        docker stop company-website || true
                        docker rm company-website || true

                        docker run -d \
                            --name company-website \
                            --restart unless-stopped \
                            -p 9090:80 \
                            "$DOCKER_USER/${DOCKER_IMAGE}:${DOCKER_TAG}"

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
