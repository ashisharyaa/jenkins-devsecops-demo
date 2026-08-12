pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-devsecops-demo"
        IMAGE_TAG  = "1.0"
        CONTAINER_NAME = "devsecops-app"
    }

    stages {

        stage('Environment') {
            steps {
                sh '''
                    echo "===== Environment ====="
                    echo "User: $(whoami)"
                    echo "Working Directory: $(pwd)"

                    echo "===== Docker ====="
                    docker --version

                    echo "===== Trivy ====="
                    trivy --version
                '''
            }
        }

        stage('Checkout') {
            steps {
                sh '''
                    echo "===== Checkout ====="
                    ls -la
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    echo "===== Docker Build ====="

                    docker build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} \
                      .
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "===== Trivy Security Scan ====="

                    trivy image \
                      --scanners vuln \
                      --timeout 10m \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Run') {
            steps {
                sh '''
                    echo "===== Deploy Application ====="

                    echo "Stopping old container..."
                    docker stop ${CONTAINER_NAME} || true

                    echo "Removing old container..."
                    docker rm ${CONTAINER_NAME} || true

                    echo "Starting new container..."
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 7000:7000 \
                      ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Running Containers ====="
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Application is available on http://localhost:5000'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}