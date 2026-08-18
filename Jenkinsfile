
pipeline {
    agent any

    environment {
        IMAGE_NAME = "jenkins-devsecops-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "devsecops-app"

        DOCKERHUB = credentials('dockerhub-credentials')
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
                    git branch --show-current
                    git log -1 --oneline
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

        stage('Trivy Security Gate') {
            steps {
                sh '''
                    echo "===== Trivy Security Scan ====="
                    echo "Scanning: ${IMAGE_NAME}:${IMAGE_TAG}"

                    trivy image \
                      --scanners vuln \
                      --severity CRITICAL \
                      --timeout 10m \
                      --skip-version-check \
                      ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Trivy Security Gate Passed ====="
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                sh '''
                    echo "===== Docker Hub Login ====="

                    echo "$DOCKERHUB_PSW" | docker login \
                      -u "$DOCKERHUB_USR" \
                      --password-stdin

                    echo "===== Tagging Image ====="

                    docker tag \
                      ${IMAGE_NAME}:${IMAGE_TAG} \
                      ${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Pushing Image To Docker Hub ====="

                    docker push \
                      ${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Docker Hub Push Completed ====="
                '''
            }
        }

        stage('Docker Deploy') {
            steps {
                sh '''
                    echo "===== Pulling Image From Docker Hub ====="

                    docker pull \
                      ${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Stopping Old Container ====="

                    docker stop ${CONTAINER_NAME} || true

                    echo "===== Removing Old Container ====="

                    docker rm ${CONTAINER_NAME} || true

                    echo "===== Starting Container From Docker Hub ====="

                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p 8000:7000 \
                      ${DOCKERHUB_USR}/${IMAGE_NAME}:${IMAGE_TAG}

                    echo "===== Removing Old Local Images ====="

                    docker images "${IMAGE_NAME}" \
                      --format "{{.Repository}}:{{.Tag}}" \
                      | grep -v "^${IMAGE_NAME}:${IMAGE_TAG}$" \
                      | xargs -r docker rmi || true

                    echo "===== Running Container ====="

                    docker ps

                    echo "===== Application ====="
                    echo "http://localhost:8000"
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'Pipeline completed successfully!'
            echo "Image: ${IMAGE_NAME}:${IMAGE_TAG}"
            echo 'Image pushed to Docker Hub'
            echo 'Image pulled from Docker Hub'
            echo 'Application: http://localhost:8000'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'Pipeline FAILED!'
            echo 'Check the failed stage above.'
            echo '======================================'
        }
    }
}