# DevSecOps Design

## Goal

DevSecOps is integrated directly into the pipeline so issues are detected before
images are used for GitOps deployment. The checks focus on source code,
dependencies, Dockerfile/IaC configuration, container images, and an optional
DAST baseline.

## SAST

SAST uses Semgrep default rules in the `sast` job. Results are written to
`semgrep.json` and uploaded as an artifact. The pipeline fails on `ERROR`
severity findings; lower severity findings are kept for review and reporting.

Reasons for choosing Semgrep:

- It runs in CI without secrets.
- It supports TypeScript and JavaScript well.
- It produces reports that are easy to attach to the project evidence.

## Dependency scanning

Dependency scanning uses `npm audit` for the main packages:

- Root app.
- `scripts`.
- `blog-site`.
- `pro-test`.
- `consumer-prices-core`.

The pipeline fails only when a vulnerability is high/critical and
`fixAvailable` is not `false`. This avoids hard-failing on low-risk findings or
issues without a fix, while still blocking serious vulnerabilities that can be
addressed immediately.

## IaC and Dockerfile scanning

The `iac-scan` job uses Trivy config scanning with the `misconfig` scanner. The
scan covers the whole repository, prioritizing Dockerfiles, Compose files, and
IaC/Kubernetes files if Person 2 adds them later. The pipeline fails on
high/critical misconfigurations.

## Container image scanning

The `container-scan` job builds the `worldmonitor` image from the root
`Dockerfile`, then scans it with Trivy image scanning. The scan uses
`--ignore-unfixed` so it fails only on high/critical vulnerabilities that have a
fix or mitigation path.

The relay and redis-rest images are built and pushed by the publish workflow. If
time allows, container scanning can be extended into a matrix for all 3 images.

## DAST baseline

DAST uses OWASP ZAP baseline and runs only through `workflow_dispatch` when the
`dast_target_url` input is provided. This matches Person 1's boundary: the
pipeline supports DAST, but the real target URL is provided by Person 2 after
deployment.

Example:

```text
GitHub Actions -> DevSecOps -> Run workflow -> dast_target_url=http://<app-url>
```

The job uploads 3 artifacts:

- `zap-report.json`
- `zap-report.html`
- `zap-report.xml`

The pipeline fails when ZAP reports a high-risk alert.

## Secret management

Pull request workflows do not use secrets. Real credentials such as
`REDIS_PASSWORD`, `REDIS_TOKEN`, `RELAY_SHARED_SECRET`, and external API keys
must live in GitHub Secrets or Kubernetes Secrets and must not be committed to
the repository. Documentation should reference only `.env.example` and the
secret templates created by Person 2.
