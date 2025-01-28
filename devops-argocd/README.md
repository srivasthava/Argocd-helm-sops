# ArgoCD App of Apps with External Secrets

## Prerequisites

1. ArgoCD installed in your cluster (v2.4.0+ for multiple source support)
2. SOPS installed locally
3. GPG key or Azure Key Vault configured
4. kubectl configured for your cluster
5. Access to both application and secrets repositories

## Repository Structure

```
Application Repository:
.
├── apps/
│   ├── root-app.yaml
│   └── templates/
│       └── app-template.yaml
└── helm/
    └── charts/
        └── [app-name]/
            ├── Chart.yaml
            ├── values.yaml
            └── templates/

Secrets Repository:
.
└── environments/
    └── prod/
        ├── frontend/
        │   └── secrets.enc.yaml
        ├── backend/
        │   └── secrets.enc.yaml
        ├── auth/
        │   └── secrets.enc.yaml
        ├── cache/
        │   └── secrets.enc.yaml
        └── monitoring/
            └── secrets.enc.yaml
```

## Setup Instructions

1. Configure SOPS as before (GPG or Azure Key Vault)

2. Add repository credentials to ArgoCD:
   ```bash
   # For application repository
   kubectl create secret generic app-repo-secret \
     --from-literal=username=git-user \
     --from-literal=password=git-pat \
     -n argocd

   # For secrets repository
   kubectl create secret generic secrets-repo-secret \
     --from-literal=username=git-user \
     --from-literal=password=git-pat \
     -n argocd
   ```

3. Configure repositories in ArgoCD:
   ```bash
   argocd repo add https://github.com/your-org/your-gitops-repo --username git-user --password git-pat
   argocd repo add https://github.com/your-org/secrets-repo --username git-user --password git-pat
   ```

4. Deploy root application:
   ```bash
   kubectl apply -f apps/root-app.yaml
   ```

## Important Notes

1. Each application's secrets are stored in a separate path in the secrets repository
2. The ApplicationSet template uses multiple sources to reference both app and secrets repos
3. Ensure proper access controls on the secrets repository
4. Update image tags in the ApplicationSet template when deploying new versions
5. ArgoCD v2.4.0+ is required for multiple source support