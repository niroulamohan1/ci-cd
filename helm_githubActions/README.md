# GitHub Actions + Helm Deployment

Simple, streamlined setup for deploying Spring Boot applications to GKE using GitHub Actions and Helm.

## Directory Structure

```
helm_githubActions/
├── Dockerfile                          # Docker image for Spring Boot
├── workflows/
│   └── deploy.yaml                     # GitHub Actions workflow
└── charts/
    └── springboot-app/
        ├── Chart.yaml                  # Helm chart metadata
        ├── values.yaml                 # Configuration values
        └── templates/
            ├── deployment.yaml         # K8s Deployment
            ├── service.yaml            # K8s Service
            ├── hpa.yaml                # Horizontal Pod Autoscaler
            └── serviceaccount.yaml     # Service Account
```

## Setup

### 1. GitHub Secrets

Add these secrets to your GitHub repository settings:

| Secret | Value |
|--------|-------|
| `GCP_SA_KEY` | GCP Service Account JSON key |
| `GCP_PROJECT_ID` | Your GCP project ID |
| `GCP_REGION` | GKE region (e.g., us-central1) |
| `GKE_CLUSTER_NAME` | Your GKE cluster name |
| `DOCKER_IMAGE` | GCR image URL (e.g., gcr.io/project/springboot-app) |

### 2. Workflow Activation

The workflow runs automatically on push to `main` branch. To enable manual trigger, add:

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:  # Manual trigger
```

## Features

✅ Maven build for Spring Boot  
✅ Multi-stage Docker build  
✅ Push to Google Container Registry  
✅ Helm deployment to GKE  
✅ Auto-scaling with HPA  
✅ Health checks (liveness & readiness)  
✅ Security: non-root user, read-only filesystem  

## Customization

### Change Image Tag

In `workflows/deploy.yaml`:
```yaml
--set image.tag=${{ github.ref_name }}  # Use branch name
```

### Enable Dry-Run

```yaml
- name: Helm Dry Run
  run: |
    helm template springboot-app ./helm_githubActions/charts/springboot-app \
      --set image.repository=${{ secrets.DOCKER_IMAGE }} \
      --set image.tag=${{ github.sha }}
```

### Change Replicas

```yaml
--set replicaCount=3
```

### Disable Auto-scaling

```yaml
--set autoscaling.enabled=false
```

## Troubleshooting

### Build fails
- Check Java version in pom.xml
- Verify Maven dependencies are accessible

### Docker push fails
- Verify `GCP_SA_KEY` contains valid JSON
- Check service account has Container Registry role

### Helm deployment fails
- Validate chart: `helm lint ./helm_githubActions/charts/springboot-app`
- Check GKE cluster accessibility
- Verify image exists in GCR

### Pod not starting
```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

## Deployment

The workflow:
1. Checkouts code
2. Builds Spring Boot app with Maven
3. Creates Docker image and pushes to GCR
4. Authenticates with GKE cluster
5. Deploys/upgrades using Helm
6. Verifies deployment

Access your app:
```bash
kubectl get svc springboot-app
# Get external IP and access on port 80
```
