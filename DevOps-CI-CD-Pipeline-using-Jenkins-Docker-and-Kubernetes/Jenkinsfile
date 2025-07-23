pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {
        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/TejPATHAK/DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    docker.build("tpathak21/devops-backend:latest", "./backend")
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    docker.build("tpathak21/devops-frontend:latest", "./frontend")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        echo 'Logged in to DockerHub'
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        docker.image("tpathak21/devops-backend:latest").push()
                        docker.image("tpathak21/devops-frontend:latest").push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export KUBECONFIG=/var/lib/jenkins/.kube/config
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/frontend-deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
