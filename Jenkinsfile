pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-devsecops-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
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
                    echo "Building image: ${IMAGE_NAME}:${IMAGE_TAG}"

                    docker build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} \
                      .

                    echo "===== Image Created ====="
                    docker images ${IMAGE_NAME}
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    echo "===== Trivy Security Scan ====="
                    echo "Scanning: ${IMAGE_NAME}:${IMAGE_TAG}"

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
                      -p 8000:7000 \
                      ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Running Container ====="
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'Pipeline completed successfully!'
            echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo 'Application: http://localhost:8000'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'Pipeline FAILED!'
            echo '======================================'
        }
    }
}