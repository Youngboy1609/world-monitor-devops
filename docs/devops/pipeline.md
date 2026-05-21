# CI/CD Pipeline

## Goal

The project pipeline must demonstrate the DevOps path from commit to container
image:

1. A developer pushes code or opens a pull request.
2. GitHub Actions runs quality gates and security scans.
3. The application image is built and scanned.
4. After merging to `main`, images are pushed to GHCR.
5. Person 2 uses the image tag in the GitOps manifests so Argo CD can sync the
   Kubernetes cluster.

## Workflow `devsecops.yml`

This workflow runs on `pull_request`, `push main`, and `workflow_dispatch`.

Main jobs:

- `quality`: installs dependencies with Node 22, then runs lint, typecheck, API
  typecheck, unit/data tests, and sidecar tests.
- `sast`: runs Semgrep default rules and fails on `ERROR` severity findings.
- `dependency-scan`: runs `npm audit` for the main packages in the repository
  and fails on fixable high/critical vulnerabilities.
- `iac-scan`: uses Trivy to scan Dockerfile/IaC configuration and fails on
  high/critical misconfigurations.
- `container-scan`: builds the `worldmonitor` image and scans it with Trivy,
  ignoring vulnerabilities that do not have a fix.
- `dast-baseline`: runs only when workflow dispatch provides `dast_target_url`;
  it uses OWASP ZAP baseline and fails on high-risk alerts.

The workflow does not require secrets for pull requests. This allows forked PRs
to run basic quality and security checks without exposing credentials.

## Workflow `container-publish.yml`

This workflow runs only on pushes to `main` or manual `workflow_dispatch`. It
builds and pushes 3 images to GHCR:

- `ghcr.io/<owner>/worldmonitor-app`
- `ghcr.io/<owner>/worldmonitor-ais-relay`
- `ghcr.io/<owner>/worldmonitor-redis-rest`

Each image receives these tags:

- `sha-<commit-sha>` for exact traceability.
- `main-latest` for the latest build from the `main` branch.

## Promotion flow for Person 2

Person 2 uses the `sha-<commit-sha>` tag in Kubernetes or Kustomize manifests.
After committing the manifest update, Argo CD detects the Git change and syncs
the cluster. This keeps the GitOps rule intact: the desired system state lives
in Git, and deployments do not rely on mutable image tags or manual commands.

## Local verification commands for Person 1

```bash
npm ci
npm run lint
npm run typecheck
npm run typecheck:api
npm run test:data
npm run test:sidecar
docker build -t worldmonitor-app:test -f Dockerfile .
docker build -t worldmonitor-ais-relay:test -f Dockerfile.relay .
docker build -t worldmonitor-redis-rest:test -f docker/Dockerfile.redis-rest docker
```
