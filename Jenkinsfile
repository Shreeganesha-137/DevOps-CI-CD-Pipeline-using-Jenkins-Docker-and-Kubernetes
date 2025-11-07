pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKERHUB_REPO = 'shreeganesha237'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        KUBECONFIG_PATH = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/Shreeganesha-137/DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    echo '🐳 Building backend Docker image...'
                    sh """
                        docker build -t ${DOCKERHUB_REPO}/devops-backend:${IMAGE_TAG} ./backend
                        docker tag ${DOCKERHUB_REPO}/devops-backend:${IMAGE_TAG} ${DOCKERHUB_REPO}/devops-backend:latest
                    """
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    echo '🐳 Building frontend Docker image...'
                    sh """
                        docker build -t ${DOCKERHUB_REPO}/devops-frontend:${IMAGE_TAG} ./frontend
                        docker tag ${DOCKERHUB_REPO}/devops-frontend:${IMAGE_TAG} ${DOCKERHUB_REPO}/devops-frontend:latest
                    """
                }
            }
        }

        stage('Push Images to DockerHub') {
            steps {
                script {
                    echo '📤 Pushing Docker images to DockerHub...'
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                            docker push ${DOCKERHUB_REPO}/devops-backend:${IMAGE_TAG}
                            docker push ${DOCKERHUB_REPO}/devops-backend:latest

                            docker push ${DOCKERHUB_REPO}/devops-frontend:${IMAGE_TAG}
                            docker push ${DOCKERHUB_REPO}/devops-frontend:latest

                            docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Deploying application to Kubernetes...'
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}

                    # Update deployments with new image versions
                    kubectl set image deployment/backend-deployment backend=${DOCKERHUB_REPO}/devops-backend:${IMAGE_TAG} --record
                    kubectl set image deployment/frontend-deployment frontend=${DOCKERHUB_REPO}/devops-frontend:${IMAGE_TAG} --record

                    # Apply services
                    kubectl apply -f k8s/service.yaml

                    # Verify rollout
                    kubectl rollout status deployment/backend-deployment
                    kubectl rollout status deployment/frontend-deployment
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully! Images pushed and deployed with tag ${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
