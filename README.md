# Doctor Recommendation System — MLOps & DevOps Pipeline

An end-to-end **MLOps-driven healthcare recommendation platform** built using modern DevOps and cloud-native technologies. This project automates the complete software delivery lifecycle for a microservices-based Doctor Recommendation System using **Docker, Kubernetes, Jenkins, Ansible, and the ELK Stack**.

---

# Project Overview

The system is designed as a **microservices-based healthcare recommendation platform** consisting of three independent services:

- Frontend Gateway Service
- Speciality Predictor Service
- Bandit Recommender Service

The application allows users to enter symptoms through a web interface, predicts the most relevant medical speciality using a biomedical NLP model, and recommends ranked doctors using a reinforcement learning-based recommendation engine.

---

# Key Features

- Microservices-based architecture
- Automated CI/CD pipeline using Jenkins
- Docker-based containerization
- Kubernetes orchestration with Minikube
- Automated deployments using Ansible
- Centralized logging using Elasticsearch & Kibana
- Biomedical NLP-based speciality prediction
- Reinforcement learning-based doctor recommendation
- End-to-end MLOps workflow automation

---

# System Architecture

```text
GitHub → Jenkins CI/CD → Docker Images → Docker Hub →
Minikube Kubernetes Cluster → Ansible Deployment →
Frontend + ML Microservices → ELK Monitoring Stack
```

The architecture includes:

- GitHub for version control
- Jenkins for CI/CD automation
- Docker for containerization
- Kubernetes for orchestration
- Ansible for deployment automation
- Elasticsearch + Kibana for monitoring and observability

---

# Microservices

## 1. Frontend Gateway

Acts as the user interaction layer of the application.

### Responsibilities
- Accepts user symptom input
- Sends requests to backend services
- Displays predicted specialities and doctor recommendations

### Tech Stack
- FastAPI
- HTML/CSS/JavaScript
- Uvicorn

---

## 2. Speciality Predictor Service

Predicts the most relevant medical speciality from symptom descriptions.

### Features
- Biomedical NLP-based inference
- BioDistillGPT-2 transformer model
- REST API-based prediction service

### Tech Stack
- Flask
- Transformers
- PyTorch
- spaCy

---

## 3. Bandit Recommender Service

Ranks and recommends doctors using reinforcement learning principles.

### Features
- Epsilon-Greedy Multi-Armed Bandit algorithm
- Balances exploration and exploitation
- Recommendation ranking engine

### Tech Stack
- FastAPI
- NumPy
- Pandas

---

# Tech Stack

| Category | Technologies |
|---|---|
| Backend Frameworks | FastAPI, Flask |
| Machine Learning | BioDistillGPT-2, PyTorch, Transformers |
| Recommendation Engine | Epsilon-Greedy Multi-Armed Bandit |
| Containerization | Docker |
| Orchestration | Kubernetes (Minikube) |
| CI/CD | Jenkins |
| Configuration Management | Ansible |
| Monitoring | Elasticsearch, Kibana |
| Version Control | Git, GitHub |

---

# Repository Structure

```text
.
├── main1/                  # Bandit Recommender Service
├── main2/                  # Speciality Predictor Service
├── app.py                  # Frontend Gateway
├── Ansible/                # Deployment automation
├── Kubernetes/             # Kubernetes manifests
├── Jenkinsfile             # Jenkins CI/CD pipeline
├── Dockerfile              # Frontend Dockerfile
├── requirements.txt
└── README.md
```

---

# CI/CD Workflow

The Jenkins pipeline automates:

1. Code checkout from GitHub
2. Docker image builds
3. Docker image push to Docker Hub
4. Loading images into Minikube
5. Kubernetes deployment using Ansible
6. Rollout verification
7. Continuous deployment automation

---

# Docker Images

The project uses the following Docker images:

```text
shrutimrao/bandit
shrutimrao/speciality
shrutimrao/frontend
```

---

# Setup Guide

## PHASE 1 — Install Prerequisites

Install the following tools on your system:

- Docker Desktop
- Minikube
- kubectl
- Ansible
- Jenkins

Verify installations:

```bash
docker --version
minikube version
kubectl version --client
ansible --version
jenkins --version
```

---

## PHASE 2 — Build & Push Docker Images

```bash
# Login to Docker Hub
docker login

# Build bandit service image
docker build -t shrutimrao/bandit:latest ./main1

# Build speciality service image
docker build -t shrutimrao/speciality:latest ./main2

# Build frontend image
docker build -t shrutimrao/frontend:latest .

# Push images to Docker Hub
docker push shrutimrao/bandit:latest
docker push shrutimrao/speciality:latest
docker push shrutimrao/frontend:latest
```

> NOTE: `main2` (speciality service) contains a large ML model (`BioDistillGPT2`) and may take 5–10 minutes to build.

---

