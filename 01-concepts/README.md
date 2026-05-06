# Concepts Overview

The vocabulary of agentic engineering. If you've heard people talk about "agents", "context windows", "MCP", "trust calibration" and weren't sure exactly what they meant, this section pins down the terms.

These four pages are the conceptual foundation for everything else in the repo. The other sections assume you've read these.

## The pages

| Page | What it covers |
|------|----------------|
| [What is an Agent](./what-is-an-agent.md) | The loop, tools, where agents differ, where they fail |
| [Context and Memory](./context-and-memory.md) | Context windows, what an agent "remembers", how state persists |
| [MCP Explained](./mcp-explained.md) | The Model Context Protocol — what it is, why it matters, what it doesn't solve |
| [Trust and Specs](./trust-and-specs.md) | How to calibrate trust, why specs are the connective tissue, the centaur model |

## How to read this section

Read [what-is-an-agent.md](./what-is-an-agent.md) first. Everything else assumes that mental model.

After that, pick based on what you're trying to do:

- **Building or extending an agentic system?** → [mcp-explained.md](./mcp-explained.md), then [trust-and-specs.md](./trust-and-specs.md).
- **Debugging an agent that "forgets things" or behaves inconsistently?** → [context-and-memory.md](./context-and-memory.md).
- **Trying to decide how much autonomy to give an agent?** → [trust-and-specs.md](./trust-and-specs.md), then [04-automation/autonomous-control-levels.md](../04-automation/autonomous-control-levels.md).

## Beyond this section

Once these four are familiar, the rest of the repo is the "how": platform setup, multi-agent workflows, automation, cost control, tool design. The concepts here are the load-bearing words for all of it.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
