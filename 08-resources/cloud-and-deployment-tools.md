# Cloud and Deployment Tools

Where your agents, MCP servers, sandbox environments, and overnight runs actually live. The patterns and the tools that implement them.

This page is opinionated and provider-aware. Cloud picks ARE provider-specific in ways that shell tools aren't — but the *patterns* are portable across providers, and naming the patterns separately from the tools lets you re-implement anywhere.

## How to read this page

The structure goes pattern → implementing tools → discipline. Each pattern names what it solves; the tool table shows current options across major providers; the discipline notes are the load-bearing operational details.

Provider-agnostic where possible; provider-specific where the picks matter.

---

## Always-on zero-cost gateway

**Pattern:** A tiny, permanently-on machine that handles SSH proxying, reverse proxying, and DNS — at zero ongoing cost. It exists so your *expensive* compute can sleep when not in use.

The architecture: cheap free-tier VM as the always-on entry point; larger compute VM behind it that sleeps. The free-tier VM costs nothing; the compute VM only runs when you're using it.

| Pattern | Tool(s) implementing it |
|---------|------------------------|
| Always-free micro VM | GCP e2-micro (perpetually free in select regions); Oracle Cloud Always Free (multiple instances); Fly.io free tier |
| 12-month free trial micro | AWS t3.micro; Azure B1s — useful for exploration, not long-term zero-cost |

The GCP always-free tier is the standout for this pattern. AWS and Azure offer free trials but not perpetual free; their micro tiers eventually bill.

**Discipline:** the gateway holds the public IP (or DNS) and the TLS certificate. The compute VM is private — no public IP. Connecting to compute happens *through* the gateway via ProxyJump or Tailscale (next pattern).

---

## VPN-only private networking

**Pattern:** SSH and admin access happens exclusively over a private overlay network, not the public internet. Port 22 is closed at the firewall to the world.

| Pattern | Tool(s) |
|---------|---------|
| Mesh overlay VPN (zero-config) | **Tailscale** (WireGuard underneath; free up to 100 devices) |
| Self-hosted overlay | WireGuard |
| Cloud-native VPN | AWS Client VPN; GCP Cloud VPN |
| Outbound-only tunnel (no inbound at all) | Cloudflare Tunnel (eliminates need for public IP entirely) |
| IAM-based shell access (no VPN) | AWS Systems Manager Session Manager; GCP IAP (Identity-Aware Proxy) |

**Tailscale is the dominant pick in this category.** It assigns each node a stable IP in the `100.x.x.x` CGNAT range regardless of cloud provider or NAT. Cross-provider access just works — laptop, GCP VM, AWS instance all on the same Tailnet. SSH firewall rule restricts to the CGNAT range; brute-force exposure goes to zero.

**Discipline:**
- Always have a backup access path. If Tailscale fails, you need a way in. AWS SSM Session Manager and GCP IAP are credentialed differently from Tailscale, so a Tailscale compromise doesn't lock you out.
- Treat each Tailscale device IP as a device identity. Audit logs tied to source IPs effectively tie events to devices.

---

## Reverse proxy and TLS

**Pattern:** TLS terminates at the gateway. Internal services bind to localhost on the compute VM and are never directly exposed to the internet.

| Pattern | Tool(s) |
|---------|---------|
| Reverse proxy | **nginx** (workhorse); Caddy (auto-TLS built in); Traefik (container-native); HAProxy (high throughput) |
| Auto-issued TLS certs | Certbot + Let's Encrypt; Cloudflare managed TLS; Caddy's built-in ACME |
| Dynamic DNS (no static IP) | DuckDNS (free); Cloudflare DDNS API; no-ip |
| Eliminate static IP entirely | Cloudflare Tunnel (outbound-only) |

