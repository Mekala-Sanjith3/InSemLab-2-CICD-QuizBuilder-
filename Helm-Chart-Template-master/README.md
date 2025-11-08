# Quiz Builder Application - Kubernetes Helm Chart

This Helm chart deploys a full-stack Quiz Builder application with a Spring Boot backend, React frontend, and MySQL database on Kubernetes.

## Architecture Overview

The application consists of three main components:

1. **Frontend**: React application served by Nginx
2. **Backend**: Spring Boot REST API application
3. **Database**: MySQL 8.0 for data persistence

## Prerequisites

- Kubernetes cluster (1.19+)
- Helm 3.0+
- Docker registry access
- Ingress controller (nginx recommended)

## Components Configuration

### Backend (Spring Boot)
- **Port**: 9096
- **Technology**: Spring Boot with embedded Tomcat
- **Database**: MySQL connection configured
- **Scaling**: HPA enabled (2-5 replicas)

### Frontend (React + Nginx)
- **Port**: 80
- **Technology**: React build served by Nginx
- **API Integration**: Connected to backend via `/api` endpoint
- **Scaling**: HPA enabled (2-5 replicas)

### Database (MySQL)
- **Port**: 3306
- **Version**: MySQL 8.0
- **Storage**: Persistent Volume Claim (5Gi)
- **Backup**: Data persisted across pod restarts

## Quick Start

### 1. Build Docker Images

```bash
# Build backend image
docker build -f Helm-Chart-Template-master/Dockerfile.backend -t quiz-builder-backend:latest .

# Build frontend image  
docker build -f Helm-Chart-Template-master/Dockerfile.frontend -t quiz-builder-frontend:latest .
```

### 2. Deploy to Kubernetes

```bash
# Install the Helm chart
helm install quiz-builder Helm-Chart-Template-master

# Check deployment status
kubectl get pods
kubectl get services
kubectl get ingress
```

### 3. Access the Application

Add the following to your `/etc/hosts` file:
```
127.0.0.1 quiz-builder.local
```

Access the application at: `http://quiz-builder.local`

## Configuration

### Values.yaml Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `mysql.image` | MySQL Docker image | `mysql:8.0` |
| `mysql.storage` | Storage size for MySQL | `5Gi` |
| `mysql.rootPassword` | MySQL root password | `root` |
| `mysql.database` | Database name | `quizbuilder` |
| `backend.image` | Backend Docker image | `quiz-builder-backend:latest` |
| `backend.port` | Backend service port | `9096` |
| `backend.replicas` | Initial backend replicas | `2` |
| `frontend.image` | Frontend Docker image | `quiz-builder-frontend:latest` |
| `frontend.port` | Frontend service port | `80` |
| `frontend.replicas` | Initial frontend replicas | `2` |
| `ingress.enabled` | Enable Ingress | `true` |
| `ingress.host` | Ingress hostname | `quiz-builder.local` |
| `autoscaling.backend.enabled` | Enable backend HPA | `true` |
| `autoscaling.backend.minReplicas` | Minimum backend replicas | `2` |
| `autoscaling.backend.maxReplicas` | Maximum backend replicas | `5` |
| `autoscaling.frontend.enabled` | Enable frontend HPA | `true` |
| `autoscaling.frontend.minReplicas` | Minimum frontend replicas | `2` |
| `autoscaling.frontend.maxReplicas` | Maximum frontend replicas | `5` |

### Custom Configuration

Create a custom values file:
```yaml
# custom-values.yaml
mysql:
  rootPassword: your-secure-password
  storage: 10Gi

backend:
  replicas: 3
  
frontend:
  replicas: 3

ingress:
  host: your-domain.com
```

Deploy with custom values:
```bash
helm install quiz-builder Helm-Chart-Template-master -f custom-values.yaml
```

## Monitoring and Scaling

### Horizontal Pod Autoscaler (HPA)

The application automatically scales based on CPU utilization:
- **Target CPU**: 60%
- **Backend**: 2-5 replicas
- **Frontend**: 2-5 replicas

Check HPA status:
```bash
kubectl get hpa
```

### Health Checks

**Backend Health Check**:
```bash
curl http://backend:9096/actuator/health
```

**Frontend Health Check**:
```bash
curl http://frontend/
```

**Database Health Check**:
```bash
kubectl exec -it mysql-pod -- mysqladmin ping
```

## Troubleshooting

### Common Issues

1. **Pods not starting**
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

2. **Database connection issues**
   ```bash
   kubectl exec -it backend-pod -- curl mysql:3306
   ```

3. **Ingress not working**
   ```bash
   kubectl describe ingress quiz-builder-ingress
   ```

4. **Scaling issues**
   ```bash
   kubectl describe hpa backend-hpa
   kubectl describe hpa frontend-hpa
   ```

### Logs and Debugging

View application logs:
```bash
# Backend logs
kubectl logs -f deployment/backend

# Frontend logs  
kubectl logs -f deployment/frontend

# Database logs
kubectl logs -f deployment/mysql
```

## Security Considerations

- Change default MySQL passwords in production
- Use secrets for sensitive configuration
- Enable TLS for Ingress in production
- Configure network policies as needed
- Regular security updates for base images

## Maintenance

### Upgrading

```bash
# Update Helm repository
helm repo update

# Upgrade deployment
helm upgrade quiz-builder Helm-Chart-Template-master
```

### Backup and Restore

**Database Backup**:
```bash
kubectl exec -it mysql-pod -- mysqldump -u root -p quizbuilder > backup.sql
```

**Database Restore**:
```bash
kubectl exec -i mysql-pod -- mysql -u root -p quizbuilder < backup.sql
```

### Cleanup

```bash
# Uninstall the Helm release
helm uninstall quiz-builder

# Delete persistent volumes (optional)
kubectl delete pvc mysql-pvc
```

## Support

For issues and questions:
1. Check the troubleshooting section
2. Review application logs
3. Verify Kubernetes cluster health
4. Check resource availability and limits