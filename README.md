# AI Engineering Patterns

A practitioner's reference for building with AI agents — focused on the patterns, infrastructure, and tradeoffs that make AI useful for real engineering work rather than demos.

This repo collects the durable lessons from building agentic systems: how to structure context, when to trust which model, how multiple agents coordinate without falling over, how to keep cost and latency under control, and how to instrument the whole thing so you can tell when it's broken.

## Who this is for

- **Engineering leaders** who want to understand how AI is actually being used by their teams and where the leverage is. Start with [GUIDE-FOR-LEADERS.md](./GUIDE-FOR-LEADERS.md).
- **Engineers** who have used Copilot or ChatGPT and want to move into agent-driven workflows, multi-agent orchestration, and autonomous loops. Start with [01-concepts](./01-concepts/).
- **Platform builders** who are putting an agentic spine into a project or organization and want to skip the dead ends. Start with [02-platform](./02-platform/).

## Sections

| # | Section | What's in it |
|---|---------|--------------|
| 01 | [Concepts](./01-concepts/) | Agents, context, MCP, trust models — the vocabulary |
| 02 | [Platform](./02-platform/) | Dev environment, project scaffolding, the agentic spine |
| 03 | [Multi-agent workflows](./03-multi-agent-workflows/) | Plan/execute/judge, councils, multi-LLM review, handoff |
| 04 | [Automation](./04-automation/) | Sandboxes, overnight runs, autonomous control levels |
| 05 | [Local LLMs](./05-local-llms/) | Model selection, benchmarking, when local makes sense |
| 06 | [Token efficiency](./06-token-efficiency/) | Model tiering, prompt caching, context management, cost |
| 07 | [Tools and MCP](./07-tools-and-mcp/) | MCP server patterns, hooks, skills, observability |

## How to read this

Each section has its own `overview.md`. If you're orienting, read the seven overviews — that's roughly an hour and gives you the full landscape. Then drill into specific files based on what you're trying to do.

Nothing here is theoretical. Every pattern came from running into the failure mode and figuring out the way around it.

## Conventions

- "Agent" means an LLM running in a loop with tools, not a single completion call.
- "Frontier model" means the current top-tier model from a major provider. "Local model" means something you run on your own hardware.
- "Spec" means a written description of the work that's specific enough for a different agent (or a different person) to pick up and execute.
- Code samples use shell, Python, or TypeScript depending on what's idiomatic for that pattern. They're illustrative — not copy-paste-ready.

## Caveat

The patterns here reflect the state of the tools as of writing. Models, harnesses, and protocols are moving fast. Treat the *pattern* as durable; treat the specific tool names and flags as snapshots.
