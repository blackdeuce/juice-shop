pipeline {
    agent any

    environment {
        IMAGE_NAME = 'juice-shop'
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        REGISTRY   = 'docker.io/blackcyber07'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'd14c7385-43cf-44fc-ab9a-0559725e9564', usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
                    sh "echo \$REG_PASS | docker login -u \$REG_USER --password-stdin"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "docker rm -f ${IMAGE_NAME} || true"
                sh "docker run -d --name ${IMAGE_NAME} -p 3000:3000 ${REGISTRY}/${IMAGE_NAME}:latest"
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
