# goit-argo

GitOps repository watched by ArgoCD. All manifests here are automatically synced to the EKS cluster via an ApplicationSet.

## Repository structure

```
goit-argo/
└── namespaces/
    ├── application/
    │   ├── ns.yaml              # application namespace
    │   ├── nginx.yaml           # nginx Deployment + Service
    │   ├── mlflow.yaml          # MLflow Tracking Server (ClusterIP :5000)
    │   ├── mlflow-postgres.yaml # PostgreSQL for MLflow backend store
    │   ├── minio.yaml           # MinIO object storage (bucket: mlflow-artifacts)
    │   ├── grafana.yaml         # kube-prometheus-stack (Prometheus + Grafana)
    │   ├── loki.yaml            # Loki + Promtail log aggregation
    │   └── pushgateway.yaml     # Prometheus PushGateway (monitoring ns, :9091)
    └── infra-tools/
        └── ns.yaml              # infra-tools namespace
```

## How it works

ArgoCD is deployed to the EKS cluster via Terraform (see [eks-vpc-cluster/argocd](https://github.com/dem-v/goit-mlops/tree/main/eks-vpc-cluster/argocd)).

An `ApplicationSet` with a Git generator watches the `namespaces/` directory. Each subdirectory becomes a separate ArgoCD Application that is automatically synced (prune + selfHeal enabled).

## Verify deployment

```bash
# Check all ArgoCD applications
kubectl get applications -n infra-tools

# Check workloads
kubectl get pods -n application
kubectl get pods -n monitoring    # PushGateway
kubectl get pods -n infra-tools   # Prometheus + Grafana
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

## Access MLflow UI

```bash
kubectl port-forward svc/mlflow -n application 5000:5000
```

Open http://localhost:5000

## Access Grafana UI

```bash
kubectl port-forward svc/prometheus-operator-grafana -n infra-tools 3000:80
```

Open http://localhost:3000 — login with `admin` / `prom-operator`.

To check PushGateway metrics in Grafana: **Explore → Prometheus** and query:
- `mlflow_accuracy`
- `mlflow_loss`

## Access PushGateway

```bash
kubectl port-forward svc/prometheus-pushgateway -n monitoring 9091:9091
```

Open http://localhost:9091
