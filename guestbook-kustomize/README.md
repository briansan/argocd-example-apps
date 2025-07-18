# Guestbook Kustomize Configuration

This directory contains a Kustomize-based configuration for the guestbook application that can be deployed using ArgoCD across multiple environments (dev and prod).

## Directory Structure

```
guestbook-kustomize/
├── base/                          # Base Kubernetes manifests
│   ├── deployment.yaml            # Base deployment configuration
│   ├── service.yaml               # Base service configuration
│   └── kustomization.yaml         # Base kustomization
├── overlays/                      # Environment-specific overlays
│   ├── dev/                       # Development environment
│   │   └── kustomization.yaml     # Dev-specific customizations
│   └── prod/                      # Production environment
│       └── kustomization.yaml     # Prod-specific customizations
├── argocd-apps/                   # ArgoCD Application manifests
│   ├── guestbook-dev.yaml         # ArgoCD app for dev environment
│   └── guestbook-prod.yaml        # ArgoCD app for prod environment
└── README.md                      # This file
```

## Environment Configurations

### Base Configuration
- **Replicas:** 2
- **Image:** `gcr.io/google-samples/gb-frontend:v5`
- **Resources:** 
  - Requests: 64Mi memory, 50m CPU
  - Limits: 128Mi memory, 100m CPU

### Development Environment
- **Namespace:** `guestbook-dev`
- **Replicas:** 1
- **Name Prefix:** `dev-`
- **Resources:**
  - Requests: 32Mi memory, 25m CPU
  - Limits: 64Mi memory, 50m CPU

### Production Environment
- **Namespace:** `guestbook-prod`
- **Replicas:** 3
- **Name Prefix:** `prod-`
- **Resources:**
  - Requests: 128Mi memory, 100m CPU
  - Limits: 256Mi memory, 200m CPU

## Testing the Configuration

Test the Kustomize configurations locally:

```bash
# Test dev environment
kubectl kustomize overlays/dev

# Test prod environment
kubectl kustomize overlays/prod
```

## Deployment with ArgoCD

### Prerequisites
1. ArgoCD installed and running in your cluster
2. This repository accessible to ArgoCD
3. Update the `repoURL` in the ArgoCD Application manifests to point to your repository

### Deploy Applications

1. **Update Repository URL:**
   Edit the ArgoCD Application manifests in `argocd-apps/` and replace `https://github.com/your-org/argocd-example-apps` with your actual repository URL.

2. **Apply ArgoCD Applications:**
   ```bash
   # Deploy dev environment
   kubectl apply -f argocd-apps/guestbook-dev.yaml
   
   # Deploy prod environment
   kubectl apply -f argocd-apps/guestbook-prod.yaml
   ```

3. **Monitor Deployments:**
   ```bash
   # Check application status
   argocd app list
   
   # Sync if needed
   argocd app sync guestbook-dev
   argocd app sync guestbook-prod
   ```

## Manual Deployment (without ArgoCD)

You can also deploy directly using kubectl:

```bash
# Deploy to dev environment
kubectl apply -k overlays/dev

# Deploy to prod environment
kubectl apply -k overlays/prod
```

## Accessing the Application

After deployment, the guestbook UI will be available through the services:

- **Dev:** `dev-guestbook-ui` service in `guestbook-dev` namespace
- **Prod:** `prod-guestbook-ui` service in `guestbook-prod` namespace

To access the application, you can port-forward:

```bash
# Access dev environment
kubectl port-forward -n guestbook-dev svc/dev-guestbook-ui 8080:80

# Access prod environment
kubectl port-forward -n guestbook-prod svc/prod-guestbook-ui 8080:80
```

Then visit `http://localhost:8080` in your browser.

## Customization

To customize for your environment:

1. **Modify base configuration:** Edit files in `base/` directory
2. **Adjust environment-specific settings:** Modify `kustomization.yaml` files in overlay directories
3. **Add new environments:** Create new overlay directories following the same pattern
4. **Update ArgoCD configuration:** Modify sync policies, retry logic, etc. in ArgoCD Application manifests

## Features

- ✅ Environment isolation with separate namespaces
- ✅ Resource optimization per environment
- ✅ Automated ArgoCD sync with retry logic
- ✅ Proper labeling and naming conventions
- ✅ GitOps-ready configuration
- ✅ Easy to extend for additional environments 