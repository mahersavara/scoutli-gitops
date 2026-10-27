# Scoutli GitOps (ArgoCD)

This repository is the **Source of Truth** for the state of the Scoutli application running on Kubernetes. It follows the **GitOps** pattern using [ArgoCD](https://argo-cd.readthedocs.io/).

## How it works

1.  **GitHub Actions (CI)** builds Docker images and pushes them to ECR.
2.  **GitHub Actions** then updates the environment values file in this repository (the `image.tag` field) with the new commit SHA.
3.  **ArgoCD** (running in the cluster) detects the change in this repository.
4.  **ArgoCD** syncs the state, deploying the new image version to the EKS cluster.

## Directory Structure

We use the **App of Apps** pattern:

```
.
├── root-app.yaml               # Parent Application that manages everything in apps/
├── apps/                       # One ArgoCD Application per service, plus ingresses
│   ├── scoutli-auth.yaml
│   ├── scoutli-discovery.yaml
│   ├── scoutli-interaction.yaml
│   ├── scoutli-frontend.yaml
│   ├── ingress-frontend.yaml
│   └── ingress-argocd.yaml
├── envs/
│   └── dev/
│       ├── backend-values.yaml  # Image tag updated by CI
│       └── frontend-values.yaml
├── bootstrap/                   # Applied once, by hand
│   ├── root-app.yaml
│   ├── cluster-issuer.yaml      # cert-manager ClusterIssuer
│   ├── db-init-job.yaml         # Creates per-service databases on RDS
│   └── db-secret.example.yaml   # Template only, see "Secrets" below
└── configs/
    └── keycloak/scoutli-realm.json  # Demo realm (dev-only credentials)
```

## ArgoCD Setup

To bootstrap the cluster:

```bash
kubectl apply -f bootstrap/root-app.yaml
```

## Secrets

Real credentials are never committed. Create the database secret locally from the template:

```bash
cp bootstrap/db-secret.example.yaml bootstrap/db-secret.yaml   # git-ignored
# fill in DB_HOST / DB_PASSWORD, then:
kubectl apply -f bootstrap/db-secret.yaml
```

For a longer-lived setup, use [External Secrets Operator](https://external-secrets.io/) with AWS Secrets Manager or Sealed Secrets instead of a manual step.

The Keycloak realm export contains **dev-only demo users and client secrets**. Override them before using the realm anywhere real.
