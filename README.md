# Python FastAPI Application - Kubernetes Deployment

This directory contains Kubernetes manifests for deploying the Python FastAPI application.

## 📋 Overview

Our Kubernetes deployment includes:
- **Deployment**: Main application deployment with security best practices
- **Service**: ClusterIP service for internal communication
- **HPA**: Horizontal Pod Autoscaler for automatic scaling
- **Ingress**: Nginx ingress for external access
- **ConfigMap**: Application configuration

## 🚀 Key Features

### ✅ **Production-Ready Configuration**
- **Multi-replica deployment** (3 replicas by default)
- **Resource limits and requests** optimized for FastAPI
- **Health checks** using `/health` endpoint from the application
- **Security context** with non-root user and dropped capabilities

### ✅ **Automatic Scaling**
- **HPA** scales based on CPU (70%) and memory (80%) utilization
- Scales between 2-10 replicas
- Smart scaling policies to prevent flapping

### ✅ **Health Monitoring**
- **Startup Probe**: Ensures app starts properly (grace period up to 100 seconds)
- **Readiness Probe**: Checks if app can accept traffic
- **Liveness Probe**: Ensures app is healthy, restarts if unhealthy

### ✅ **Network Configuration**
- **Port 8000** for the Python application (internal)
- **Port 80** exposed via Service and Ingress
- **NGINX Ingress** configured for external access

## 📁 File Structure

```
base/
├── kustomization.yaml     # Main kustomization file
├── deployment.yaml        # Deployment and Service
├── configmap.yaml        # Environment configuration
├── hpa.yaml             # Horizontal Pod Autoscaler
└── ingress.yaml         # NGINX Ingress configuration
```

## 🔧 Usage Instructions

### **1. Prerequisites**
- Kubernetes cluster running
- NGINX Ingress Controller installed
- Docker image built and pushed to registry

### **2. Update Image Reference**
Update the image reference in `base/kustomization.yaml`:
```yaml
images:
  - name: vidal777/python-api
    newTag: your-version-tag  # e.g., v1.0.0, latest
```

### **3. Configure Domain**
Update the host in `base/ingress.yaml`:
```yaml
spec:
  rules:
  - host: your-domain.com  # Change python-api.local
```

### **4. Deploy**
```bash
# Build and apply kustomization
kustomize build . | kubectl apply -f -

# Or apply directly
kubectl apply -k .
```

### **5. Verify Deployment**
```bash
# Check deployment status
kubectl get deployments,pods,services,ingress

# Check logs
kubectl logs -l app=python-api

# Test health endpoint
kubectl port-forward svc/python-api-service 8080:80
curl http://localhost:8080/health
```

## 🔍 Configuration Details

### **Resource Allocation**
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"
```

### **Probing Strategy**
- **Startup**: 10s delay, 10s interval, 10 max failures (100s total)
- **Readiness**: 10s delay, 5s interval, 3 max failures
- **Liveness**: 30s delay, 10s interval, 3 max failures

### **Security Context**
```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: false  # FastAPI needs write access
  runAsNonRoot: true
  runAsUser: 1000  # Matches Dockerfile appuser
  capabilities:
    drop: [ALL]
```

### **Environment Variables**
```yaml
env:
  - name: PORT
    value: "8000"
  - name: PYTHONDONTWRITEBYTECODE
    value: "1"
  - name: PYTHONUNBUFFERED
    value: "1"
```

## 🌐 Access Methods

### **Via Port Forwarding**
```bash
kubectl port-forward svc/python-api-service 8080:80
curl http://localhost:8080/health
curl http://localhost:8080/docs  # Interactive API docs
```

### **Via Ingress**
If you configured a domain:
```bash
curl http://your-domain.com/health
curl http://your-domain.com/docs  # Swagger UI
curl http://your-domain.com/redoc # ReDoc
```

### **API Endpoints**
- `GET /` - Root endpoint with API info
- `GET /health` - Health check endpoint
- `GET /docs` - Swagger UI documentation
- `GET /redoc` - ReDoc documentation
- `GET /items` - List all items
- `POST /items` - Create new item
- `GET /items/{id}` - Get specific item
- `PUT /items/{id}` - Update item
- `DELETE /items/{id}` - Delete item

## 📊 Monitoring

### **Pod Status**
```bash
kubectl get pods -l app=python-api
kubectl describe pod <pod-name>
```

### **Service Status**
```bash
kubectl get svc python-api-service
kubectl describe svc python-api-service
```

### **HPA Status**
```bash
kubectl get hpa python-api-hpa
kubectl describe hpa python-api-hpa
```

### **Logs**
```bash
# All pods
kubectl logs -l app=python-api

# Specific pod
kubectl logs <pod-name>

# Follow logs
kubectl logs -f <pod-name>
```

## 🔄 Updating the Application

### **Rolling Update**
```bash
# Update image tag
kubectl set image deployment/python-api-app python-api=vidal777/python-api:v1.1.0

# Check rollout status
kubectl rollout status deployment/python-api-app

# Rollback if needed
kubectl rollout undo deployment/python-api-app
```

### **Configuration Updates**
```bash
# Update ConfigMap
kubectl apply -f configmap.yaml

# Restart deployment to pick up new config
kubectl rollout restart deployment/python-api-app
```

## 🚨 Troubleshooting

### **Common Issues**

1. **Pods not starting**
   ```bash
   kubectl describe pod <pod-name>
   kubectl logs <pod-name>
   ```

2. **Service not accessible**
   ```bash
   kubectl get endpoints python-api-service
   kubectl describe svc python-api-service
   ```

3. **Health checks failing**
   ```bash
   kubectl logs <pod-name>
   # Check if /health endpoint is working
   ```

4. **Image pull errors**
   ```bash
   # Check image exists and is accessible
   docker pull vidal777/python-api:latest
   ```

5. **Ingress not working**
   ```bash
   kubectl get ingress python-api-ingress
   kubectl describe ingress python-api-ingress
   ```

## 🔗 Related Files

- **Docker Image**: Build from `skeleton/Dockerfile`
- **Application Code**: `skeleton/main.py`
- **CI/CD Pipeline**: `skeleton/.github/workflows/ci-cd.yml`

This deployment provides a production-ready setup for your Python FastAPI application with monitoring, scaling, and security best practices! 🎉
