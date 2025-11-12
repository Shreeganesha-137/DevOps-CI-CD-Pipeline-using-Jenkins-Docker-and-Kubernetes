# DevOps CI/CD Pipeline using Jenkins, Docker, and Kubernetes

## 🚀 Project Objective

Design and automate a robust CI/CD pipeline to deploy a multi-tier web application (frontend and backend) using:

- **Jenkins** for Continuous Integration and Continuous Deployment
- **Docker** for containerization
- **Kubernetes** for orchestration (via **Minikube**)
- **AWS EC2** (t2.medium) as the host environment
- **GitHub** as the code repository

The goal is to build a reliable, production-ready DevOps workflow.

---

## 🌐 Hosting Environment

- **Cloud Platform**: AWS EC2
- **Instance Type**: t2.medium (2 vCPU, 4 GB RAM)
- **Operating System**: Ubuntu 22.04 LTS
- **Security Group Ports Opened**: 22 (SSH), 8080 (Jenkins), 30080 (Backend), 30081 (Frontend)
- **Minikube Tunnel**: Enabled using `minikube tunnel --bind-address 0.0.0.0`

---

## 🛠️ Tools & Technologies

| Tool       | Purpose                            |
| ---------- | ---------------------------------- |
| GitHub     | Version Control                    |
| Docker     | Containerize frontend/backend apps |
| Jenkins    | Automate CI/CD pipeline            |
| Kubernetes | Manage containerized workloads     |
| Minikube   | Local Kubernetes cluster           |
| Nginx      | Frontend static file server        |
| Flask      | Python backend web server          |

---

## 📁 Project Structure

```
DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes/
├── backend/
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── frontend/
│   ├── Dockerfile
│   └── index.html
├── k8s/
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── service.yaml
├── Jenkinsfile
├── docker-compose.yml
└── README.md
```

---

## ⚙️ Setup and Installation

### 🔧 Install Required Tools

```bash
sudo apt update && sudo apt install -y git curl docker.io
sudo usermod -aG docker $USER
```

Install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Install kubectl:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

Start Minikube:

```
minikube start --driver=docker
minikube tunnel --bind-address 0.0.0.0
```

---

## 📦 Docker: Containerization

## Backend Dockerfile
```
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install Flask==2.0.3 Werkzeug==2.0.3
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```
## Frontend Dockerfile
```
FROM nginx:alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
### Build & Push Images
```
docker build -t tpathak21/devops-backend:latest ./backend
docker build -t tpathak21/devops-frontend:latest ./frontend

docker push tpathak21/devops-backend:latest
docker push tpathak21/devops-frontend:latest

```
### Apply Resources

```
kubectl apply -f k8s/
kubectl get pods -o wide
kubectl get svc
```

---

## 🔄 Jenkins CI/CD Pipeline

### Jenkinsfile
---
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
`
---
## ✅ Output Validation

curl Test

```
curl http://<EC2-PUBLIC-IP>:30080    # Backend
curl http://<EC2-PUBLIC-IP>:30081    # Frontend
```

### Jenkins Pipeline

- All stages: Clone > Build > Push > Deploy should show as green ✅


## 📎 Links

- 🔗 GitHub Repository: [DevOps-CI-CD-Pipeline](https://github.com/tpathak21/DevOps-CI-CD-Pipeline-using-Jenkins-Docker-and-Kubernetes)
- 🔗 LinkedIn Profile: [Tejaswi Pathak](https://www.linkedin.com/in/tejaswi-pathak)

---

## 📚 Conclusion

This project is a full-fledged demonstration of how to containerize applications, automate deployments using Jenkins, and orchestrate containers via Kubernetes — all deployed on an AWS EC2 instance. It proves your ability to integrate multiple DevOps tools in a production-like setup.

