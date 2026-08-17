// building and deploying python based app in docker using jenkins pipeline
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

        stage('Docker Hub Push & Deploy') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "===== Docker Hub Login ====="

                        echo "$DOCKER_PASSWORD" | docker login \
                          -u "$DOCKER_USERNAME" \
                          --password-stdin

                        echo "===== Tagging Image ====="

                        docker tag \
                          ${IMAGE_NAME}:${IMAGE_TAG} \
                          ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "===== Pushing Image To Docker Hub ====="

                        docker push \
                          ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "===== Pulling Image From Docker Hub ====="

                        docker pull \
                          ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "===== Stopping Old Container ====="

                        docker stop ${CONTAINER_NAME} || true

                        echo "===== Removing Old Container ====="

                        docker rm ${CONTAINER_NAME} || true

                        echo "===== Starting Container From Docker Hub ====="

                        docker run -d \
                          --name ${CONTAINER_NAME} \
                          -p 9000:7000 \
                          ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}

                        echo "===== Removing Old Local Images ====="

                        docker images "${IMAGE_NAME}" \
                          --format "{{.Repository}}:{{.Tag}}" \
                          | grep -v "^${IMAGE_NAME}:${IMAGE_TAG}$" \
                          | xargs -r docker rmi || true

                        echo "===== Running Container ====="

                        docker ps

                        echo "===== Docker Hub Logout ====="

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'Pipeline completed successfully!'
            echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo 'Image deployed from Docker Hub'
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