**Discipline:**
- Internal services bind to `127.0.0.1`, not `0.0.0.0`. Even if your firewall is misconfigured, services that bind to localhost can't be reached externally.
- Two-layer auth on web portals: nginx HTTP basic auth at the gateway *plus* application-level auth at the service. Defense in depth costs almost nothing and catches misconfigured proxy rules.
- Renew certs automatically. A cert that expires takes the entire surface down silently.

---

## Container orchestration

**Pattern:** Reproducible isolated environments for agents, MCP servers, sandbox runs.

| Pattern | Tool(s) |
|---------|---------|
| Single-host container runtime | **Docker Engine + Compose** (most common); Podman (rootless, daemonless) |
| Lightweight Kubernetes | **k3s** (~512MB RAM overhead); k0s; minikube (dev only) |
| Production Kubernetes | EKS (AWS); GKE (GCP); AKS (Azure); self-hosted via kubeadm |
| Serverless containers | Cloud Run (GCP); ECS Fargate (AWS); Azure Container Apps |

**Docker Compose is right-sized for solo and small-team agentic work.** Eight GB RAM supports 3–5 containers comfortably. Multi-container agent setups, MCP servers, sandbox environments — all live cleanly in Compose files.

**k3s is worth it when** you need namespacing for multi-tenant agent runs, rolling deploys, or multi-node scheduling. The ~512MB RAM overhead is real; don't add k3s until you actually need its features.

**Cloud Run / Fargate / Container Apps are worth it when** you need scale-to-zero economics for HTTP-fronted services (e.g., MCP servers as serverless endpoints).

---

## Serverless

**Pattern:** Compute that bills per-invocation, scales to zero, has minimal operational surface.

| Pattern | Tool(s) |
|---------|---------|
| Frontend + edge functions | **Vercel** (Next.js-native); Cloudflare Pages |
| Edge compute (low cold-start) | **Cloudflare Workers**; Deno Deploy |
| Event-driven backend | AWS Lambda; GCP Cloud Functions; Azure Functions |
| Container serverless | Cloud Run; ECS Fargate (above) |

**For agentic work specifically:**
- **MCP servers as serverless** is increasingly viable. HTTP-transport MCP servers can run as Cloudflare Workers or Cloud Run instances with stateless tool execution. State (when needed) lives in Durable Objects, Workers KV, or Redis.
- **Cloudflare Workers** has the best cold-start profile for latency-sensitive tool calls.
- **Lambda** is the right pick if you're already in AWS and the function is event-driven (queue-triggered, scheduled, S3-triggered).

---

## Cost control

The discipline that separates a hobby project from sustainable infrastructure.

### Pattern: sleep-on-idle, wake-on-demand

The compute VM runs an idle detector (systemd timer every 5 minutes). Checks for active SSH sessions AND running agent processes. After a configurable idle threshold, the VM stops itself. The connection wrapper on the laptop transparently wakes it on next connect.

User experiences a ~30 second delay on first connect; then normal operation. Cost reduction: ~70% versus always-on for typical 8-hour-per-day usage.

**Critical detail:** the idle check must be aware of headless agent processes. A naive "no SSH sessions" check shuts down a VM mid-agent-run. Correct check: `who` for sessions AND `pgrep` for the agent process name. Both must be inactive before the idle clock starts.

For providers without a clean auto-shutdown story (e.g., AWS without ASGs), the discipline is procedural: explicit start/stop scripts and a reminder banner on every connection. Less elegant; equally important. Cost differential between always-on and stopped is stark — often 10×+.

### Pattern: budget alerts and anomaly detection

| Pattern | Tool(s) |
|---------|---------|
| Budget threshold alerts | GCP Cloud Monitoring; AWS Budgets; Azure Cost Management |
| Anomaly detection | AWS Cost Anomaly Detection; GCP recommender |
| Per-task token budgets | Application-level (per-agent-run ceilings); see [04-automation/overnight-runs.md](../04-automation/overnight-runs.md) |

