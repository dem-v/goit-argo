# goit-argo

GitOps repository watched by ArgoCD. All manifests here are automatically synced to the EKS cluster via an ApplicationSet.

## Repository structure

```
goit-argo/
├── namespaces/
│   ├── application/
│   │   ├── ns.yaml        # application namespace
│   │   └── nginx.yaml     # nginx Deployment + Service
│   └── infra-tools/
│       └── ns.yaml        # infra-tools namespace
└── apps/
    └── mlflow/
        ├── application.yaml   # ArgoCD Application (Bitnami MLflow chart)
        └── values.yaml        # Helm values override
```

## How it works

ArgoCD is deployed to the EKS cluster via Terraform (see [eks-vpc-cluster/argocd](https://github.com/dem-v/goit-mlops/tree/main/eks-vpc-cluster/argocd)).

An `ApplicationSet` with a Git generator watches the `namespaces/` directory. Each subdirectory becomes a separate ArgoCD Application that is automatically synced (prune + selfHeal enabled).

## Verify deployment

```bash
# Check ArgoCD applications
kubectl get applications -n infra-tools

# Check workloads in application namespace
kubectl get pods -n application

# Check MLflow
kubectl get pods -n application -l app.kubernetes.io/name=mlflow
```

## Access ArgoCD UI

```bash
# Get admin password
kubectl -n infra-tools get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Open tunnel
kubectl port-forward svc/argocd-server -n infra-tools 8080:80
```

Open http://localhost:8080 — login with `admin` and the password from above.
