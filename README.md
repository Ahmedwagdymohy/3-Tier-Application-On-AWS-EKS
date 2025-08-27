# DevOps Learning Platform – 3-Tier EKS Application

This project is a hands-on, production-grade example of deploying a full-stack 3-tier application on AWS using Kubernetes (EKS). It is ideal for anyone looking to apply Kubernetes in practice while gaining real-world experience with AWS cloud infrastructure.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Tech Stack & Tools](#tech-stack--tools)
- [Features](#features)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Development](#local-development)
  - [Production Deployment on AWS EKS](#production-deployment-on-aws-eks)
- [Troubleshooting & Real-World Issues](#troubleshooting--real-world-issues)
- [Contributing](#contributing)
- [License](#license)

---

## Project Overview

This platform helps users learn DevOps concepts through interactive quizzes. It demonstrates a real-world, cloud-native deployment with:

- **Frontend:** React app served via NGINX
- **Backend:** Flask REST API (Python)
- **Database:** PostgreSQL (AWS RDS)
- **Orchestration:** Kubernetes (AWS EKS)
- **Load Balancing:** AWS Application Load Balancer (ALB)

---

## Architecture

```
User → ALB → Ingress → Services → Pods → Database (RDS)
```

- **Frontend:** React + NGINX (Dockerized)
- **Backend:** Flask API (Dockerized)
- **Database:** PostgreSQL (managed by AWS RDS)
- **Kubernetes:** EKS cluster manages pods/services
- **Ingress:** AWS ALB Ingress Controller routes traffic
- **Networking:** VPC, subnets, and security groups for isolation

---

## Tech Stack & Tools

- **AWS EKS:** Kubernetes orchestration
- **eksctl, kubectl:** Cluster and resource management
- **AWS RDS:** Managed PostgreSQL database
- **ALB + Ingress Controller:** Traffic routing
- **Terraform:** Infrastructure as Code (in `infra/`)
- **Helm:** Kubernetes package management
- **Docker:** Containerization
- **CloudWatch, Prometheus, Grafana:** Monitoring & logging

---

## Features

- Interactive DevOps quizzes (Kubernetes, AWS, Docker, Jenkins, Linux)
- User-friendly React frontend
- RESTful Flask backend API
- Secure, scalable, and highly available architecture
- Real-world CI/CD, monitoring, and troubleshooting patterns

---

## Getting Started

### Prerequisites

- AWS CLI, kubectl, eksctl, Helm, Docker installed
- AWS account with permissions for EKS, RDS, VPC, IAM, ALB, Route 53

### Local Development

#### Backend

```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Start local PostgreSQL (Docker)
docker run --name flask_postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=password -e POSTGRES_DB=devops_learning -p 5432:5432 -d postgres

# Set environment variables (see .env.example)
export FLASK_APP=run.py
flask db init
flask db migrate -m "Initial migration"
flask db upgrade

# Seed data
python seed_data.py

# Run backend
python run.py
```

#### Frontend

```bash
cd frontend
npm install
npm start
# App runs at http://localhost:3000
```

### Production Deployment on AWS EKS

1. **Clone the repository and organize manifests:**
	```bash
	git clone <repo-url>
	cd 3-tier-app-eks/
	```

2. **Provision EKS Cluster:**
	```bash
	eksctl create cluster --name <cluster-name> --region <region> --version 1.31 --nodegroup-name standard-workers --node-type t3.medium --nodes 2 --managed
	```

3. **Create RDS PostgreSQL Database:**
	- Use Terraform in `infra/` or AWS CLI.
	- Ensure RDS is in private subnets and security group allows EKS nodes on port 5432.

4. **Build & Push Docker Images:**
	```bash
	# Authenticate to ECR
	aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

	# Build and push images
	docker build -t <ecr-repo>/frontend:latest ./frontend
	docker build -t <ecr-repo>/backend:latest ./backend
	docker push <ecr-repo>/frontend:latest
	docker push <ecr-repo>/backend:latest
	```

5. **Update Kubernetes Secrets/ConfigMaps:**
	- Set correct `DATABASE_URL` and secrets in `k8s/secrets.yaml`.

6. **Deploy Kubernetes Resources:**
	```bash
	kubectl apply -f k8s/namespace.yaml
	kubectl apply -f k8s/database-service.yaml
	kubectl apply -f k8s/secrets.yaml
	kubectl apply -f k8s/backend.yaml
	kubectl apply -f k8s/frontend.yaml
	kubectl apply -f k8s/hpa.yaml
	kubectl apply -f k8s/ingress.yaml
	```

7. **Configure ALB Ingress Controller:**
	- Follow AWS docs to install and configure.
	- Ensure correct IAM roles and OIDC provider.

8. **Set up DNS (Optional):**
	- Use Route 53 to point your domain to the ALB DNS.

9. **SSL/TLS (Optional):**
	- Use cert-manager and Route 53 for automated Let's Encrypt certificates (see `k8s/readme2.md`).

---

## Troubleshooting & Real-World Issues

### 1. Database Connectivity
- **Problem:** Backend can't connect to RDS.
- **Cause:** RDS security group doesn't allow EKS node access.
- **Fix:** Attach a custom SG to RDS allowing inbound 5432 from EKS worker nodes.

### 2. Frontend-Backend Communication
- **Problem:** React app can't reach backend API.
- **Cause:** Port/path mismatch.
- **Fix:** NGINX reverse proxy in frontend container forwards `/api/*` to backend.

### 3. ALB Health Check
- **Problem:** ALB marks backend as "Unhealthy".
- **Cause:** Health check path `/` not present in Flask backend.
- **Fix:** Change ingress annotation to use `/api/topics` or another valid endpoint.

### 4. General Kubernetes Debugging

```bash
# Check pod logs
kubectl logs -f deploy/backend -n <namespace>
kubectl logs -f deploy/frontend -n <namespace>

# Describe pod status
kubectl describe pod -l app=backend -n <namespace>
kubectl describe pod -l app=frontend -n <namespace>

# Restart deployments
kubectl rollout restart deployment backend -n <namespace>
kubectl rollout restart deployment frontend -n <namespace>
```

### 5. Monitoring & Logging

- Prometheus and Grafana are set up for monitoring (see `setup-monitoring.sh`).
- AWS CloudWatch agent can be installed for log aggregation.


