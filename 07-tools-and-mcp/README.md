# Tools and MCP Overview

An agent without tools is a chatbot. The set of tools you wire up — and how you wire them — determines what the agent can actually do. This section covers the patterns for tool design, the protocol that's quickly becoming the standard for agent tooling (MCP), and the harness-side mechanisms that let you customize agent behavior without modifying the model.

If you haven't read the [MCP explainer](../01-concepts/mcp-explained.md), start there. This section assumes you already know what MCP is.

## The patterns

| Pattern | What it does |
|---------|--------------|
| [MCP Server Patterns](./mcp-server-patterns.md) | How to design tools agents can actually use. Naming, scoping, descriptions, errors. |
| [Hooks](./hooks.md) | Event-driven automation in the agent harness. Run code before/after tool calls, on session start, on completion. |
| [Skills](./skills.md) | Discoverable, reusable capabilities. The difference between skills and tools, and when to use each. |
| [Observability](./observability.md) | Logging, tracing, and metrics for agent runs. What to capture and how to read it. |

## How tools, hooks, and skills relate

The terms get used loosely. Pinned definitions:

- **Tools** are functions the model can call. Defined by name, description, input schema. The model sees these in its prompt and decides when to invoke them.
- **Hooks** are harness-level event handlers. They run automatically on defined events (before a tool call, after a file edit, on session start). The model doesn't decide when they fire; the harness does.
- **Skills** are pre-packaged capabilities — usually a combination of tools, prompts, and instructions — that the model can discover and invoke as a unit. Bigger than a tool, smaller than a workflow.

A typical agentic system uses all three:
- Tools for primitive capabilities (read file, run command, call API)
- Hooks for guardrails and automation (block dangerous commands, log all writes)
- Skills for opinionated workflows (run the test suite, write a commit message in the project's style)

## The capability surface

When you design what an agent can do, you're shaping its capability surface. Three forces pull on this:

**Capability:** more tools = more things the agent can do.

**Confusion:** more tools = harder for the model to pick the right one.

**Risk:** more tools = larger blast radius if something goes wrong.

The right balance depends on the workflow. A code-editing agent benefits from a focused, well-named tool set (read, edit, search, run). A research agent benefits from a broader set (web fetch, document search, multiple data sources). An ops agent benefits from a narrow but powerful set with strong guardrails (deploy, rollback, query metrics — each carefully scoped).

Designing capability surfaces is genuinely a craft. Most teams overshoot and add too many tools, then complain that the model "uses the wrong one." Almost always the fix is fewer, better-named tools.

## Where MCP fits

MCP standardizes how tools are declared and called across agent harnesses. Without MCP, every harness has its own tool format and you re-implement everything. With MCP, write the tool once, use it anywhere.

The practical implications:

- **Don't fork tools per harness.** Build them as MCP servers. Connect them to whatever harness you're using.
- **Reuse community servers when they exist.** GitHub, filesystem, common databases, popular SaaS — many have well-built MCP servers already.
- **Build internal MCP servers for internal systems.** Your ticket tracker, your monitoring, your deployment system. One server, every agent gets the capability.

This is the highest-leverage architectural decision in the category. A team that invests in good internal MCP servers compounds value across every agent they ever use.

## Where to start

If you're designing your first set of tools: [mcp-server-patterns.md](./mcp-server-patterns.md). It's the longest read in this section because tool design has the most subtle craft.

If you're trying to add guardrails or automation: [hooks.md](./hooks.md).

If you're trying to package a workflow for reuse: [skills.md](./skills.md).

If your agent is doing things you can't explain in retrospect: [observability.md](./observability.md). You can't improve what you can't see.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
