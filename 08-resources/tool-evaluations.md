# Tool Evaluations

Honest, technically-grounded assessments of the tools that come up when you're building agentic systems. The format for each: **what it is**, **when to reach for it**, **when to look elsewhere**, and a current pick where one exists.

Two ground rules:

- **Diplomatic, not blunted.** I won't trash a vendor or maintainer for the sake of it. I will tell you where a tool's strengths and weaknesses actually fall, because pretending everything is great helps no one.
- **Snapshots, not verdicts.** These are May 2026 assessments. Tools improve, projects pivot, new entrants appear. Re-evaluate annually.

---

## Agent harnesses (the IDE/CLI you actually use)

The harness shapes your day-to-day more than the underlying model does. Switching costs are real, so this is worth getting right.

### Claude Code (CLI)

**What it is.** A CLI agent harness from Anthropic. Tool-using, file-editing, project-aware. Strong support for subagents, hooks, custom skills, and MCP. Configurable via per-project and global `CLAUDE.md` files.

**Reach for it when:** you want a CLI-first agentic flow, you value the hook/skill/subagent extensibility, you're already comfortable in the terminal, or you're integrating with custom MCP servers and want first-class support.

**Look elsewhere when:** you need an IDE-integrated UX as the primary surface, your team is already deep in Cursor or Continue, or you're allergic to the terminal.

**Current pick rating:** strong default for CLI-driven agentic work.

### Cursor

**What it is.** A VS Code fork with deeply integrated AI editing — inline completion, multi-file edits, agent-style task execution from the editor.

**Reach for it when:** your team is IDE-centric, you do a lot of inline editing alongside agentic work, and you value the "agent in the editor" UX where the AI sits next to where you read code.

**Look elsewhere when:** you're cost-sensitive at scale (Cursor's model routing has historically defaulted to expensive providers), you need the harness extensibility of Claude Code, or you want full control over which model handles which task.

**Current pick rating:** strong for IDE-first developers; check pricing model carefully if scaling beyond individual use.

### Aider

**What it is.** Open-source, terminal-based AI pair programming tool. Smaller scope than Claude Code; tighter focus on git-tracked code editing with diff review.

**Reach for it when:** you want an open-source harness, you're cost-sensitive, you appreciate explicit diff review on every change, or you want to point it at any model provider without lock-in.

**Look elsewhere when:** you need the full agent-with-tools loop, you want subagent orchestration, or you need MCP integration.

**Current pick rating:** excellent if its scope matches your need; not a full Claude Code replacement.

### VS Code + Continue / Zed AI / GitHub Copilot

**What they are.** IDE-integrated AI assistants of varying scope. Continue is open-source and pluggable; Zed has integrated AI in a fast modern editor; Copilot is Microsoft's mature autocomplete + chat product.

**Reach for them when:** you want AI-in-the-editor without leaving your existing workflow, your work is more "edit and refine" than "dispatch and review," or you're embedded in an organization with one of these as the standardized tool.

**Look elsewhere when:** you need full agentic loops, you're doing multi-file changes that benefit from explicit planning, or you need MCP/extensibility.

**Current pick rating:** complementary to a primary harness rather than a replacement for one.

---

## Multi-agent frameworks

These are the libraries you'd use if you were building your own agentic system from the ground up.

### LangGraph

**What it is.** A graph-based orchestration framework, part of the LangChain family. The differentiating feature: it supports cycles in the execution graph, not just DAGs. This is essential for genuine agentic behavior (reflect → revise → reflect; retry on failure; re-plan when stuck).

**Reach for it when:** you're building stateful, multi-step agent workflows with feedback loops; you need fine-grained control over flow; you're comfortable defining typed state schemas.

**Look elsewhere when:** your workflow is genuinely linear (no cycles, no branching) — a simpler pipeline tool will be less ceremony; or you want minimal API surface area.

