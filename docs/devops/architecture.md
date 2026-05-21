# WorldMonitor Architecture For The DevOps Project

## Current application state

WorldMonitor is a near real-time global intelligence dashboard. The current
repository contains these main components:

- A Vite and TypeScript single-page application.
- A Node.js API/local sidecar packaged in the main `worldmonitor` image.
- An AIS relay service running from `scripts/ais-relay.cjs`.
- Redis for cache data and seed metadata.
- A Redis REST proxy compatible with `UPSTASH_REDIS_REST_URL`.
- `consumer-prices-core`, an independent service with its own Dockerfile. It is
  treated as an extension because it requires PostgreSQL and migrations.

The original architecture is not a pure microservices architecture split by API
domain. This project therefore takes a pragmatic approach: operate the existing
runtime components as separate Kubernetes services instead of deeply refactoring
business logic.

## Mapping to microservices

| Service | Build source | Role | Port |
| --- | --- | --- | --- |
| `worldmonitor` | `Dockerfile` | Serves the SPA and proxies `/api/*` to the local API sidecar | `8080` |
| `ais-relay` | `Dockerfile.relay` | WebSocket relay, seed loops, RSS/OREF polling | `3004` |
| `redis-rest` | `docker/Dockerfile.redis-rest` | REST proxy for Redis | `80` |
| `redis` | `redis:7-alpine` image | Cache, seed metadata, rate-limit state | `6379` |

This model is enough to demonstrate the core DevOps concerns: container images,
service discovery, configuration and secret separation, health checks, CI/CD,
security scanning, GitOps deployment, and monitoring/logging.

## Target data flow

Users access `worldmonitor`. Nginx inside the container serves static assets and
proxies `/api/*` to the Node sidecar. The sidecar reads and writes cache data
through `redis-rest`. `ais-relay` provides relay data and selected seed loops.
Redis stores cache entries and freshness metadata used by `/api/health`.

## Out of scope for Person 1

Person 1 does not change logic under `src/`, `server/`, or `api/`, and does not
split domain APIs into new services. Kubernetes manifests, Argo CD sync,
monitoring stack installation, and cluster testing belong to Person 2.
