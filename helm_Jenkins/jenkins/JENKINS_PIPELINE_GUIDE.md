# Jenkins Pipeline for Spring Boot + Helm + GKE

Complete CI/CD pipeline that builds a Spring Boot application, creates Docker images, publishes to Google Container Registry, and deploys to Google Kubernetes Engine using Helm.

## Prerequisites

### 1. GCP Setup
```bash
# Create GCP service account
gcloud iam service-accounts create jenkins-sa --display-name="Jenkins Service Account"

# Grant necessary permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:jenkins-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/container.admin

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:jenkins-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/storage.admin

# Create and download service account key
gcloud iam service-accounts keys create jenkins-key.json \
  --iam-account=jenkins-sa@PROJECT_ID.iam.gserviceaccount.com
```

### 2. GKE Cluster
```bash
# Create GKE cluster
gcloud container clusters create my-gke-cluster \
  --zone us-central1-a \
  --num-nodes 3 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 5

# Create service account for Jenkins
kubectl create serviceaccount jenkins -n kube-system
kubectl create clusterrolebinding jenkins --clusterrole=cluster-admin --serviceaccount=kube-system:jenkins
```

### 3. Jenkins Configuration

#### Install Plugins
- Kubernetes plugin
- Docker Pipeline plugin
- Google Cloud Storage plugin
- Google Kubernetes Engine plugin

#### Configure Credentials
Add the following credentials to Jenkins:

1. **GCP Project ID** (Secret text)
   - ID: `gcp-project-id`
   - Secret: Your GCP project ID

2. **GCP Service Account Key** (Secret file)
   - ID: `gcp-service-account-key`
   - Upload: `jenkins-key.json`

3. **GCP Zone** (Secret text)
   - ID: `gcp-zone`
   - Secret: `us-central1-a`

4. **GKE Cluster Name** (Secret text)
   - ID: `gke-cluster-name`
   - Secret: `my-gke-cluster`

## Pipeline Stages

### 1. **Checkout**
   - Pulls the latest code from your Git repository

### 2. **Build Spring Boot Application**
   - Runs Maven build (or Gradle if configured)
   - Compiles and packages the application

### 3. **Run Tests**
   - Executes unit tests
   - Generates test reports

### 4. **Build Docker Image**
   - Builds Docker image using the Dockerfile
   - Tags with build number

### 5. **Push Image to GCR**
   - Authenticates with Google Container Registry
   - Pushes the Docker image and latest tag

### 6. **Create/Update Helm Chart**
   - Validates the Helm chart
   - Performs dry-run deployment validation

### 7. **Configure GKE Access**
   - Authenticates with GKE cluster
   - Sets up kubectl context

### 8. **Deploy to GKE**
   - Uses Helm to deploy/upgrade the application
   - Sets image repository and tag
   - Waits for deployment to be ready

### 9. **Verify Deployment**
   - Checks rollout status
   - Displays pod and service information

## Project Structure

```
.
├── Jenkinsfile                          # Jenkins pipeline definition
├── Dockerfile                           # Docker image build configuration
├── pom.xml                              # Maven configuration
├── src/
│   ├── main/
│   │   └── java/
│   │   └── resources/
│   │       └── application.properties
│   └── test/
├── helm/
│   └── springboot-app/
│       ├── Chart.yaml                   # Helm chart metadata
│       ├── values.yaml                  # Default values
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── hpa.yaml                 # Horizontal Pod Autoscaler
│           ├── serviceaccount.yaml
│           ├── _helpers.tpl
│           └── NOTES.txt
└── README.md
```

## Dockerfile Details

- **Multi-stage build** for optimized image size
- Base image: `openjdk:17-slim`
- Non-root user for security
- Health checks enabled
- Memory limits: 256MB min / 512MB max

## Helm Chart Details

### Key Features
- **Auto-scaling**: HPA configured for CPU and memory metrics
- **Health Checks**: Liveness and readiness probes configured
- **Security**: Pod security context, non-root user, read-only filesystem
- **Resource Limits**: CPU and memory requests/limits
- **Service Type**: LoadBalancer (can be changed to NodePort/ClusterIP)

