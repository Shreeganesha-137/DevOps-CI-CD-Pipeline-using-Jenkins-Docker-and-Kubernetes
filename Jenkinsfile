pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        DOCKERHUB_REPO = 'shreeganesha237'
        KUBECONFIG_PATH = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📦 Cloning repository...'
                git branch: 'main', url: 'https://github.com/Shreeganesha-137/DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo '🐳 Building Docker images...'
                    sh """
                        docker build -t ${DOCKERHUB_REPO}/devops-backend:latest ./backend
                        docker build -t ${DOCKERHUB_REPO}/devops-frontend:latest ./frontend
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
                            docker push ${DOCKERHUB_REPO}/devops-backend:latest
                            docker push ${DOCKERHUB_REPO}/devops-frontend:latest
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo '🚀 Deploying application to Kubernetes...'
                    sh """
                        export KUBECONFIG=${KUBECONFIG_PATH}

                        # Apply or update deployments and service
                        kubectl apply -f k8s/backend-deployment.yaml
                        kubectl apply -f k8s/frontend-deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        # Verify rollout
                        kubectl rollout status deployment/devops-backend
                        kubectl rollout status deployment/devops-frontend
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful — backend and frontend images updated to :latest!"
        }
        failure {
            echo "❌ Pipeline failed — check Jenkins logs."
        }
    }
}
