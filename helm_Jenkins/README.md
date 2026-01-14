# Helm Directory Structure

This directory contains all CI/CD pipeline components organized by functionality.

## Directory Layout

```
helm/
├── jenkins/
│   ├── Jenkinsfile                 # Jenkins pipeline definition
│   └── JENKINS_PIPELINE_GUIDE.md   # Complete setup and usage guide
├── docker/
│   └── Dockerfile                  # Spring Boot Docker image
├── kubernetes/
│   └── kubernetes-config.yaml      # K8s ConfigMap templates
└── charts/
    └── springboot-app/
        ├── Chart.yaml              # Helm chart metadata
        ├── values.yaml             # Default configuration values
        └── templates/
            ├── deployment.yaml     # K8s Deployment resource
            ├── service.yaml        # K8s Service resource
            ├── hpa.yaml            # Horizontal Pod Autoscaler
            ├── serviceaccount.yaml # RBAC Service Account
            ├── _helpers.tpl        # Helm template helpers
            └── NOTES.txt           # Post-install notes
```

## Quick Start

### 1. Configure Jenkins
- Review `/helm/jenkins/JENKINS_PIPELINE_GUIDE.md` for complete setup instructions
- Update credentials in Jenkins with GCP project details

### 2. Build Docker Image
```bash
docker build -t gcr.io/your-project/springboot-app:latest -f helm/docker/Dockerfile .
```

### 3. Deploy with Helm
```bash
helm install springboot-app ./helm/charts/springboot-app \
  --set image.repository=gcr.io/your-project/springboot-app \
  --set image.tag=latest
```

### 4. Verify Deployment
```bash
kubectl get pods -l app=springboot-app
kubectl get svc springboot-app
```

## Directory Descriptions

### `jenkins/`
Contains the Jenkins pipeline definition and documentation.
- **Jenkinsfile**: The main pipeline orchestrating the entire CI/CD workflow
- **JENKINS_PIPELINE_GUIDE.md**: Detailed setup, configuration, and troubleshooting guide

### `docker/`
Docker image configuration for containerizing the Spring Boot application.
- **Dockerfile**: Multi-stage build optimized for size and security

### `kubernetes/`
Raw Kubernetes configuration files (optional ConfigMaps and other resources).
- **kubernetes-config.yaml**: Example ConfigMap templates for Spring Boot configuration

### `charts/`
Helm chart for deploying applications to Kubernetes.
- **Chart.yaml**: Chart metadata and version information
- **values.yaml**: Default values for all chart variables
- **templates/**: Kubernetes resource templates processed by Helm

## Environment Configuration

Update these values in `helm/charts/springboot-app/values.yaml`:

```yaml
replicaCount: 2              # Number of pods
image:
  repository: gcr.io/your-project/springboot-app
  tag: latest
service:
  type: LoadBalancer         # Can be NodePort, ClusterIP
  port: 80
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
```

## Jenkins to Helm Workflow

The pipeline (`helm/jenkins/Jenkinsfile`) performs these steps:

1. **Checkout** → Pulls source code
2. **Build** → Compiles Spring Boot application
3. **Test** → Runs unit tests
4. **Docker** → Builds and pushes image using `helm/docker/Dockerfile`
5. **Validate** → Lints and validates `helm/charts/springboot-app`
6. **Deploy** → Uses Helm to deploy the chart to GKE
7. **Verify** → Confirms deployment success

## Security Features

✓ Non-root container user  
✓ Resource limits and requests  
✓ Liveness and readiness probes  
✓ Pod security context  
✓ RBAC with ServiceAccount  
✓ Health checks  

## Customization

### Change Replicas
```bash
helm upgrade springboot-app ./helm/charts/springboot-app --set replicaCount=3
```

### Change Service Type
```bash
helm upgrade springboot-app ./helm/charts/springboot-app --set service.type=ClusterIP
```

### Scale Automatically
```bash
helm upgrade springboot-app ./helm/charts/springboot-app \
  --set autoscaling.enabled=true \
  --set autoscaling.maxReplicas=10
```

## Troubleshooting

### Validate Helm Chart
```bash
helm lint ./helm/charts/springboot-app
helm template springboot-app ./helm/charts/springboot-app
```

### Check Deployment Status
```bash
kubectl get deployment springboot-app
kubectl describe deployment springboot-app
kubectl logs deployment/springboot-app
```

### Rollback Deployment
```bash
helm rollback springboot-app
```

## Additional Resources

- [Helm Documentation](https://helm.sh/docs/)
- [Spring Boot on Kubernetes](https://spring.io/guides/gs/spring-boot-docker/)
- [GKE Best Practices](https://cloud.google.com/kubernetes-engine/docs/best-practices)
