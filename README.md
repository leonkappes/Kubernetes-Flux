# Kubernetes-Flux

> GitOps-driven Kubernetes home lab managed with [FluxCD](https://fluxcd.io/)

This repository contains the declarative configuration for my personal Kubernetes cluster — a "home-lab" environment where my workloads run through GitOps principles. Every change to the cluster goes through this repo preventing config drift.

---

## Architecture

The cluster is continuously reconciled by **FluxCD**, which watches this repository and applies changes automatically. Secrets are encrypted at rest using **Sealed Secrets**, so it is safe to store them in a public Git repository.

```
Kubernetes-Flux/
├── bootstrap/          # Flux bootstrap & cluster entry point
├── kappes-space/       # Personal Website
├── longhorn-system/    # Distributed block storage
├── podinfo/            # Podinfo – health check / canary app
├── reflector/          # Secret & ConfigMap replication across namespaces
├── reverse-proxy/      # Reverse proxy for on-prem Workloads
├── sealed-secrets/     # Bitnami Sealed Secrets controller
├── traefik/            # Traefik ingress controller
└── vaultwarden/        # Self-hosted Bitwarden-compatible password manager
```

---

## Stack

| Component | Purpose |
|---|---|
| [FluxCD](https://fluxcd.io/) | GitOps operator — keeps the cluster in sync with this repo |
| [Traefik](https://traefik.io/) | Ingress controller & reverse proxy |
| [Longhorn](https://longhorn.io/) | Cloud-native distributed block storage |
| [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) | Encrypted Kubernetes secrets safe for Git |
| [Reflector](https://github.com/emberstack/kubernetes-reflector) | Mirrors secrets/configmaps across namespaces |
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | Self-hosted, Bitwarden-compatible password manager |
| [Podinfo](https://github.com/stefanprodan/podinfo) | Lightweight demo / health-check microservice |

---

## Getting Started

### Prerequisites

Before bootstrapping Flux, set up the Sealed Secrets controller key so existing sealed secrets can be decrypted:

```bash
# Create the sealed-secrets namespace
kubectl create namespace sealed-secrets

# Import your existing keypair
export NAMESPACE="sealed-secrets"
export PRIVATEKEY="mytls.key"
export PUBLICKEY="mytls.crt"
export SECRETNAME="sealed-secrets-key"

kubectl -n "$NAMESPACE" create secret tls "$SECRETNAME" \
  --cert="$PUBLICKEY" \
  --key="$PRIVATEKEY"

kubectl -n "$NAMESPACE" label secret "$SECRETNAME" \
  sealedsecrets.bitnami.com/sealed-secrets-key=active
```

### Bootstrap Flux

```bash
flux bootstrap github \
  --owner=leonkappes \
  --repository=Kubernetes-Flux \
  --personal \
  --path bootstrap
```

Flux connects to this repository and will begin reconciling all resources automatically.

---

## Secret Management

All secrets in this repository are encrypted with **Sealed Secrets**. The `kubeseal` CLI is used to encrypt plain Kubernetes secrets into `SealedSecret` objects before committing:

```bash
kubeseal --format yaml < secret.yaml > sealed-secret.yaml
```

Only the cluster, holding the corresponding private key, can decrypt them.

---

## GitOps Workflow

```
git push → Flux detects change → applies manifests → cluster converges
```

1. Make changes to YAML manifests in this repository
2. Commit and push to `main`
3. Flux reconciles the cluster within its configured sync interval
4. Done — no manual intervention required
