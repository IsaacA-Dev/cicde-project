pipeline {
    agent any
    
    environment {
        DOCKER_CREDENTIALS = credentials('docker_hub')
        DOCKER_IMAGE = 'isaacadev/mi-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Código ya disponible en workspace'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                script {
                    echo 'Logging into Docker Hub...'
                    sh 'echo $DOCKER_CREDENTIALS_PSW | docker login -u $DOCKER_CREDENTIALS_USR --password-stdin'
                    
                    echo "Pushing ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'
                    // Actualizar la imagen en el deployment
                    sh """
                        sed -i 's|image: .*|image: ${DOCKER_IMAGE}:${IMAGE_TAG}|g' kubernetes-deployment.yaml
                        kubectl apply -f kubernetes-deployment.yaml --validate=false
                        kubectl set image deployment/mi-app-deployment mi-app=${DOCKER_IMAGE}:${IMAGE_TAG}
                        kubectl rollout status deployment/mi-app-deployment --timeout=2m
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    echo 'Verificando deployment...'
                    sh 'kubectl get deployments'
                    sh 'kubectl get services'
                    sh 'kubectl get pods'
                }
            }
        }
    }
    
    post {
        success {
            echo '¡Pipeline ejecutado exitosamente!'
            echo 'Accede a tu aplicación con: minikube service mi-app-service --url'
        }
        failure {
            echo 'Pipeline falló. Revisar logs.'
        }
        always {
            sh 'docker logout'
        }
    }
}
