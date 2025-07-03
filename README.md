# oci-edge example 

# Clone this repo 

# Install v3.1.0 of Argo CD
https://github.com/argoproj/argo-cd/releases/tag/v3.1.0-rc1

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.1.0-rc1/manifests/install.yaml

# Set up Argo CD
Create this Argo CD Application, which points to an OCI Registry. In my examples case using Docker Hub.

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

# Render manifests to deploy. Generate OCI Artifact.
I'm using Kustomize to do this, also to best show this example first navigate your terminal to the flashdrive folder. This is where we want to generate our OCI Artifact

Render manifests with Kustomize:
kustomize build guestbook > guestbook.yaml

Zip the contents:
tar -czf guestbook.tar.gz .

Using an OCI client, push OCI Artifact into registry:
I'm using ORAS here.

oras push registry-1.docker.io/dhpup/oci-edge:1.0.1 guestbook.tar.gz:application/vnd.oci.image.layer.v1.tar+gzip

Iterate new changes using SemVer and Argo CD will detect and deploy it. 