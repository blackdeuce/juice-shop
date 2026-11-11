
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'blackcyber07/juice-shop'
        IMAGE_TAG  = "${BUILD_NUMBER}"
        REGISTRY   = 'docker.io'
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
                    docker build \
                    -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .

                    docker tag \
                    ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                    ${REGISTRY}/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'd14c7385-43cf-44fc-ab9a-0559725e9564',
                        usernameVariable: 'REG_USER',
                        passwordVariable: 'REG_PASS'
                    )
                ]) {
                    sh '''
                        set +x

                        printf '%s' "$REG_PASS" | docker login \
                        -u "$REG_USER" --password-stdin

                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f juice-shop || true

                    docker run -d \
                    --name juice-shop \
                    -p 3000:3000 \
                    ${REGISTRY}/${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Application Test') {
            steps {
                sh '''
                    for i in $(seq 1 20); do
                        if curl -fsS http://localhost:3000/ > /dev/null; then
                            echo "Juice Shop is running successfully!"
                            exit 0
                        fi

                        sleep 3
                    done

                    echo "Application health check failed!"
                    docker logs juice-shop
                    exit 1
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
            cleanWs()
        }
    }
}
