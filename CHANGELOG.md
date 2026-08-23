# Changelog

All notable changes to this project are documented in this file.

The project follows [Semantic Versioning](https://semver.org/).

## [1.0.1] - 2026-08-23

### Added

- Terraform configuration for the local Kind cluster.
- Terraform variables and outputs.
- GitHub Actions CI and release workflows.

### Changed

- Improved Kubernetes deployment configuration.
- Improved project documentation.
- Added `.gitignore` for Terraform state and generated files.

## [1.0.0] - 2026-08-19

### Added

- Initial Dockerized application.
- Local three-node Kind Kubernetes cluster configuration.
- Kubernetes Deployment with two replicas.
- Kubernetes ClusterIP Service.
- Readiness and liveness probes.
- CPU and memory requests and limits.
- Rolling update strategy.
- Shell logging script with timestamped output.
