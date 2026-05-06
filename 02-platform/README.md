# Platform Overview

The "platform" is the boring infrastructure layer that makes agents productive in your environment. None of it is exciting individually. Together it's the difference between an agent that does useful work and one that flails.

## What the platform layer covers

1. **Dev environment.** The shell, editor, and tooling configured so an agent can run commands without surprises. See [dev-environment.md](./dev-environment.md).
2. **Project template.** A scaffolding pattern for new repos so they're agent-ready from day one. See [project-template.md](./project-template.md).
3. **Agentic spine.** The set of files (`CLAUDE.md`, `AGENTS.md`, conventions, scripts) that every project carries to make it legible to agents. See [agentic-os-spine.md](./agentic-os-spine.md).
4. **Cross-platform parity.** Keeping your setup consistent across Mac / Linux / Windows / WSL so agents (and you) get the same behavior on every machine. See [cross-platform-parity.md](./cross-platform-parity.md).

## Why this layer matters

The most common failure mode in agentic engineering isn't "the model is dumb." It's:

- The agent doesn't know how to run the tests
- The agent runs the wrong build command
- The agent makes a change that violates an unwritten convention
- The agent reads files in a way that wastes context
- The agent uses tools that aren't quite what was needed

Every one of these is a platform problem. A model can't infer that you call your tests with `make test:unit` instead of `npm test`. It can't know that your project uses spaces and not tabs unless someone says so. It can't know which directories are "source of truth" vs. "generated."

When agents have good platform context, the same model produces dramatically better work. The leverage is in the setup.

## Two principles

**Make implicit knowledge explicit.** Anything an experienced team member "just knows" about the project should be written down somewhere the agent will read. This is true for human onboarding too, but agents fail more visibly when it's missing.

**Make the right thing the easy thing.** Agents will use whatever tools and patterns are most discoverable. If your platform makes the correct path obvious — clear file names, idiomatic scripts, well-described tools — agents will follow it. If the correct path is buried, they'll improvise something close-but-wrong.

## Platform vs. agent

The line: the **platform** is the project setup. The **agent** is the harness running against it. The same platform should work with multiple agents (Claude Code, Cursor, Codex, custom). Don't bake assumptions about a specific agent into your platform — write context files that any agent can read, scripts any harness can run, MCP servers any client can connect to.

## What this looks like in practice

A project with a strong platform layer has, at minimum:

- A `README.md` for humans
- A `CLAUDE.md` (or `AGENTS.md`) for agents — see [agentic-os-spine.md](./agentic-os-spine.md)
- A `Makefile` or equivalent with the standard tasks (`build`, `test`, `lint`, `format`, `dev`)
- A `.env.example` showing required environment variables (no real secrets)
- A `scripts/` directory for project-specific automation
- An MCP server config (or equivalent) listing tools the agent should have available

Without these, every agent run starts by inferring the project structure. That's wasted context, wasted tokens, and wasted runs.

## Where platform investment pays back

For a one-off prototype, platform investment is overkill. For anything you'll touch more than a handful of times, it pays for itself within a week.

The break-even is usually around the third agent run. After three runs, the time you spent setting up `CLAUDE.md` and a clean `Makefile` has saved more time than it cost.

For team projects, the math is even more favorable, because every team member's agent benefits from the same platform.

## What's not in this section

- How to *use* agents effectively (that's [01-concepts](../01-concepts/) and [03-multi-agent-workflows](../03-multi-agent-workflows/))
- How to manage cost (that's [06-token-efficiency](../06-token-efficiency/))
- How to wire up tools (that's [07-tools-and-mcp](../07-tools-and-mcp/))

This section is just about getting the project itself in good shape. Once that's done, everything else gets easier.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
