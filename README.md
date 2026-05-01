🚀 Project: End-to-End CI/CD Pipeline with Kubernetes on AWS

🎯 Introduction

This project demonstrates a complete DevOps workflow using Docker, Kubernetes (Kind), AWS EC2 & ECR, and GitHub Actions. A multi-service application (frontend, backend, MongoDB) is containerized, deployed to a local Kubernetes cluster on EC2, and managed via a CI/CD pipeline.

Every push to the main branch:

Builds Docker images for frontend and backend
Pushes images to Amazon ECR
Updates deployments in Kubernetes cluster automatically via GitHub Actions

🌍 End Result
✅ Fully automated deployment of multi-service application on AWS EC2 ✅ Kubernetes cluster managed via Kind ✅ CI/CD pipeline using GitHub Actions and a self-hosted runner ✅ Dockerized frontend (Nginx + HTML) and backend (Flask) ✅ MongoDB service running inside cluster ✅ Application accessible via EC2 public IP on port 30080

📁 Project Structure
.
├── README.md
├── backend
│   ├── Dockerfile
│   ├── app.py
│   └── requirements.txt
├── frontend
│   ├── Dockerfile
│   ├── index.html
│   └── nginx.conf
├── k8s
│   ├── backend-deployment.yaml
│   ├── frontend-deployment.yaml
│   └── mongo-deployment.yaml
├── kind-cluster.yaml
└── .github
    └── workflows
        └── cicd.yaml
🧠 Application Overview
🖥 Frontend
Tech: Nginx + HTML
Service Type: NodePort
Communicates with backend using Kubernetes DNS: http://backend-service:5000
⚙ Backend
Tech: Python + Flask
Service Type: ClusterIP
Connects to MongoDB via environment variables
🗄 Database
Tech: MongoDB
Service Type: ClusterIP
Ephemeral storage (no persistent volume yet)

🐳 Step 1 – EC2 & Kubernetes Setup
EC2 Configuration
Amazon Linux 2023 (or Amazon Linux 2)

Instance type: t2.medium

Security Group:

SSH → 22
Application → 30080 (NodePort)

sudo dnf update -y
sudo dnf install -y docker git curl
sudo systemctl start docker
sudo usermod -aG docker ec2-user
newgrp docker

Install kubectl

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

Install Kind

curl -Lo kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/

Create Kubernetes Cluster

kind create cluster --name multi-service --config kind-cluster.yaml
kubectl get nodes

Export kubeconfig for GitHub runner:

kind get kubeconfig --name multi-service > ~/.kube/config
export KUBECONFIG=~/.kube/config

✅ Cluster name: multi-service ✅ NodePort mapped: 30080

☁️ Step 2 – AWS ECR Setup

Configure AWS CLI

aws configure

Create Repositories

aws ecr create-repository --repository-name backend-service
aws ecr create-repository --repository-name frontend-service

Login to ECR

aws ecr get-login-password --region <AWS_REGION> \
| docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com

Create Kubernetes Image Pull Secret

kubectl create secret docker-registry ecr-secret \
--docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com \
--docker-username=AWS \
--docker-password=$(aws ecr get-login-password --region <AWS_REGION>)

Use in deployments:

imagePullSecrets:
  - name: ecr-secret
⚠️ Do not hardcode AWS account ID; use GitHub secrets instead.

🔑 Step 3 – GitHub Repository Secrets
Create the following repository secrets:

Secret Name	Description
AWS_ACCESS_KEY_ID	AWS access key
AWS_SECRET_ACCESS_KEY	AWS secret key
These are used by GitHub Actions to authenticate with AWS and ECR.

⚙ Step 4 – GitHub Self-Hosted Runner Setup

Install Dependencies
sudo dnf install -y libicu krb5-libs zlib openssl-libs libcurl ca-certificates tar gzip --allowerasing

Configure Runner
Go to GitHub → Repo → Settings → Actions → Runners → New self-hosted runner → Linux

Download and configure runner:

./config.sh --url https://github.com/<OWNER>/<REPO> --token <RUNNER_TOKEN>

Run as service:

./run.sh          # Test listener
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
Verify:

kubectl get nodes

🐳 Step 5 – Build & Push Docker Images

Backend

docker build -t backend-service:1.0 backend/
docker tag backend-service:1.0 <ECR_URI>/backend-service:1.0
docker push <ECR_URI>/backend-service:1.0

Frontend

docker build -t frontend-service:1.0 frontend/
docker tag frontend-service:1.0 <ECR_URI>/frontend-service:1.0
docker push <ECR_URI>/frontend-service:1.0
<ECR_URI> = <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com

🏗 Step 6 – Kubernetes Deployment

Apply manifests:

kubectl apply -f k8s/mongo-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
Validate:

kubectl get pods
kubectl get svc
Access app:

http://<EC2_PUBLIC_IP>:30080

🔁 Step 7 – GitHub Actions Workflow

Push code → GitHub Actions → Build Docker images → Push to ECR → Self-hosted Runner → Update Kubernetes → Pods restart → App updated in browser.
Workflow uses repository secrets for AWS authentication.
imagePullSecrets ensures Kubernetes can pull private images from ECR.

🧪 Step 8 – Validation Commands

kubectl get pods
kubectl get svc
kubectl rollout status deployment backend
kubectl rollout status deployment frontend
kubectl logs <pod-name>
kubectl describe pod <pod-name>

✅ Summary
✔ Built multi-service application (frontend + backend + MongoDB) ✔ Dockerized services ✔ Created Kubernetes cluster using Kind on EC2 ✔ Configured CI/CD with GitHub Actions and self-hosted runner ✔ Pushed images to AWS ECR ✔ Automated deployment to Kubernetes ✔ Access via EC2 public IP

🧑‍🎓 Learning Outcomes

Kubernetes cluster setup on EC2 using Kind
Multi-service container orchestration
CI/CD pipelines with GitHub Actions
AWS ECR integration
Self-hosted GitHub runners
Real-world DevOps automation
