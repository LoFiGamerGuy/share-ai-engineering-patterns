# Resources Overview

The first seven sections describe patterns. This section is the practical apparatus around them: what dev environment to set up, what to read, and which tools to actually pick when you have a choice.

Resources age faster than patterns. Treat the recommendations here as opinionated snapshots — the picks reflect what I'd recommend in May 2026, with reasons. Re-evaluate annually; the underlying *criteria* hold up longer than the specific picks.

## The pages

| Page | What it covers |
|------|----------------|
| [Dev Environment](./dev-environment.md) | A reference setup for productive agentic work. Cross-platform patterns. Links to working examples. |
| [Shell and Terminal Tools](./shell-and-terminal-tools.md) | Deep reference for shell/terminal tooling specifically for AI engineering. The modern Unix toolchain, multiplexers, runtime managers, fzf widgets, security-first scripting. |
| [Cloud and Deployment Tools](./cloud-and-deployment-tools.md) | Patterns for hosting agents, MCP servers, sandbox envs. Always-free gateway, sleeping compute, VPN-only access, container orchestration, serverless options. |
| [Recommended Reading](./recommended-reading.md) | Curated books mapped to the seven sections. Distillations and citations from a personal reference library of ~150 technical books. |
| [Tool Evaluations](./tool-evaluations.md) | Honest assessments of frameworks, harnesses, runtimes, and the smaller utilities that turn out to matter at scale. |
| [Ecosystem and Plugins](./ecosystem-and-plugins.md) | Reconnaissance map of notable open-source repos: plugin collections (ECC), codebase knowledge graphs (Graphify), memory systems, observability, MCP server catalogs. |

## What's NOT here

- **Vendor benchmarks.** Provider-published numbers age in days. See [05-local-llms/benchmarking.md](../05-local-llms/benchmarking.md) for how to do your own.
- **Specific provider pricing.** It changes. See [06-token-efficiency/cost-estimation.md](../06-token-efficiency/cost-estimation.md) for the math; plug in current rates yourself.
- **A definitive "best stack."** There isn't one. The picks here trade off in different directions; pick by what you're optimizing for.

## How to use this section

If you're setting up a new development environment for agentic work: start with [dev-environment.md](./dev-environment.md).

If you want to go deeper on a specific topic from sections 01–07: [recommended-reading.md](./recommended-reading.md) is mapped section-by-section.

If you're evaluating which agent framework, IDE, or token-saving tool to commit to: [tool-evaluations.md](./tool-evaluations.md) gives you the comparison axes and current picks.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