These are reactive, not preventive — but they surface problems before month-end billing surprises. Set them at meaningful thresholds: per-developer daily, per-team monthly, per-task hard ceilings. The hard ceiling is what protects against runaway loops.

### Pattern: free-tier exploitation as architecture

The always-free GCP micro tier (e2-micro in select regions) is genuinely zero-cost forever. Build *around* this, not despite it. The gateway pattern above is one example: use the free tier as the always-on listener, sleep the expensive compute. Some teams run entire personal infrastructures (DNS, monitoring, light proxies, MCP server frontends) on free-tier VMs because the offering is large enough.

Same logic applies to Cloudflare's free tier (Workers, R2, KV with generous limits) and Oracle Cloud's Always Free (which is unusually generous for compute).

---

## MCP server deployment

**Pattern:** Where MCP servers live, how they're authenticated, how they scale.

Three viable deployment tiers:

**Local stdio MCP** — runs on the developer's machine. Zero latency. Auth boundary is the OS process model. The simplest option; right for development and personal tooling.

**Cloud VM HTTP MCP** — runs on the same VM as the agent runtime (or alongside it). HTTP/SSE transport. Requires explicit auth (OAuth 2.0 per the MCP spec, or API key headers). Right when multiple agents share the server.

**Serverless MCP** — runs as Cloudflare Workers, Cloud Run, or Lambda. Scales to zero. Cold-start adds latency. Right for HTTP-fronted MCP servers used by multiple users or harnesses.

| Pattern | Implementation |
|---------|----------------|
| Local stdio | The MCP TypeScript/Python SDKs default to stdio; just run the process |
| Cloud VM HTTP | nginx in front of the MCP server process; Tailscale-only access OR OAuth/API key auth |
| Serverless HTTP | Cloudflare Workers (lowest cold-start); Cloud Run (containerized); Lambda (event-shaped) |

**Authentication discipline:**
- Local stdio needs no auth — process model is the boundary
- HTTP MCP exposed publicly requires OAuth 2.0 or strong API key auth at minimum
- VPN-only access (Tailscale) is a clean alternative: don't expose the MCP server to the internet at all, only to the Tailnet. Access is then controlled by Tailscale device enrollment.

**Secret injection:** MCP servers that need API keys for downstream services should fetch them from a managed secret service at startup (GCP Secret Manager, AWS Secrets Manager, HashiCorp Vault) — not hardcode, not in env files committed to git.

---

## Session recording and audit

**Pattern:** Reconstructable record of what an agent did, where, and when. Required for debugging, compliance, and forensics.

Three complementary layers (because each covers gaps the others leave):

**1. Terminal session recording.** `asciinema` records terminal sessions as text (~100KB/hour). Triggered by PAM hooks (`pam_exec.so` in `/etc/pam.d/sshd`) on SSH login/logout. Captures keystrokes and output; searchable and replayable. Limitation: SSH sessions only; browser terminal sessions bypass PAM.

**2. Agent state snapshotting.** Rsync or cloud sync of the agent's state directory (`~/.claude/`, equivalent for other harnesses) on a 5-minute systemd timer. Captures conversations, memory, settings. Covers headless agent runs that don't generate terminal output.

**3. Git auto-commit.** Every project directory committed on a 5-minute timer (systemd or cron). Covers all file changes regardless of who or what made them. Combined with layer 2, you have full coverage: layer 2 captures agent state, layer 3 captures code state.

**Append-only event log.** A `session-registry.jsonl` recording timestamps, session IDs, and source IPs for every login/checkpoint/logout. Lightweight but auditable security trail. Not SIEM-grade; sufficient for "what device connected, when, for how long."

For more on observability patterns: [07-tools-and-mcp/observability.md](../07-tools-and-mcp/observability.md).

---

## Security patterns

**VPN-only access as primary model.** Public IP for HTTPS only. Port 22 closed to the world. SSH reaches gateway via Tailscale; SSH reaches compute via ProxyJump through gateway. The compute VM has no public IP at all.

