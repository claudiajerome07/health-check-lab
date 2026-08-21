# Cloud Run Health Check Mapping

## Local Health Checks

This service exposes two endpoints with different responsibilities:

- `GET /health` is the liveness check. It verifies that the Orders API process is alive and does not depend on PostgreSQL.
- `GET /ready` is the readiness check. It verifies that PostgreSQL is reachable using `SELECT 1`. It returns `200 READY` when the dependency is available and `503 NOT READY` when it is unavailable.

The Docker Compose healthcheck uses `/ready` so the container is considered healthy only when the application can reach its database dependency.

## Cloud Run Mapping

The local `/health` endpoint maps conceptually to a Cloud Run liveness check. A liveness check answers whether the application instance is still functioning and should continue running.

The local `/ready` endpoint maps conceptually to readiness behavior. It answers whether the application is currently able to serve requests that require its database dependency.

Cloud Run also supports startup probes/checks to determine whether a newly started container has finished initializing before normal health evaluation begins. The Docker Compose `start_period` serves a similar purpose locally by giving the application time to start before healthcheck failures count against it.

This project does not deploy to Cloud Run. The mapping above is documentation only.

## Kubernetes Mapping

If this service were deployed to Kubernetes:

- `/health` would be appropriate for a `livenessProbe`.
- `/ready` would be appropriate for a `readinessProbe`.
- A startup probe could protect the application during slow initialization.

The key principle is to keep liveness and readiness separate:

- Liveness: "Is the process alive?"
- Readiness: "Can the process currently serve requests successfully?"