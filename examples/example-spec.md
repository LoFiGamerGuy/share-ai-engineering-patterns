# Spec: Add /healthz endpoint to the API gateway

> **What this is.** An example of a single-task spec. It's the kind of file you'd write before dispatching a plan/execute/judge agent workflow. Notice that it's short, names files concretely, and pins down what "done" looks like.

## Goal

Add a `GET /healthz` endpoint to the API gateway service that returns the gateway's liveness status and the connection status of each downstream dependency.

## Why

Operations needs a single endpoint to point monitoring at. Currently `/status` returns a free-form text blob and `/admin/health` requires auth — neither works for the load balancer's health check.

## What "done" looks like

A new endpoint at `GET /healthz` that:

1. Returns HTTP 200 with JSON when the gateway and all dependencies are healthy
2. Returns HTTP 503 with JSON when any dependency is unhealthy
3. Has a per-call latency under 50ms p95 (no synchronous network calls; uses cached connection state)
4. Requires no authentication
5. Is documented in `docs/api/health-checks.md`
6. Has tests at `services/gateway/tests/health_test.go` covering: all-healthy, one-dependency-down, all-down, request latency under threshold

## Response shape

```json
{
  "status": "ok",        // "ok" | "degraded" | "down"
  "version": "1.4.7",
  "uptime_seconds": 38421,
  "dependencies": {
    "postgres": "ok",     // "ok" | "down" | "unknown"
    "redis": "ok",
    "auth_service": "ok"
  }
}
```

When degraded or down, the response includes a `failures` array describing which dependencies failed and their last successful check time.

## Constraints

- **Don't add new dependencies.** Use the existing `internal/healthcheck` package.
- **Don't break the existing `/status` endpoint.** It's used by the legacy admin dashboard.
- **Cache connection state.** The endpoint must not block on dependency checks. Use the background health-check goroutine that already runs every 5 seconds.
- **Follow gateway service conventions.** See `services/gateway/CLAUDE.md` for the project's patterns on routing, error handling, and tests.

## Files likely to change

- `services/gateway/handlers/health.go` (new)
- `services/gateway/router.go` (register the route)
- `services/gateway/tests/health_test.go` (new)
- `docs/api/health-checks.md` (new)

## Out of scope

- `/readyz` (Kubernetes-style readiness check) — separate spec
- Auth-required deeper diagnostics — already exists at `/admin/health`
- Metrics export — Prometheus already scrapes the existing `/metrics` endpoint
- Changes to the load balancer config — operations will handle that after this ships

## Acceptance

- All tests in `services/gateway/tests/health_test.go` pass
- `curl localhost:8080/healthz` returns the documented JSON
- p95 latency under load (`hey -n 1000 -c 50 http://localhost:8080/healthz`) is under 50ms
- Documentation is up to date and matches the implementation
- No regression in existing `/status` and `/metrics` endpoints