**Principle of least privilege, applied concretely.** Every service account, IAM role, and API key gets only the permissions its task requires. Examples:
- A gateway VM service account gets `compute.instanceAdmin` scoped to the dev VM only — not project-wide
- A dev VM service account gets `secretAccessor` on one specific secret, and `compute.instances.stop` on itself — nothing else
- No security group rules opening to `0.0.0.0/0` for non-public services

**API keys never on disk.** Fetch from managed secret service at session start; export to environment only; never write to `~/.bashrc` or any file. The key exists only in the running session's environment and is garbage-collected at logout.

**Defense in depth on web portals.** Reverse-proxy auth at the gateway plus application-level auth at the service. Neither alone is sufficient; the combination catches misconfigured proxy rules.

**Backup access path.** Never design a system where losing the VPN means losing all access. The backup path should require different credentials and a different trust path (AWS SSM, GCP IAP) so a VPN compromise doesn't also compromise the fallback.

**Audit trails as a security control.** A source IP from a known device, plus a session timestamp, plus a session ID — enough to answer "what happened, when, by whom" without external SIEM tooling.

---

## Picking a stack

**For solo developer with personal AI infrastructure:**
- GCP e2-micro (always-free) as gateway
- Single GCP e2-standard-2 (sleeping) as compute
- Tailscale for VPN
- nginx + Certbot for TLS
- Docker Compose for containers
- Cloudflare Workers for any public-facing serverless

Total cost at moderate use: $5–$15/month.

**For small team with shared infrastructure:**
- GCP or AWS with always-free or t3.micro gateway
- Auto-scaling group for compute (or always-on if usage is consistent)
- Tailscale (free tier supports 100 devices)
- Vercel for frontend; Cloud Run / Lambda for backend
- Centralized secrets manager
- Audit logging enabled at the cloud level

Total cost: $50–$300/month depending on team size and compute intensity.

**For production-facing agentic systems:**
- Whatever your existing cloud provider is (don't switch for AI)
- Container orchestration appropriate to scale (Kubernetes if you have ops capacity; Cloud Run / ECS Fargate if you don't)
- Managed observability (Datadog, Honeycomb, or cloud-native equivalents)
- Cost monitoring and per-service budget caps
- The same security discipline as any production service

The patterns scale; the tools change.

---

## What to skip

- **Multi-cloud as a default.** Pick one provider and learn it well. Multi-cloud adds operational complexity that's only worth it for genuine availability requirements (rare for agentic work) or strong cost arbitrage (rare in practice).
- **Kubernetes for solo work.** Compose is right-sized; k3s is for when you genuinely need namespacing or multi-node.
- **Always-on expensive compute.** The sleep-on-idle pattern saves real money with minimal user friction.
- **Custom infrastructure when managed services exist.** Hosting your own Postgres / Redis / vector DB usually costs more in time than the managed equivalent costs in money.
- **Bleeding-edge cloud features.** Stick to GA services with stable APIs. Beta features get deprecated; your infrastructure shouldn't depend on them.

## See also

- [02-platform/cross-platform-parity.md](../02-platform/cross-platform-parity.md) — keeping local dev environments consistent across the machines that connect to this infrastructure
- [04-automation/sandbox-environments.md](../04-automation/sandbox-environments.md) — the containment patterns this infrastructure hosts
- [04-automation/overnight-runs.md](../04-automation/overnight-runs.md) — the workloads this infrastructure exists to serve
- [07-tools-and-mcp/observability.md](../07-tools-and-mcp/observability.md) — the audit and monitoring layer
- [shell-and-terminal-tools.md](./shell-and-terminal-tools.md) — the local-side counterpart; what you run *from* to operate this infrastructure
- [tool-evaluations.md](./tool-evaluations.md) — deeper evaluation of agent harnesses and runtimes deployed on this infrastructure

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
