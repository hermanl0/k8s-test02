# k8s-test02 — hl0.dev lab

> *You are the Kubernetes administrator. The demons are the microservices. Good luck.*

Personal Kubernetes lab on Azure — live at **[doom.hl0.dev](https://doom.hl0.dev)**.

![Doom Dashboard](https://hermanl0.github.io/img/doom-dashboard.png)

---

## The Arsenal

| Weapon | What it does |
|--------|--------------|
| **AKS** (2 × Standard_D2s_v6) | Runs the cluster |
| **NGINX Gateway Fabric** | Gateway API ingress — HTTP→HTTPS redirect, TLS termination |
| **cert-manager** | Automatic Let's Encrypt TLS via Cloudflare DNS-01 |
| **kube-prometheus-stack** | Prometheus + Grafana + Alertmanager |
| **Loki + Promtail** | Log aggregation |
| **Uptime Kuma** | Public status page |
| **Doom dashboard** | Flask app — live cluster metrics + ServiceNow incidents |

---

## Live Endpoints

| URL | What's there |
|-----|-------------|
| [doom.hl0.dev](https://doom.hl0.dev) | The dashboard |
| [grafana.hl0.dev](https://grafana.hl0.dev) | Grafana |
| [status.hl0.dev](https://status.hl0.dev) | Uptime Kuma |
| [test02.hl0.dev](https://test02.hl0.dev) | nginx |

---

## Observability

![Grafana](https://hermanl0.github.io/img/k8s-test02-grafana.png)

Prometheus scrapes everything — nodes, pods, and NGF request metrics via `ServiceMonitor`. Loki + Promtail handles logs. All wired into Grafana.

---

## Status

![Uptime Kuma](https://hermanl0.github.io/img/k8s-test02-uptimekuma.png)

Public status page at [status.hl0.dev](https://status.hl0.dev) — monitors all four endpoints.

---

## Security

- **NetworkPolicies**: default-deny in `lab` namespace; per-pod allow rules
- **Rate limiting**: NGF `SnippetsFilter` — 20 req/s per IP, burst 60, 429 on excess
- **Security headers**: HSTS, X-Content-Type-Options, X-Frame-Options, Referrer-Policy on all routes
- **TLS**: cert-manager + Let's Encrypt, HSTS preload

---

## CI/CD

Push to `main` → GitHub Actions → validate YAML + secret scan → helm upgrades → `kubectl apply` → Cloudflare DNS upsert → rollout verify. About **3–4 minutes** end to end.

**Secrets needed:** `AZURE_CREDENTIALS`, `GH_PAT`, `CLOUDFLARE_API_TOKEN`, `GRAFANA_ADMIN_PASSWORD`, `SNOW_PASS`, `UPTIME_KUMA_API_KEY`

---

## Key Commands

```bash
# Credentials
az aks get-credentials --resource-group lab-rg --name lab-aks

# Cluster overview
kubectl get pods,svc,pvc -n lab
kubectl get pods -n monitoring
kubectl get gateway,httproute -A

# Cert status
kubectl get certificate -A

# Doom dashboard logs
kubectl logs -n lab deployment/doom-dashboard -f

# Restart doom-dashboard
kubectl rollout restart deployment/doom-dashboard -n lab
```
