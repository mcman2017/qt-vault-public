# Qt-Vault Helm Chart

Quantum-safe Kubernetes operator for managing encrypted secrets using post-quantum cryptography.

## Introduction

This chart deploys the Qt-Vault controller on a Kubernetes cluster. Qt-Vault provides quantum-safe encryption for Kubernetes Secrets using:

- **ML-KEM-768** (NIST FIPS 203) + X25519 hybrid key encapsulation
- **ML-DSA-65** (NIST FIPS 204) + Ed25519 hybrid signatures

These NIST-standardized post-quantum algorithms protect your secrets against both classical and future quantum computer attacks.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installing the Chart

Add the Qt-Vault Helm repository:

```bash
helm repo add qt-vault https://mcman2017.github.io/qt-vault
helm repo update
```

Install the chart with the release name `qt-vault`:

```bash
helm install qt-vault qt-vault/qt-vault -n qt-vault --create-namespace
```

## Uninstalling the Chart

```bash
helm uninstall qt-vault -n qt-vault
```

> **Note:** By default, the CRD is kept after uninstallation. Set `crd.keep=false` to remove it.

## Configuration

See [values.yaml](./values.yaml) for the full list of configurable parameters.

### Common Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.registry` | Image registry | `ghcr.io` |
| `image.repository` | Image repository | `mcman2017/qt-vault` |
| `image.tag` | Image tag | `v0.1.0` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `replicaCount` | Number of replicas | `1` |
| `secretName` | Name of the keys secret | `qt-vault-keys` |

### Security Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `podSecurityContext.enabled` | Enable pod security context | `true` |
| `podSecurityContext.runAsNonRoot` | Run as non-root | `true` |
| `containerSecurityContext.enabled` | Enable container security context | `true` |
| `containerSecurityContext.readOnlyRootFilesystem` | Read-only root filesystem | `true` |

### RBAC Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create service account | `true` |
| `rbac.create` | Create RBAC resources | `true` |
| `rbac.clusterRole` | Create ClusterRole | `true` |

### Metrics Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `metrics.serviceMonitor.enabled` | Enable Prometheus ServiceMonitor | `false` |
| `metrics.serviceMonitor.interval` | Scrape interval | `""` |

## Usage

### 1. Get the Public Key

After installation, retrieve the public key:

```bash
kubectl get secret qt-vault-keys -n qt-vault -o jsonpath='{.data.public-key}' | base64 -d > qt-vault.pub
```

### 2. Seal a Secret

Use the `qt-seal` CLI to encrypt a Kubernetes Secret:

```bash
kubectl create secret generic my-secret \
  --from-literal=password=supersecret \
  --dry-run=client -o json | \
  qt-seal seal --cert qt-vault.pub --format yaml > my-quantumsecret.yaml
```

### 3. Apply the QuantumSecret

```bash
kubectl apply -f my-quantumsecret.yaml
```

The Qt-Vault controller will decrypt it and create the corresponding Kubernetes Secret.

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   qt-seal CLI   │────▶│  QuantumSecret  │────▶│    K8s Secret   │
│ (Client-side)   │     │      (CRD)      │     │   (Decrypted)   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │  Qt-Vault       │
                        │  Controller     │
                        │  (Server-side)  │
                        └─────────────────┘
```

## Cryptography

| Algorithm | Purpose | Standard |
|-----------|---------|----------|
| ML-KEM-768 | Key Encapsulation | NIST FIPS 203 |
| X25519 | Classical Key Exchange | RFC 7748 |
| ML-DSA-65 | Digital Signatures | NIST FIPS 204 |
| Ed25519 | Classical Signatures | RFC 8032 |

Hybrid mode combines classical and post-quantum algorithms for defense in depth.

## License

Apache 2.0
