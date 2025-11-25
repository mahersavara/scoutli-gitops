# Scoutli GitOps (ArgoCD)

This repository is the **Source of Truth** for the state of the Scoutli application running on Kubernetes. It follows the **GitOps** pattern using [ArgoCD](https://argo-cd.readthedocs.io/).

## How it works

1.  **GitHub Actions (CI)** builds Docker images and pushes them to ECR.
2.  **GitHub Actions** then updates the `values.yaml` in this repository (specifically the `image.tag` field) with the new commit SHA.
3.  **ArgoCD** (running in the cluster) detects the change in this repository.
4.  **ArgoCD** syncs the state, deploying the new image version to the EKS cluster.

## Directory Structure

We use the **App of Apps** pattern:

```
.
├── bootstrap/          # The "Parent" Application that manages other Apps
│   └── root-app.yaml
├── apps/               # Application definitions
│   ├── scoutli-prod.yaml
│   ├── scoutli-staging.yaml
│   └── ...
└── envs/               # Environment-specific configurations
    ├── prod/
    │   └── values.yaml # The file updated by CI
    └── staging/
        └── values.yaml
```

## ArgoCD Setup

To bootstrap the cluster:

```bash
kubectl apply -f bootstrap/root-app.yaml
```