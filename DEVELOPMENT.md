# Development Setup Guide

This guide helps you set up the Quiz Builder application for local development.

## Prerequisites

- Docker Desktop
- Kubernetes cluster (minikube, kind, or Docker Desktop Kubernetes)
- Helm 3.0+
- Java 21
- Node.js 18+
- Maven 3.6+

## Local Development Setup

### 1. Start Local Kubernetes Cluster

**Using Minikube:**
```bash
minikube start --cpus=4 --memory=8192
minikube addons enable ingress
```

**Using Kind:**
```bash
kind create cluster --name quiz-builder-dev
```

**Using Docker Desktop:**
Enable Kubernetes in Docker Desktop settings.

### 2. Build Application Locally

**Backend Development:**
```bash
cd QuizBuilder-master/backend

# Install dependencies
mvn clean install

# Run locally
mvn spring-boot:run
```

**Frontend Development:**
```bash
cd QuizBuilder-master/frontend

# Install dependencies
npm install

# Run development server
npm start
```

### 3. Build Docker Images

```bash
# Build backend image
docker build -f Helm-Chart-Template-master/Dockerfile.backend -t quiz-builder-backend:local .

# Build frontend image
docker build -f Helm-Chart-Template-master/Dockerfile.frontend -t quiz-builder-frontend:local .
```

### 4. Deploy to Local Kubernetes

```bash
# Install the Helm chart with local images
helm install quiz-builder-dev Helm-Chart-Template-master \
  --set backend.image=quiz-builder-backend:local \
  --set frontend.image=quiz-builder-frontend:local \
  --set ingress.host=quiz-builder.local

# For minikube, get the ingress IP
minikube ip
# Add to /etc/hosts: <minikube-ip> quiz-builder.local

# For Docker Desktop, use localhost
# Add to /etc/hosts: 127.0.0.1 quiz-builder.local
```

### 5. Access the Application

- Application: http://quiz-builder.local
- Backend API: http://quiz-builder.local/api
- MySQL: localhost:3306 (port-forward)

## Development Workflow

### 1. Backend Changes

```bash
# Make changes to backend code
cd QuizBuilder-master/backend

# Build and test locally
mvn clean test

# Build new Docker image
docker build -f Helm-Chart-Template-master/Dockerfile.backend -t quiz-builder-backend:dev .

# Upgrade Helm release
helm upgrade quiz-builder-dev Helm-Chart-Template-master \
  --set backend.image=quiz-builder-backend:dev
```

### 2. Frontend Changes

```bash
# Make changes to frontend code
cd QuizBuilder-master/frontend

# Test locally
npm test

# Build production bundle
npm run build

# Build new Docker image
docker build -f Helm-Chart-Template-master/Dockerfile.frontend -t quiz-builder-frontend:dev .

# Upgrade Helm release
helm upgrade quiz-builder-dev Helm-Chart-Template-master \
  --set frontend.image=quiz-builder-frontend:dev
```

### 3. Database Changes

```bash
# Connect to MySQL
kubectl port-forward svc/mysql 3306:3306

# Use your preferred MySQL client
mysql -h localhost -u root -p quizbuilder
```

## Debugging

### Check Pod Status
```bash
kubectl get pods
kubectl describe pod <pod-name>
```

### View Logs
```bash
# Backend logs
kubectl logs -f deployment/backend

# Frontend logs
kubectl logs -f deployment/frontend

# Database logs
kubectl logs -f deployment/mysql
```

### Port Forwarding
```bash
# Backend
kubectl port-forward svc/backend 9096:9096

# Frontend
kubectl port-forward svc/frontend 80:80

# Database
kubectl port-forward svc/mysql 3306:3306
```

### Shell Access
```bash
# Backend shell
kubectl exec -it deployment/backend -- /bin/sh

# Frontend shell
kubectl exec -it deployment/frontend -- /bin/sh

# Database shell
kubectl exec -it deployment/mysql -- /bin/bash
```

## Testing

### API Testing
```bash
# Test backend health
curl http://quiz-builder.local/api/health

# Test quiz endpoints
curl http://quiz-builder.local/api/quizzes
```

### Load Testing
```bash
# Install hey (HTTP load generator)
go install github.com/rakyll/hey@latest

# Test frontend
hey -n 1000 -c 50 http://quiz-builder.local/

# Test backend API
hey -n 1000 -c 50 http://quiz-builder.local/api/quizzes
```

## Cleanup

```bash
# Uninstall Helm release
helm uninstall quiz-builder-dev

# Delete persistent volumes
kubectl delete pvc mysql-pvc

# Stop local cluster
minikube stop  # or kind delete cluster --name quiz-builder-dev
```

## Tips

1. **Hot Reload**: Use `skaffold` for automatic rebuilds and deployments
2. **Database Migrations**: Use Flyway or Liquibase for schema management
3. **Monitoring**: Install Prometheus and Grafana for metrics
4. **Tracing**: Use Jaeger for distributed tracing
5. **Security**: Use sealed-secrets for managing secrets in Git