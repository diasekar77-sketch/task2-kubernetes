# Task 2: Kubernetes Deployment

**Analyst:** Divya K S  
**Email:** divyasekar4428@gmail.com  
**Date:** 2025-10-20

## Overview
This task involves containerizing the Java application from Task 1 and deploying it to a Kubernetes cluster with MongoDB. The application is modified to execute shell commands in Kubernetes pods using the Kubernetes API.

## Architecture
- **Application Pod**: Spring Boot application with Kubernetes client
- **MongoDB Pod**: MongoDB database with persistent storage
- **Services**: LoadBalancer for application, ClusterIP for MongoDB
- **Persistent Volume**: For MongoDB data persistence

## Prerequisites
- Docker installed and running
- Kubernetes cluster (Docker Desktop, Minikube, Kind, or cloud provider)
- kubectl configured to access the cluster

## Project Structure
```
task2-kubernetes/
├── k8s/
│   ├── mongodb-deployment.yaml
│   └── app-deployment.yaml
├── Dockerfile
└── README.md
```

## Setup Instructions

### 1. Build Docker Image
```bash
# Navigate to the Java application directory
cd ../task1-java-backend

# Build the application
mvn clean package -DskipTests

# Build Docker image
docker build -t task-management-api:latest .

# Tag for Kubernetes
docker tag task-management-api:latest task-management-api:latest
```

### 2. Deploy MongoDB
```bash
# Apply MongoDB deployment
kubectl apply -f k8s/mongodb-deployment.yaml

# Wait for MongoDB to be ready
kubectl wait --for=condition=ready pod -l app=mongodb --timeout=300s
```

### 3. Deploy Application
```bash
# Apply application deployment
kubectl apply -f k8s/app-deployment.yaml

# Wait for application to be ready
kubectl wait --for=condition=ready pod -l app=task-management-api --timeout=300s
```

### 4. Verify Deployment
```bash
# Check pods
kubectl get pods

# Check services
kubectl get services

# Check persistent volumes
kubectl get pv,pvc
```

## Kubernetes Manifests

### MongoDB Deployment (`k8s/mongodb-deployment.yaml`)
- **Deployment**: MongoDB with persistent storage
- **Service**: ClusterIP service for internal communication
- **PVC**: Persistent Volume Claim for data persistence
- **Environment Variables**: MongoDB authentication

### Application Deployment (`k8s/app-deployment.yaml`)
- **Deployment**: Spring Boot application
- **Service**: LoadBalancer for external access
- **Environment Variables**: MongoDB connection details
- **Kubernetes Client**: For pod creation and management

## Key Features

### 1. Persistent Storage
- MongoDB data persists across pod restarts
- Uses PersistentVolumeClaim for storage
- Data survives pod deletions

### 2. Environment Configuration
- MongoDB connection via environment variables
- Configurable database credentials
- Service discovery through Kubernetes DNS

### 3. Kubernetes Integration
- Application creates pods for command execution
- Uses busybox image for command execution
- Proper resource management and cleanup

## Testing the Deployment

### 1. Check Application Health
```bash
# Get the external IP
kubectl get service task-management-service

# Test the application
curl http://<EXTERNAL-IP>:8080/api/tasks
```

### 2. Create and Execute Tasks
```bash
# Create a task
curl -X PUT http://<EXTERNAL-IP>:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "id": "test-123",
    "name": "Test Task",
    "owner": "Kubernetes User",
    "command": "echo Hello from Kubernetes!"
  }'

# Execute the task
curl -X PUT http://<EXTERNAL-IP>:8080/api/tasks/test-123/execute
```

### 3. Verify Data Persistence
```bash
# Delete MongoDB pod
kubectl delete pod -l app=mongodb

# Wait for pod to restart
kubectl wait --for=condition=ready pod -l app=mongodb --timeout=300s

# Check if data persists
curl http://<EXTERNAL-IP>:8080/api/tasks
```

## Screenshots
![task2](https://github.com/user-attachments/assets/1bb3bef7-a5e9-4707-bb24-0f8492759f3f)
![task2_2](https://github.com/user-attachments/assets/c0be93f5-85e2-49a2-998b-0069c2149b18)
![task2_3](https://github.com/user-attachments/assets/d2eb9bb8-4aad-4f8e-9d5d-e7ac454e39c2)

![task2_4](https://github.com/user-attachments/assets/468417d6-21ec-4a3b-8a69-3a6e5f8980e3)

## Monitoring and Debugging

### 1. View Logs
```bash
# Application logs
kubectl logs -l app=task-management-api

# MongoDB logs
kubectl logs -l app=mongodb
```

### 2. Check Resource Usage
```bash
# Pod resource usage
kubectl top pods

# Node resource usage
kubectl top nodes
```

### 3. Debug Pod Issues
```bash
# Describe pod for events
kubectl describe pod <pod-name>

# Check pod status
kubectl get pods -o wide
```

## Security Considerations

### 1. Network Policies
- Application and MongoDB communicate within cluster
- External access only through LoadBalancer service
- No direct MongoDB access from outside

### 2. Resource Limits
- CPU and memory limits for pods
- Persistent volume size limits
- Proper resource requests and limits

### 3. RBAC Configuration
- Service account for application
- Minimal required permissions
- Secure pod creation permissions

## Troubleshooting

### Common Issues

1. **Pod Creation Failed**
   ```bash
   # Check RBAC permissions
   kubectl auth can-i create pods --as=system:serviceaccount:default:default
   
   # Check service account
   kubectl get serviceaccount
   ```

2. **MongoDB Connection Failed**
   ```bash
   # Check MongoDB service
   kubectl get service mongodb-service
   
   # Test connectivity
   kubectl run test-pod --image=busybox --rm -it -- nslookup mongodb-service
   ```

3. **Persistent Volume Issues**
   ```bash
   # Check PVC status
   kubectl get pvc
   
   # Check PV status
   kubectl get pv
   ```

### Debug Commands
```bash
# Get all resources
kubectl get all

# Describe specific resource
kubectl describe <resource-type> <resource-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Performance Optimization

### 1. Resource Tuning
- Adjust CPU and memory limits
- Optimize JVM settings
- Configure connection pooling

### 2. Scaling
- Horizontal Pod Autoscaler
- Vertical Pod Autoscaler
- Cluster autoscaling

### 3. Monitoring
- Prometheus metrics
- Grafana dashboards
- Alerting rules

## Cleanup
```bash
# Delete application
kubectl delete -f k8s/app-deployment.yaml

# Delete MongoDB
kubectl delete -f k8s/mongodb-deployment.yaml

# Clean up persistent volumes (optional)
kubectl delete pvc mongodb-pvc
```

## Future Enhancements
- **Helm Charts**: Package management
- **Ingress Controller**: Advanced routing
- **Service Mesh**: Istio integration
- **Monitoring**: Prometheus and Grafana
- **Logging**: ELK stack integration
- **Security**: Network policies and RBAC
