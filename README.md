# Cluster Config

GitOps repository for my personal Kubernetes cluster.

## Status

| Component | Status | URL |
|-----------|--------|-----|
| Kubernetes | ✅ Running | - |
| ingress-nginx | ✅ Running | - |
| cert-manager | ✅ Running | - |
| ArgoCD | ✅ Running | https://argocd.andresfelipemendez.com |
| Garden | ✅ Running | https://garden.andresfelipemendez.com |
| Dashboard | ✅ Running | https://dashboard.andresfelipemendez.com |
| E-ink Dashboard | ✅ Running | https://eink.andresfelipemendez.com |
| Castercat | ✅ Running | https://castercatcrew.com |

## Cluster Info

| Property | Value |
|----------|-------|
| **IP** | 178.156.213.185 |
| **Domain** | andresfelipemendez.com |
| **DNS** | Cloudflare |
| **K8s** | kubeadm v1.35 |
| **GitOps** | ArgoCD |
| **TLS** | Let's Encrypt (cert-manager) |
| **Credentials** | 1Password |

## Structure

```
cluster-config/
├── apps/
│   ├── argocd/              # ArgoCD apps and ingress
│   │   ├── ingress.yaml
│   │   ├── garden-app.yaml
│   │   ├── eink-dashboard-app.yaml
│   │   ├── castercat-app.yaml
│   │   ├── external-dns-app.yaml
│   │   ├── image-updater-app.yaml
│   │   └── kustomization.yaml
│   ├── garden/              # Digital garden (notes)
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   └── kustomization.yaml
│   ├── dashboard/           # Personal dashboard
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   ├── configmap.yaml
│   │   └── kustomization.yaml
│   ├── eink-dashboard/      # E-ink display dashboard
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   └── kustomization.yaml
│   └── castercat/           # Castercat website
│       ├── deployment.yaml
│       ├── service.yaml
│       ├── ingress.yaml
│       └── kustomization.yaml
├── infrastructure/
│   ├── namespaces/
│   ├── ingress-nginx/
│   └── cert-manager/
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
- URL: [argocd.andresfelipemendez.com](https://argocd.andresfelipemendez.com)


## Adding a New Service

1. Create `apps/<service-name>/` with deployment, service, ingress, kustomization
2. Create `apps/argocd/<service-name>-app.yaml`
3. Add to `apps/argocd/kustomization.yaml`
4. Push and ArgoCD auto-syncs

## Notes

- **ingress-nginx** uses `hostNetwork: true` to bind directly to ports 80/443 (single VPS setup)
- **ClusterIssuer** must be applied separately after cert-manager CRDs are ready
- **Secrets** are not committed - use `secrets.yaml.example` as template
- **Image updates** handled by ArgoCD Image Updater (digest strategy)
