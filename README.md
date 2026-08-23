# DevOps Interview Solution

This repository contains my solution for the Kubernetes deployment and DevOps review exercise.

The solution is intentionally kept local and uses Kind, Terraform, Docker, Kubernetes and GitHub Actions. Releases are versioned using Semantic Versioning and published to GHCR.

---

## Architecture

```text
                         GitHub
                           |
                 +---------+---------+
                 |                   |
              Push / PR          Git tag
                 |                v1.0.0
                 v                   |
          GitHub Actions             v
               CI             GitHub Actions
                 |                Release
          +------+------+            |
          |             |            v
     Docker build   K8s validation  GHCR
                                    |
                                    v
                              Versioned image


                 Terraform
                     |
                     v
                Kind cluster
                     |
                     v
                Kubernetes
                     |
                     v
              Application Pods
                     |
                     v
                  Service
```

---

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── release.yml
├── app/
│   ├── Dockerfile
│   └── index.html
├── kind/
│   └── cluster.yaml
├── k8s/
│   └── nginx.yaml
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── .terraform.lock.hcl
├── shell/
│   └── script.sh
├── .gitignore
├── CHANGELOG.md
└── README.md
```

---

## Infrastructure

Terraform is used to create and manage the local Kind cluster.

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

Check the cluster:

```bash
kubectl get nodes
```

To remove the infrastructure:

```bash
terraform destroy
```

Terraform state and generated files are excluded through `.gitignore`, while the provider lock file is committed for reproducible provider selection.

---

## Application

The application is packaged as a Docker image.

Build locally:

```bash
docker build -t devops-interview-app:1.0.0 ./app
```

The Kubernetes Deployment uses two replicas and includes:

- Rolling update strategy
- Readiness and liveness probes
- CPU and memory requests/limits
- Versioned container image
- ClusterIP Service

Deploy:

```bash
kubectl apply -f k8s/nginx.yaml
kubectl rollout status deployment/devops-interview-app
```

For local access:

```bash
kubectl port-forward service/devops-interview-app 8080:80
```

Then:

```bash
curl http://localhost:8080
```

---

## CI

The CI workflow is:

```text
.github/workflows/ci.yml
```

It runs on pushes and pull requests.

It:

1. Builds the Docker image
2. Validates the Kubernetes manifest with kubeconform

The image is built during CI but is not pushed to a registry. Publishing is handled by the release workflow.

---

## Release

The release workflow is:

```text
.github/workflows/release.yml
```

A release is triggered by a Semantic Versioning Git tag:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow:

1. Extracts the version from the Git tag
2. Builds the Docker image
3. Pushes the versioned image to GHCR
4. Creates a GitHub Release

Example:

```text
ghcr.io/rohitison/devops-interview-app:1.0.0
```

Versioned images are used instead of `latest` so releases remain traceable and reproducible.

---

## Kubernetes

The Deployment uses:

- 2 replicas
- RollingUpdate strategy
- Readiness probe
- Liveness probe
- CPU and memory requests
- CPU and memory limits
- Versioned container image

The rolling update configuration is:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

The application is exposed internally through a `ClusterIP` Service.

I also tested Kubernetes self-healing by deleting a running Pod and verifying that Kubernetes created a replacement.

---

## Review

### Shell Script

The provided shell script was reviewed and improved with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

Variable inconsistencies, logging issues and error handling were also corrected.

### Kubernetes Manifest

The provided manifest was reviewed and corrected for:

- Deployment selectors and labels
- Service selectors and ports
- Versioned images
- Readiness and liveness probes
- Resource requests and limits
- Rolling update configuration

---

## Design Decisions

### Kind

Kind provides a simple multi-node Kubernetes cluster locally using Docker and satisfies the local-cluster requirement.

### Terraform

Terraform provides a reproducible way to create and destroy the local infrastructure instead of relying only on manual cluster commands.

### Versioned Images

Release tags are mapped to versioned container images rather than using `latest`.

### CI vs Release

CI validates changes without publishing artifacts.

A SemVer tag triggers the release workflow, which builds and publishes the versioned image to GHCR and creates a GitHub Release.

### GitOps

GitOps/Argo CD was intentionally not included. It was not required for the exercise, so the implementation remains focused on Kubernetes, IaC, CI/CD and automated releases.

Without GitOps, publishing a new image does not automatically update an existing Kubernetes Deployment. The Kubernetes manifest therefore references an explicit image version.