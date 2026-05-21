# DevOps Final Project Report

## 1. Project title

**Design and implement a DevOps, DevSecOps, and GitOps process for the
WorldMonitor microservices application on Kubernetes.**

## 2. Project objective

This project builds a complete DevOps process for WorldMonitor, including
containerizing the main runtime components, automating CI/CD, integrating
DevSecOps security checks, deploying to Kubernetes with GitOps, and adding
monitoring and logging for operations.

Person 1 focuses on CI/CD, DevSecOps workflows, and documentation/reporting.
Person 2 owns Kubernetes, Argo CD, deployment, testing, monitoring, and logging.

## 3. Current system state

WorldMonitor is a near real-time global intelligence dashboard. The application
currently includes:

- A Vite and TypeScript SPA.
- A Node.js API sidecar/local API server.
- An AIS relay service for WebSocket relay, seed loops, and selected upstream
  polling.
- Redis and a Redis REST proxy for cache data, freshness metadata, and rate
  limiting.

The repository already contains Dockerfiles and Docker Compose. Therefore the
project chooses a pragmatic deployment approach: run the existing components as
separate Kubernetes services instead of deeply refactoring every API domain into
new microservices.

## 4. Target deployment architecture

Target services:

| Service | Role | Image |
| --- | --- | --- |
| `worldmonitor` | SPA + API sidecar | `ghcr.io/<owner>/worldmonitor-app` |
| `ais-relay` | Relay and seed service | `ghcr.io/<owner>/worldmonitor-ais-relay` |
| `redis-rest` | REST proxy for Redis | `ghcr.io/<owner>/worldmonitor-redis-rest` |
| `redis` | Cache/stateful store | `redis:7-alpine` |

Main flow: users access `worldmonitor`; `/api/*` requests are proxied to the
local API sidecar; the sidecar uses `redis-rest` to read and write Redis;
`ais-relay` provides relay data and selected seed loops; `/api/health` checks
data freshness and service health.

## 5. CI/CD design

The pipeline is implemented with GitHub Actions.

Workflow `devsecops.yml` runs on pull requests, pushes to `main`, and manual
dispatch. It performs:

- Quality gates: lint, typecheck, API typecheck, unit/data tests, sidecar tests.
- SAST with Semgrep.
- Dependency scanning with `npm audit`.
- IaC/Dockerfile scanning with Trivy.
- Container image scanning with Trivy.
- DAST baseline with OWASP ZAP when a target URL is provided.

Workflow `container-publish.yml` runs on pushes to `main` or manual dispatch. It
builds and pushes 3 images to GHCR using `sha-<commit>` and `main-latest` tags.

## 6. DevSecOps design

DevSecOps is integrated into the pipeline to detect issues before images are
deployed.

SAST uses Semgrep default rules. Results are uploaded as an artifact. The
pipeline fails on `ERROR` severity findings.

Dependency scanning uses `npm audit` for the root app and supporting packages.
The pipeline fails on fixable high/critical vulnerabilities.

IaC scanning uses Trivy config scanning, focused on Dockerfiles and IaC or
Kubernetes manifests added by Person 2.

Container scanning uses Trivy image scanning with `--ignore-unfixed`, blocking
fixable high/critical vulnerabilities.

DAST uses OWASP ZAP baseline. Because DAST needs a deployed URL, this job runs
manually through `workflow_dispatch` with the `dast_target_url` input.

## 7. GitOps design

GitOps is implemented by Person 2 with Argo CD. The intended flow is:

1. Images are published to GHCR after merging to `main`.
2. Kubernetes or Kustomize manifests are updated with the `sha-<commit>` tag.
3. The manifest update is committed to Git.
4. Argo CD detects the change, syncs the cluster, and reconciles the desired
   state.

The key rule is: do not deploy manually with `kubectl set image`; the desired
system state must live in Git.

## 8. Monitoring and logging

Monitoring and logging are implemented by Person 2. The intended design is:

- Prometheus collects Kubernetes and service metrics.
- Grafana shows CPU, memory, restart count, pod status, and HTTP health
  dashboards.
- Loki/Promtail collects pod logs for debugging.

Evidence to add:

- Grafana dashboard screenshot.
- Loki logs for `worldmonitor` and `ais-relay`.
- `/api/health` check result.

## 9. Work division

Person 1:

- Designs and creates CI/CD workflows.
- Integrates DevSecOps checks.
- Standardizes the build/publish flow to GHCR.
- Writes architecture, pipeline, DevSecOps, and report documentation.

Person 2:

- Creates Kubernetes manifests.
- Installs and configures Argo CD.
- Deploys the system to K3s/Minikube.
- Tests services, health checks, restart, and rollback.
- Installs monitoring/logging and provides screenshots/logs.

## 10. Test results

### Person 1 results

Fill in after running workflows:

- `devsecops.yml`: evidence not added yet.
- `container-publish.yml`: evidence not added yet.
- Semgrep/Trivy/npm audit/ZAP artifacts: evidence not added yet.

### Evidence Person 2 must add

- Argo CD screenshot showing `Healthy` and `Synced`.
- Output of `kubectl get pods,svc -n worldmonitor`.
- Curl output for `/api/health`.
- Grafana dashboard screenshot.
- Loki logs.
- Rollback or pod restart test result.

## 11. Limitations and future improvements

Limitations:

- The project does not refactor every API domain into an independent
  microservice.
- DAST depends on a real deployed URL.
- Monitoring/logging requires runtime evidence from Person 2.
- Image scanning currently focuses on the `worldmonitor` image; it can be
  extended to relay and redis-rest.

Future improvements:

- Add Helm charts or Kustomize overlays for staging/production.
- Extend container scanning to all images.
- Add policy-as-code with Conftest or Kyverno.
- Add progressive delivery with Argo Rollouts.
