# Django Todo App - CI/CD & Kubernetes Deployment

A simple todo app built with Django, containerized with Docker, and deployed on Kubernetes using a Jenkins CI/CD pipeline running on an AWS EC2 instance.

![todo App](https://raw.githubusercontent.com/Pratik543/django-notes-app-ci-cd/main/staticfiles/todoApp.png)

## Architecture Overview
- **Application:** Django Todo App
- **Infrastructure:** AWS EC2 (Ubuntu)
- **CI/CD:** Jenkins
- **Containerization:** Docker
- **Container Orchestration:** Kubernetes (Minikube)

---

## 🏆 Achievements & Project Highlights
- **End-to-End Automation:** Designed and implemented a fully automated Jenkins CI/CD pipeline, significantly streamlining the release process from code commit to deployment.
- **Containerized Architecture:** Successfully Dockerized a Django application, ensuring identical, reproducible environments across development, testing, and production.
- **Kubernetes Orchestration:** Engineered scalable and fault-tolerant application pods using Minikube, employing Kubernetes Deployments and Services to manage application state and networking.
- **Cloud Infrastructure Management:** Configured and provisioned AWS EC2 instances, adhering to network security best practices through meticulous Security Group configurations.
- **Secure Configuration Management:** Effectively decoupled application configuration and sensitive data from the codebase utilizing Kubernetes ConfigMaps and Secrets, securing MySQL database credentials.

---

## 1. AWS EC2 Setup and Prerequisites
1. **Launch an EC2 Instance:** Launch an Ubuntu EC2 instance on AWS (t2.medium or higher recommended for Minikube and Jenkins).
2. **Configure Security Groups:** Open the following ports in the security group:
   - `22` (SSH)
   - `80` (HTTP)
   - `8080` (Jenkins)
   - `8000` (App for Docker Compose)
   - `30007` (NodePort for Django App)

3. **Install Dependencies:**
Connect to your EC2 instance and install Java, Jenkins, Docker, Minikube, and `kubectl`.
*(Make sure to add the `ubuntu` and `jenkins` users to the `docker` group).*

```bash
# Install Java
sudo apt update
sudo apt install openjdk-21-jdk -y

# Install Jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
sudo echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

# Install Docker
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu
sudo usermod -aG docker jenkins

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg \
  https://packages.cloud.google.com/apt/doc/apt-key.gpg
sudo echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] \
  https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee \
  /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

---

## 2. CI/CD Pipeline Configuration (Jenkins)
1. **Access Jenkins:** Navigate to `http://<YOUR_EC2_PUBLIC_IP>:8080`.
2. **Install Plugins:** Ensure Docker Pipeline and Kubernetes CLI plugins are installed in Jenkins.
3. **Configure Credentials:** Add your Docker Hub credentials in `Manage Jenkins -> Credentials`.
4. **Create a Pipeline Job:** Create a new Jenkins Pipeline that pulls the code from this repository.
5. **Pipeline Stages:** The pipeline generally executes the following typical stages:
   - Checkout code
   - Build Docker Image (`docker build -t <your-dockerhub-username>/django-todo:latest .`)
   - Run Docker Compose (`docker compose up -d`)
   - Push to Docker Hub

---

## 3. Kubernetes Deployment with Minikube
Ensure Minikube is started on your EC2 instance (`minikube start`).

### Deploying the Database
First, set up the MySQL database required by the application. Run the following commands:
```bash
# Apply ConfigMap and Secret for MySQL
kubectl apply -f k8s/MYSQL-DB/configMap.yml
kubectl apply -f k8s/MYSQL-DB/secret.yml

# Deploy MySQL
kubectl apply -f k8s/MYSQL-DB/deployment.yml
```

### Deploying the Application
Deploy the Django application and expose it via a NodePort service:
```bash
# Apply App Deployment and Service
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

### Verify Deployment
Check if all pods and services are running:
```bash
kubectl get pods
kubectl get svc
```

---

## 4. Accessing the Application
The `todo-service` is exposed as a `NodePort` service on port `30007`.

Using your AWS EC2 public IP:
```text
http://<YOUR_EC2_PUBLIC_IP>:30007
```

*(Note: Minikube might require you to run `minikube service todo-service` or set up port forwarding / a reverse proxy to access NodePort from the external EC2 IP depending on your networking setup, typically by running `kubectl port-forward --address 0.0.0.0 svc/todo-service 30007:80 &`)*.

---

### Original Owner Project
To get the original repository, run the following command inside your git enabled terminal:
```bash
git clone https://github.com/shreys7/django-todo.git
```