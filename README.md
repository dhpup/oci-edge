# OCI Edge Example Guide

## Getting Started

### Clone the Repository

First, clone this repository to your local machine:

```bash
git clone <repository-url>
cd oci-edge-example
```

## Argo CD Installation

### Install Argo CD v3.1.0

Install Argo CD version 3.1.0 from the official release:

**Release Link:** [Argo CD v3.1.0-rc1](https://github.com/argoproj/argo-cd/releases/tag/v3.1.0-rc1)

```bash
# Create the argocd namespace
kubectl create namespace argocd

# Install Argo CD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.1.0-rc1/manifests/install.yaml
```

## Argo CD Application Setup

### Create Argo CD Application

Create an Argo CD Application that points to an OCI Registry. This example uses Docker Hub:

```yaml
project: default
source:
  repoURL: oci://registry-1.docker.io/dhpup/oci-edge
  path: .
  targetRevision: 1.*
destination:
  server: https://kubernetes.default.svc
  namespace: guestbook
syncPolicy:
  syncOptions:
    - CreateNamespace=true
```

## Building and Deploying OCI Artifacts

### Render Manifests and Generate OCI Artifact

This example uses Kustomize to render manifests and create OCI artifacts.

**Important:** Navigate your terminal to the `flashdrive` folder before proceeding, as this is where we want to generate our OCI Artifact.

#### Step 1: Render Manifests with Kustomize

```bash
kustomize build guestbook > guestbook.yaml
```

#### Step 2: Create Archive

```bash
tar -czf guestbook.tar.gz .
```

#### Step 3: Push OCI Artifact to Registry

Using ORAS (OCI Registry As Storage) client:

```bash
oras push registry-1.docker.io/dhpup/oci-edge:1.0.1 guestbook.tar.gz:application/vnd.oci.image.layer.v1.tar+gzip
```

### Continuous Deployment

To deploy new changes:

1. **Iterate using SemVer**: Update your version number following semantic versioning
2. **Automatic Detection**: Argo CD will automatically detect and deploy the new version

This workflow enables GitOps-style deployments using OCI registries as the source of truth for your Kubernetes manifests.