### Values Configuration
Edit `helm/springboot-app/values.yaml` to customize:
- Number of replicas
- Resource limits
- Environment variables
- Ingress settings
- Auto-scaling thresholds

## Customization Guide

### 1. Update Image Registry
Edit `Jenkinsfile` environment variables:
```groovy
DOCKER_REGISTRY = 'gcr.io'
IMAGE_NAME = "${DOCKER_REGISTRY}/${GCP_PROJECT_ID}/springboot-app"
```

### 2. Change GKE Cluster Settings
Update environment variables:
```groovy
GCP_ZONE = 'us-central1-a'
GKE_CLUSTER_NAME = 'my-gke-cluster'
```

### 3. Customize Helm Deployment
Update `helm/springboot-app/values.yaml`:
```yaml
replicaCount: 3
service:
  type: ClusterIP
  port: 8080
autoscaling:
  minReplicas: 2
  maxReplicas: 10
```

### 4. Update Spring Boot Health Check
If your app doesn't use Spring Boot Actuator, update the probe paths:
```yaml
livenessProbe:
  httpGet:
    path: /health  # Your custom health endpoint
    port: 8080
```

## Deployment Instructions

### 1. Create Jenkins Pipeline Job
- Create new Pipeline job in Jenkins
- Point to your Git repository
- Select "Pipeline script from SCM"
- Set Script Path to `Jenkinsfile`

### 2. Run the Pipeline
```bash
# Trigger manually or via webhook
# Monitor build logs in Jenkins
```

### 3. Access Deployed Application
```bash
# Get LoadBalancer IP
kubectl get svc springboot-app
export EXTERNAL_IP=$(kubectl get svc springboot-app --template="{.status.loadBalancer.ingress[0].ip}")
echo http://$EXTERNAL_IP:80
```

## Monitoring & Troubleshooting

### View Deployment Status
```bash
kubectl get deployment springboot-app
kubectl get pods -l app=springboot-app
```

### Check Pod Logs
```bash
kubectl logs -f deployment/springboot-app
```

### Describe Pod for Issues
```bash
kubectl describe pod <pod-name>
```

### Rollback Deployment
```bash
# Via Helm
helm rollback springboot-app

# Or via kubectl
kubectl rollout undo deployment/springboot-app
```

### Check HPA Status
```bash
kubectl get hpa springboot-app
kubectl describe hpa springboot-app
```

## Security Best Practices Implemented

✓ Non-root user execution
✓ Read-only filesystem
✓ Resource limits and requests
✓ Security context with dropped capabilities
✓ Health checks for availability
✓ Pod security policies via templates
✓ Service account RBAC
✓ Image pull secrets support

## Environment Variables

Configure these in the Jenkinsfile or Jenkins Credentials:

| Variable | Description | Default |
|----------|-------------|---------|
| `GCP_PROJECT_ID` | GCP Project ID | - |
| `GCP_ZONE` | GKE Zone | us-central1-a |
| `GKE_CLUSTER_NAME` | Cluster Name | my-gke-cluster |
| `IMAGE_NAME` | Docker image name | gcr.io/$PROJECT_ID/springboot-app |
| `APP_NAME` | Application name | springboot-app |
| `NAMESPACE` | K8s namespace | default |

## Troubleshooting Common Issues

### Build Fails with Maven Errors
- Ensure `mvnw` or `mvn` is in project root
- Check Java version (minimum 11, tested with 17)
- Verify dependencies are accessible

### Docker Push Fails
- Verify gcloud authentication: `gcloud auth configure-docker`
- Check GCP service account has Container Registry write permissions
- Ensure GCR is enabled in your GCP project

### Helm Deployment Fails
- Validate helm chart: `helm lint helm/springboot-app`
- Check kubectl access to cluster
- Review image tag and repository in values

### Pods Not Starting
- Check pod events: `kubectl describe pod <pod-name>`
- Verify image exists in GCR
- Check resource requests don't exceed node capacity
- Review application logs

## Additional Resources

- [Jenkins Kubernetes Plugin](https://plugins.jenkins.io/kubernetes/)
- [Helm Documentation](https://helm.sh/docs/)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs)
- [Spring Boot on Kubernetes](https://spring.io/blog/2021/06/21/spring-boot-docker)
