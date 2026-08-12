// devsecops
pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-devsecops-demo"
        IMAGE_TAG  = "1.0"
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
                    echo "Source code checked out successfully"
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
    }
}