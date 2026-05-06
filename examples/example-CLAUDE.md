# CLAUDE.md — example project context file

> **What this is.** An example of a project-level context file that an agent loads at session start. It tells the agent the things it can't infer from the code alone: conventions, decisions, where to look, what to avoid. Most teams' first attempts at these are too long and too generic. The shape that works is short, specific, and focused on what the agent gets wrong without it.

---

# Acme Gateway Service

A Go HTTP service that fronts our customer-facing APIs. Handles auth, rate limiting, and routing to internal services.

## What you need to know

- **Language:** Go 1.22. Module is `github.com/acme/gateway`.
- **Routing:** chi router. Routes are registered in `router.go`.
- **Auth:** JWT, validated against the auth service. Middleware in `internal/auth/`.
- **Tests:** standard `go test`. Run with `make test` (uses race detector and coverage). Aim for 80%+ on changed packages.
- **Linting:** `golangci-lint run` — must pass before merge.
- **Build:** `make build` produces `bin/gateway`.

## Conventions

- **Errors are values.** Use `fmt.Errorf("context: %w", err)` to wrap. Sentinel errors live in each package's `errors.go`. Don't introduce `errors.New` for things callers might want to handle differently.
- **No global state.** Services are passed via constructor (`NewHandler(deps Deps)`). Logging and metrics are dependencies, not package-level singletons.
- **Tests live next to code** (`foo.go` → `foo_test.go`). Integration tests live in `tests/integration/` and are tagged `integration`.
- **Generated code** (mocks, protobuf) lives under `internal/gen/` and is regenerated with `make gen`. Don't edit by hand.
- **Migrations** are in `migrations/`. We use `goose`. Never edit a committed migration; write a new one.

## Architecture

- `cmd/gateway/` — main entry point. Wires dependencies, starts HTTP server.
- `internal/auth/` — JWT validation, token refresh, session management.
- `internal/ratelimit/` — token-bucket rate limiter, per-user and per-IP.
- `internal/proxy/` — request forwarding to downstream services.
- `internal/healthcheck/` — background dependency probing. Cached state.
- `services/` — typed clients for each downstream service.
- `migrations/` — database schema migrations.

Downstream services are addressed by name (`auth-service`, `users-service`, etc.) and resolved via the service discovery layer in `internal/discovery/`.

## What to avoid

- **Don't add `panic()` in handlers.** Always return errors. The error middleware translates them to HTTP responses.
- **Don't bypass the rate limiter** for internal callers. Use the `internal-token` claim instead.
- **Don't add new third-party dependencies without asking.** We're conservative about the dependency graph.
- **Don't write integration tests that hit real downstream services.** Use the mock servers in `tests/integration/mocks/`.
- **Don't change the `/status` or `/metrics` endpoints.** They're load-bearing for ops tooling.

## Where things are documented

- API contract: `docs/api/`
- Architecture decisions: `docs/adr/`
- Operational runbooks: `docs/runbooks/`
- Onboarding: `docs/onboarding.md`

If you're about to add documentation, check whether it belongs in one of these existing places before creating a new one.

## Common tasks

- **Add a new endpoint:** create handler in `internal/handlers/`, register in `router.go`, add tests, update `docs/api/`.
- **Add a new downstream service:** add typed client in `services/<name>/`, add to discovery config, add to health check probes.
- **Bump a dependency:** `go get -u path/to/dep`, run `go mod tidy`, run `make test`. Note in PR description if it's a major version.

## Pre-merge checklist

- [ ] `make test` passes (including race detector)
- [ ] `golangci-lint run` clean
- [ ] Coverage on changed packages still ≥ 80%
- [ ] Docs updated if API or behavior changed
- [ ] No new third-party deps without prior discussion
- [ ] PR description explains the *why*, not just the *what*

## Don't auto-commit

This repo's policy is: agents may stage changes and prepare commits, but a human must run the actual `git commit` and `git push`. Do not bypass this.