## PHASE 3 — Start Minikube

```bash
# Start Minikube
minikube start --driver=docker --memory=6144 --cpus=4

# Verify cluster status
kubectl get nodes
```

Expected output:

```text
minikube   Ready
```

---

## PHASE 4 — Load Images into Minikube

```bash
# Load images into Minikube
minikube image load shrutimrao/bandit:latest

minikube image load shrutimrao/speciality:latest

minikube image load shrutimrao/frontend:latest

# Verify images
minikube image ls | grep shrutimrao
```

---

## PHASE 5 — Deploy Using Ansible

```bash
ansible-playbook -i Ansible/hosts.ini Ansible/deploy.yml \
  -e bandit_tag=latest \
  -e speciality_tag=latest \
  -e frontend_tag=latest
```

---

## PHASE 6 — Verify Deployment

```bash
# Watch pod status
kubectl get pods -w
```

Expected running pods:

```text
bandit-deployment-xxx      1/1   Running
speciality-deployment-xxx  1/1   Running
frontend-deployment-xxx    1/1   Running
```

```bash
# Get frontend URL
minikube service frontend --url
```

Open the generated URL in your browser.

---

## PHASE 7 — Setup ELK Stack (Centralized Logging)

```bash
# Create logging namespace
kubectl create namespace logging
```

### Deploy Elasticsearch

```bash
kubectl apply -f Kubernetes/logging/elasticsearch.yaml

kubectl get pods -n logging
```

---

### Deploy Kibana

```bash
kubectl apply -f Kubernetes/logging/kibana.yaml

kubectl get pods -n logging
```

---

### Deploy Filebeat

```bash
# Download Filebeat configuration
curl -O https://raw.githubusercontent.com/elastic/beats/7.17/deploy/kubernetes/filebeat-kubernetes.yaml
```

Update Elasticsearch endpoint inside `filebeat-kubernetes.yaml`:

```text
elasticsearch.logging.svc.cluster.local:9200
```

Deploy Filebeat:

```bash
kubectl apply -f filebeat-kubernetes.yaml
```

---

### Open Kibana

```bash
minikube service kibana -n logging --url
```

Inside Kibana:

1. Select **Explore on my own**
2. Open **Stack Management**
3. Go to **Index Patterns**
4. Create index pattern:

```text
filebeat-*
```

5. Select time field:

```text
@timestamp
```

6. Open **Discover** to view logs

---

## PHASE 8 — Configure Jenkins (CI/CD)

Start Jenkins and open:

```text
http://localhost:8080
```

Get Jenkins initial admin password:

```bash
cat ~/.jenkins/secrets/initialAdminPassword
```

Install plugins:

- Docker Pipeline
- GitHub Integration
- Ansible

---

## Configure Kubernetes Access for Jenkins

```bash
mkdir -p ~/.kube

kubectl get nodes
```

Ensure Jenkins has access to:

- Docker
- kubectl
- ansible-playbook

---

## Jenkins Pipeline Setup

### Add Docker Hub Credentials

Navigate to:

```text
Manage Jenkins → Credentials → Global → Add Credentials
```

Configuration:

- Kind: `Username with password`
- Username: `shrutimrao`
- Password: your Docker Hub password
- ID: `dockerhub-credentials`

---

### Create Pipeline Job

```text
New Item → Pipeline
```

Pipeline configuration:

- Name: `MLOps-Project`
- Pipeline Definition: `Pipeline script from SCM`
- SCM: `Git`
- Repository URL: your GitHub repository URL
- Branch: `main`
- Script Path: `Jenkinsfile`

Click:

```text
Build Now
```

---

# Service URLs

| Service | Command / URL |
|---|---|
| Frontend App | `minikube service frontend --url` |
| Kibana | `minikube service kibana -n logging --url` |
| Jenkins | `http://localhost:8080` |
| Bandit API | `frontend-url + port 8000 + /docs` |

---

# Cleanup Commands

```bash
# Delete deployments
kubectl delete deployment bandit-deployment speciality-deployment frontend-deployment

# Delete logging namespace
kubectl delete namespace logging

# Stop Minikube
minikube stop
```

---

# Troubleshooting

## Speciality Pod Restarting

```bash
kubectl logs deployment/speciality-deployment
```

The `BioDistillGPT2` model may require additional startup time.

---

## ImagePullBackOff Error

```bash
minikube image load shrutimrao/speciality:latest
```

---

## Connection Refused in Browser

```bash
kubectl get pods
```

Ensure all pods show:

```text
1/1 Running
```

If `speciality` is still `0/1`, wait a few more minutes for model loading.

---

# Future Improvements

- Cloud deployment using AWS EKS / GKE / AKS
- Automated model retraining pipelines
- Real-time monitoring dashboards
- Advanced security integration
- Scalable production-grade infrastructure
- Continuous feedback learning systems
