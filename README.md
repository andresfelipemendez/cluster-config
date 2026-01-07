# Cluster Config

GitOps repository for the e-ink dashboard Kubernetes cluster.

## Status

| Component | Status | URL |
|-----------|--------|-----|
| Kubernetes | ✅ Running | - |
| ingress-nginx | ✅ Running | - |
| cert-manager | ✅ Running | - |
| ArgoCD | ✅ Running | https://argocd.andresmendez.dev |
| Dashboard | ⬜ Not deployed | https://dashboard.andresmendez.dev |

## Cluster Info

| Property | Value |
|----------|-------|
| **IP** | 178.156.213.185 |
| **Domain** | andresmendez.dev |
| **DNS** | Cloudflare |
| **K8s** | kubeadm v1.35 |
| **GitOps** | ArgoCD |
| **TLS** | Let's Encrypt (cert-manager) |
| **Credentials** | 1Password |

## Structure

```
cluster-config/
├── apps/
│   ├── argocd/              # ArgoCD ingress (✅ deployed)
│   │   ├── ingress.yaml
│   │   └── kustomization.yaml
│   └── dashboard/           # E-ink dashboard app (⬜ pending)
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       ├── configmap.yaml
│       └── kustomization.yaml
├── infrastructure/
│   ├── namespaces/          # (✅ deployed)
│   ├── ingress-nginx/       # (✅ deployed)
│   └── cert-manager/        # (✅ deployed)
│       ├── kustomization.yaml
│       └── cluster-issuer.yaml
└── README.md
```

## Bootstrap (from scratch)

1. **Namespaces**
   ```bash
   kubectl apply -k infrastructure/namespaces
   ```

2. **Ingress controller** (uses hostNetwork for ports 80/443)
   ```bash
   kubectl apply -k infrastructure/ingress-nginx
   ```

3. **cert-manager** (CRDs + controller)
   ```bash
   kubectl apply -k infrastructure/cert-manager
   ```

4. **Wait for cert-manager webhook**
   ```bash
   kubectl wait --for=condition=Available deployment/cert-manager -n cert-manager --timeout=120s
   kubectl wait --for=condition=Available deployment/cert-manager-webhook -n cert-manager --timeout=120s
   ```

5. **ClusterIssuers** (must be separate - CRDs need time to register)
   ```bash
   kubectl apply -f infrastructure/cert-manager/cluster-issuer.yaml
   ```

6. **ArgoCD ingress**
   ```bash
   kubectl apply -k apps/argocd
   ```


## ArgoCD Access
Get admin password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

- Username: `admin`
- URL: [argocd.andresmendez.dev](https://argocd.andresmendez.dev)


## Notes

- **ingress-nginx** uses `hostNetwork: true` to bind directly to ports 80/443 (single VPS setup)
- **ClusterIssuer** must be applied separately after cert-manager CRDs are ready
- **Secrets** are not committed - use `secrets.yaml.example` as template