**Current pick rating:** the right pick when you actually need its features. Don't over-engineer with it for simple pipelines.

### CrewAI

**What it is.** A role-based multi-agent framework. You define agents by role and goal, give them tools, and orchestrate via tasks. Lower learning curve than LangGraph; less control over execution flow.

**Reach for it when:** you're prototyping multi-agent workflows quickly, the role-based mental model fits your problem (e.g., "researcher + writer + editor"), or you want a simpler API than LangGraph.

**Look elsewhere when:** you need fine-grained flow control, you need cycles for reflection loops, or your workflows are too irregular for the role-based abstraction.

**Current pick rating:** good for fast prototyping and certain class of structured collaboration patterns.

### AutoGen

**What it is.** Microsoft Research's multi-agent conversation framework. Agents communicate via structured messages; the framework manages the conversation flow.

**Reach for it when:** you're building multi-agent systems that resemble structured conversations (e.g., debate, negotiation, role-play), you want strong support for agent-agent communication patterns, or you're in a Microsoft ecosystem.

**Look elsewhere when:** you need deterministic pipelines or your agents don't really "converse" — they just hand off artifacts.

**Current pick rating:** strong in its niche; mismatched for many engineering workflows.

### LangChain

**What it is.** The original abstraction layer over LLM APIs. Sprawling, integration-heavy, frequently updated. The conceptual base for LangGraph and a vast ecosystem of community integrations.

**Reach for it when:** you need an integration that exists in LangChain's ecosystem and not elsewhere, you're learning the field and want a tour of the available primitives, or you're maintaining existing LangChain code.

**Look elsewhere when:** you're starting fresh and don't need the surface area; the framework's sprawl can become its own kind of complexity. Many teams that started in LangChain end up in LangGraph or hand-rolled code as workflows mature.

**Current pick rating:** useful for orientation; evaluate carefully before committing a new project to it.

### Haystack

**What it is.** A pipeline framework focused on retrieval-augmented use cases — search, QA, document understanding.

**Reach for it when:** RAG is the central use case, you want a framework that takes retrieval seriously as a first-class concern.

**Look elsewhere when:** you're building general agentic workflows; Haystack's RAG-centricity is a strength when you need it and a constraint when you don't.

**Current pick rating:** strong in its lane.

---

## Local LLM runtimes

For when you want to run models on your own hardware. See [05-local-llms](../05-local-llms/) for the broader context.

### Ollama

**What it is.** Easiest path to running open models locally. Single command to pull and serve models; OpenAI-compatible API; broad model support.

**Reach for it when:** you're getting started with local models, you want a simple HTTP API, you're prototyping, or you want quick model swaps for evaluation.

**Look elsewhere when:** you need maximum throughput or batching efficiency in production — Ollama prioritizes ease over peak performance.

**Current pick rating:** best default for local-first development.

### vLLM

**What it is.** High-throughput inference server, built for serious serving workloads. Supports continuous batching, PagedAttention, tensor parallelism.

**Reach for it when:** you're serving local models in production, you need maximum throughput per GPU, or you have multiple concurrent users hitting one model.

**Look elsewhere when:** you're running models for personal use, you don't need the throughput, or you don't want to manage the operational complexity.

**Current pick rating:** the right tool for production-scale local serving.

### LM Studio

**What it is.** GUI-driven local model serving. Friendly for non-CLI users; visual model selection; built-in chat UI.

**Reach for it when:** you prefer a GUI, you're evaluating models interactively, you want to share local model usage with non-engineers in your org.

**Look elsewhere when:** you're scripting against the runtime — Ollama's CLI-first design fits scripts more naturally.

**Current pick rating:** good for exploration and non-engineer-friendly local serving.

### llama.cpp

**What it is.** The C++ inference engine that underlies many of the above. Maximum portability, runs on virtually anything (CPU-only included), low-level control.

**Reach for it when:** you need to run on hardware the higher-level tools don't support, you're embedding inference into another application, or you're optimizing for resource-constrained environments.

