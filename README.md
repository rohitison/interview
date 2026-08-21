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

Local Kubernetes

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