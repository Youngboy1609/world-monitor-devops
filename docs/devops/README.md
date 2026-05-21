# DevOps Final Project

This directory contains the documentation owned by Person 1 for the final
project:

- [System architecture](architecture.md)
- [CI/CD pipeline](pipeline.md)
- [DevSecOps design](devsecops.md)
- [Final project report](report.md)

Person 1 owns CI/CD, DevSecOps workflows, and the report skeleton. Person 2 owns
Kubernetes deployment, Argo CD sync, monitoring, logging, and cluster-level
testing, then adds screenshots and logs to the report.

## Topic

**Design and implement a DevOps, DevSecOps, and GitOps process for the
WorldMonitor microservices application on Kubernetes.**

## Person 1 deliverables

- `.github/workflows/devsecops.yml` for quality gates, SAST, dependency
  scanning, IaC scanning, container scanning, and optional DAST baseline.
- `.github/workflows/container-publish.yml` for building and publishing images
  to GHCR.
- English documentation covering architecture, pipeline, DevSecOps, and the
  final report skeleton.

## Responsibility boundary

Person 1 does not deploy a real cluster, create the Argo CD Application, or edit
Kubernetes manifests. Operational evidence to be added by Person 2 includes Argo
CD status, `kubectl get pods,svc`, `/api/health`, Grafana dashboards, Loki logs,
and rollback or pod restart test results.
