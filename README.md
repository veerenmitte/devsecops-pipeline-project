DevSecOps Pipeline Project – Deploy Netflix Clone on Kubernetes
Project Overview

This project demonstrates a complete DevSecOps CI/CD pipeline by deploying a Netflix Clone application on Kubernetes using modern DevOps and security tools.

Tools Used:
GitHub
Jenkins
Docker
Kubernetes
SonarQube
Trivy
Prometheus
Grafana

Project Architecture:

Developer → GitHub → Jenkins → Docker → Kubernetes → Monitoring & Security

Features:
CI/CD Pipeline Automation
Docker Containerization
Kubernetes Deployment
Security Scanning with Trivy
Code Quality Analysis using SonarQube
Monitoring using Prometheus & Grafana

Step 1: Clone Repository
git clone https://github.com/your-username/netflix-clone-devsecops.git
cd netflix-clone-devsecops

Step 2: Install Docker
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
Step 3: Install Minikube

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

Start Minikube:

minikube start --driver=docker

Install kubectl:

sudo snap install kubectl --classic
Step 4: Install Jenkins
sudo apt update
sudo apt install openjdk-17-jdk -y
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y

Start Jenkins:

sudo systemctl enable jenkins
sudo systemctl start jenkins

Get Jenkins Password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Access Jenkins:

http://<server-ip>:8080
Step 5: Install SonarQube
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community

Access SonarQube:

http://<server-ip>:9000

Default Credentials:

Username: admin
Password: admin

Step 6: Install Trivy
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | \
gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | \
sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y

Test Trivy:

trivy image nginx

Step 7: Docker Build and Push

Build Image:

docker build -t netflix-clone .

Tag Image:

docker tag netflix-clone your-dockerhub-username/netflix-clone:latest

Push Image:

docker push your-dockerhub-username/netflix-clone:latest

Step 8: Kubernetes Deployment
deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: netflix-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: netflix
  template:
    metadata:
      labels:
        app: netflix
    spec:
      containers:
      - name: netflix-container
        image: your-dockerhub-username/netflix-clone:latest
        ports:
        - containerPort: 80
service.yaml
apiVersion: v1
kind: Service
metadata:
  name: netflix-service
spec:
  type: NodePort
  selector:
    app: netflix
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007

Apply Files:

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

Check Pods:

kubectl get pods

Check Services:

kubectl get svc

Step 9: Jenkins Pipeline
Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "your-dockerhub-username/netflix-clone"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/your-username/netflix-clone-devsecops.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                sh 'sonar-scanner'
            }
        }

        stage('Trivy File Scan') {
            steps {
                sh 'trivy fs .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image $DOCKER_IMAGE'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }
}

Step 10: Monitoring Setup

Create Monitoring Namespace:

kubectl create namespace monitoring

Add Helm Repo:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Install Prometheus & Grafana:

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

Get Grafana Password:

kubectl get secret --namespace monitoring prometheus-grafana \
-o jsonpath="{.data.admin-password}" | base64 --decode

Port Forward Grafana:

kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring

Access Grafana:

http://localhost:3000

Username:admin

Security Best Practices:
Use Kubernetes Secrets
Enable RBAC
Scan Images Regularly
Avoid Hardcoded Credentials
Use HTTPS

Future Enhancements:
Add ArgoCD
Add Terraform
Add Helm Charts
Add Email Notifications

Conclusion:
This project demonstrates a complete DevSecOps pipeline using Jenkins, Docker, Kubernetes, SonarQube, Trivy, Prometheus, and Grafana for secure and automated application deployment.