**Look elsewhere when:** you want the convenience layers Ollama or LM Studio provide on top.

**Current pick rating:** the foundation; reach for it directly when the higher-level tools don't fit.

---

## Token-saving and context-compression utilities

These are smaller, often single-purpose tools that turn out to matter at scale. For the patterns themselves, see [06-token-efficiency](../06-token-efficiency/).

### rtk-ai (RTK)

**What it is.** A CLI proxy that sits between your agent and the LLM API, reducing token consumption — typically by deduplicating, compressing, or caching content that would otherwise be re-sent on every call.

**Reach for it when:** you're running high-volume agent workflows where input tokens dominate the bill, your prompts have repetitive structure that's not benefiting from provider-side caching, or you want centralized observability of token costs across multiple agent processes.

**Look elsewhere when:** your workflow is small-scale (the proxy overhead won't pay back), you're already getting strong cache-hit rates from provider-side caching, or you can't insert a proxy due to security/compliance constraints.

**Current pick rating:** worth evaluating for high-volume usage; measure before/after with real workloads.

### Caveman (output compression)

**What it is.** A tool focused on compressing model outputs and tool results before they re-enter the context. Reduces the token cost of long tool outputs, log dumps, and verbose responses.

**Reach for it when:** your agent runs accumulate large tool outputs (test logs, command stdout, file dumps) that bloat context across the loop, you've already caching what you can and the variable parts are still expensive, or you're hitting context-window limits on long-running sessions.

**Look elsewhere when:** your tool outputs are already structured and lean, you're using retrieval/summarization patterns instead, or your workflows are short enough that the compression overhead isn't worth it.

**Current pick rating:** specific lever; useful in the right workload, irrelevant in others.

### Provider-side prompt caching

**What it is.** Not a separate tool — a feature offered by major providers (Anthropic, OpenAI, Google) where stable prompt prefixes are cached and billed at a fraction of normal rates.

**Reach for it when:** always, by default. Structure your prompts so the stable parts come first, and cache automatically.

**Look elsewhere when:** never — even if it saves nothing in a particular workflow, it costs nothing to leave on.

**Current pick rating:** non-negotiable. See [06-token-efficiency/prompt-caching.md](../06-token-efficiency/prompt-caching.md).

---

## MCP server ecosystem

When you need an MCP server, the build-vs-borrow decision matters more than the specific implementation.

**For common integrations** (GitHub, filesystem, popular databases, common SaaS): use community-maintained servers. The MCP catalog is growing; check before building.

**For internal systems** (your tracker, your monitoring, your deployments): build your own. One server per logical system. See [07-tools-and-mcp/mcp-server-patterns.md](../07-tools-and-mcp/mcp-server-patterns.md).

**Languages with mature SDKs:** TypeScript, Python, Go. All work well; pick by your team's primary language.

**Quality varies wildly across community servers.** Read the source before installing. A poorly-built server is a security and reliability liability — treat them with the same rigor you'd treat any third-party dependency.

---

## How to evaluate a new tool

The framework I use, formalized:

1. **What problem does it actually solve?** Not "what does the README say it does." What's the friction it removes from work I'm currently doing?
2. **What's the alternative if I don't use it?** Sometimes the alternative is "do the work by hand" (high-cost, but you keep full control). Sometimes it's "another tool that does almost the same thing" (then the comparison is real).
3. **What does it cost to switch off?** Lock-in is the silent killer of tool decisions. A tool that bakes itself into your workflow over months is harder to remove than one that does its job and exits.
4. **Will it still exist in two years?** Maintainer activity, funding model (commercial vs open-source vs hobby), governance. Tools that die take your investment with them.
5. **Does the team actually want to use it?** A tool nobody picks up isn't a tool; it's a sunk cost.

Most tools fail one of these. The ones that pass all five are usually worth committing to.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
