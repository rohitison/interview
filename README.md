# DevOps Interview Solution

This repository contains my solution for the Kubernetes deployment and DevOps review exercise.

The solution uses a local Kubernetes cluster with Kind and demonstrates:

- Docker containerization
- Kubernetes deployment
- CI validation
- Automated releases
- Semantic Versioning
- Changelog management
- Review and improvement of the provided shell script and Kubernetes manifest

No cloud infrastructure is required.

---

## Architecture

### CI / Release

```text
                         GitHub
                           |
                       git push
                           |
                           v
                +----------------------+
                |    GitHub Actions    |
                |         CI           |
                +----------------------+
                   |                |
                   v                v
             Docker build     K8s validation
                   |
             git tag v1.0.0
                   |
                   v
                +----------------------+
                |    GitHub Actions    |
                |       Release        |
                +----------------------+
                           |
                           v
                         GHCR
                           |
                           v
              Versioned container image
```

### Local Kubernetes

```text
                  Docker Image
                       |
                       v
              +-------------------+
              |   Kind Cluster    |
              |                   |
              |  Control Plane    |
              |                   |
              |  +-------------+  |
              |  |   Worker 1  |  |
              |  |    Pod 1    |  |
              |  +-------------+  |
              |                   |
              |  +-------------+  |
              |  |   Worker 2  |  |
              |  |    Pod 2    |  |
              |  +-------------+  |
              |         |         |
              |         v         |
              |  +-------------+  |
              |  |  ClusterIP  |  |
              |  |   Service   |  |
              |  +-------------+  |
              +-------------------+
```

The CI/release flow and the local Kubernetes environment are intentionally separate.

GitHub Actions builds and publishes the release image to GHCR, while Kind is used locally to run and test the application.

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
├── shell/
│   └── script.sh
├── CHANGELOG.md
└── README.md
```

---

## Tools Used

- Docker
- Kubernetes
- Kind
- kubectl
- GitHub Actions
- GitHub Container Registry (GHCR)
- kubeconform
- Semantic Versioning

---

## Application

The application is intentionally simple. The focus of the exercise is the DevOps workflow rather than application complexity.

It is a small HTML page served by Nginx.

The Dockerfile was created specifically for this exercise.

Build the image locally:

```bash
docker build -t devops-interview-app:1.0.0 ./app
```

The image was tested locally before deploying it to Kubernetes.

---

## Local Kubernetes Setup

Kind is used to create a local multi-node Kubernetes cluster.

The cluster contains:

- 1 control-plane node
- 2 worker nodes

Create the cluster:

```bash
kind create cluster --config kind/cluster.yaml
```

Verify the nodes:

```bash
kubectl get nodes
```

---

## Kubernetes Deployment

Because the Docker image is built locally, it needs to be loaded into the Kind nodes:

```bash
kind load docker-image devops-interview-app:1.0.0 --name devops-interview
```

Deploy the application:

```bash
kubectl apply -f k8s/nginx.yaml
```

Check the deployment:

```bash
kubectl get pods -o wide
kubectl get service
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

## Kubernetes Design

The Deployment runs two replicas.

I used two replicas mainly to demonstrate basic redundancy, rolling updates and Kubernetes self-healing.

The Deployment also includes:

- Rolling update strategy
- Readiness probe
- Liveness probe
- CPU and memory requests/limits
- Versioned container image

The application is exposed using a `ClusterIP` Service because external access is not required for this exercise.

I also tested self-healing by deleting a running Pod and verifying that Kubernetes created a replacement.

### Rolling update

The Deployment uses:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This allows a new Pod to be created before an old Pod is removed while keeping the configured replicas available during the rollout.

---

## CI

The CI workflow is located at:

```text
.github/workflows/ci.yml
```

It runs on pushes and pull requests.

The workflow currently:

1. Checks out the repository
2. Builds the Docker image
3. Validates the Kubernetes manifest using kubeconform

The CI job does not depend on the local Kind cluster. This keeps the CI environment independent from my local machine.

---

## Automated Release

The release workflow is located at:

```text
.github/workflows/release.yml
```

A release is triggered by a Semantic Versioning Git tag.

For example:

```bash
git tag v1.0.0
git push origin v1.0.0
```

The release workflow then:

1. Extracts the version from the tag
2. Builds the Docker image
3. Pushes the versioned image to GHCR
4. Creates a GitHub Release

The resulting image is:

```text
ghcr.io/rohitison/devops-interview-app:1.0.0
```

I use versioned image tags instead of `latest` so that each release points to a specific and traceable image.

---

## Semantic Versioning & Changelog

The project follows:

```text
MAJOR.MINOR.PATCH
```

For example:

```text
1.0.0
```

The general approach is:

- `MAJOR` — breaking changes
- `MINOR` — backward-compatible features
- `PATCH` — backward-compatible fixes

Release changes are documented in:

```text
CHANGELOG.md
```

---

## Review: Shell Script

The original `shell/script.sh` had several issues:

- Missing shebang
- Inconsistent variable names
- Incorrect variable expansion caused by quoting
- `LOG_FILE` / `LOGFILE` mismatch
- Incorrect logging destination
- No strict shell error handling

I changed the script to use:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

and timestamped logging with `printf`.

The main goal was to make the script predictable, fail fast, and easier to maintain.

---

## Review: Kubernetes Manifest

The original Kubernetes manifest had several issues:

- Deployment selector and Pod labels did not match
- Port configuration was inconsistent with the Nginx application
- Service configuration was incomplete
- Service selectors were incorrect/incomplete
- Image was not versioned
- No readiness probe
- No liveness probe
- No resource requests or limits
- No explicit rolling update strategy

I corrected these issues while keeping the manifest relatively simple.

One Kubernetes detail worth mentioning is that `containerPort` is metadata; setting it to `8000` does not make Nginx listen on port `8000`. The actual issue was the mismatch between the declared port and the port where the application listens.

---

## Design Decisions

### Why Kind?

The assignment requires a local Kubernetes cluster. Kind provides a simple multi-node Kubernetes environment using Docker.

### Why two replicas?

Two replicas provide basic redundancy and allow rolling updates and self-healing to be demonstrated.

### Why ClusterIP?

The application does not need to be exposed outside the cluster for this exercise, so `ClusterIP` keeps the setup simple.

### Why versioned images?

Versioned image tags make releases easier to trace and reproduce compared with using a mutable `latest` tag.

### Why separate CI and Release?

CI validates changes on pushes and pull requests.

A release only happens when a version tag is created. This keeps normal development changes separate from published release artifacts.

### Why not Terraform or GitOps?

They were not required for this exercise.

The cluster configuration and Kubernetes desired state are already stored declaratively in Git. I preferred to keep the solution focused rather than introduce additional tooling just to make the solution more complex.

For a larger production environment, Terraform and a GitOps tool such as Argo CD or Flux could be appropriate depending on the infrastructure and deployment model.

---

## Final Notes

The solution intentionally stays relatively small.

The goal was to solve the requested tasks cleanly and make the important decisions easy to explain during the interview, rather than adding infrastructure that is not required for the exercise.