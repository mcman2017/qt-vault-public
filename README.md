# Qt-Vault

Quantum-safe Kubernetes operator for managing encrypted secrets using post-quantum cryptography.

## Overview

Qt-Vault encrypts Kubernetes Secrets using NIST-standardized post-quantum algorithms:

| Algorithm | Purpose | Standard |
|-----------|---------|----------|
| **ML-KEM-768** | Key Encapsulation | NIST FIPS 203 |
| **X25519** | Classical Key Exchange | RFC 7748 |
| **ML-DSA-65** | Digital Signatures | NIST FIPS 204 |
| **Ed25519** | Classical Signatures | RFC 8032 |

Hybrid mode combines classical and post-quantum algorithms for defense in depth.

## Installation

### Add Helm Repository

```bash
helm repo add qt-vault https://mcman2017.github.io/qt-vault-public
helm repo update
```

### Install Qt-Vault

```bash
helm install qt-vault qt-vault/qt-vault -n qt-vault --create-namespace
```

### Verify Installation

```bash
kubectl get pods -n qt-vault
kubectl get crd quantumsecrets.qtvault.io
```

### Install qt-seal CLI

The `qt-seal` CLI is used to encrypt (seal) Kubernetes secrets client-side before committing them to Git.

#### macOS (Apple Silicon)

```bash
# Download the binary
curl -LO https://github.com/mcman2017/qt-vault-public/releases/download/v0.1.0/qt-seal-darwin-arm64

# Make it executable
chmod +x qt-seal-darwin-arm64

# Move to PATH
sudo mv qt-seal-darwin-arm64 /usr/local/bin/qt-seal

# Verify installation
qt-seal --version
```

#### From Source (Rust)

If you have Rust installed, you can build from source:

```bash
git clone https://github.com/mcman2017/qt-vault-public.git
cd qt-vault-public
# Note: Source code is in the private qt-vault repository
# Binary releases are available for download above
```

## Usage

### 1. Get the Public Key

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

## Configuration

See the [Helm chart documentation](./charts/qt-vault/README.md) for configuration options.

## Container Images

Images are available at:
- `ghcr.io/mcman2017/qt-vault:v0.1.0`

## License

Apache 2.0
