# Glossary

Single-page lookup for the load-bearing terms used across this repo. Most are defined inline in the section that introduces them; this page collects them for fast reference.

If you're looking for a term that isn't here, please [open an issue](https://github.com/LoFiGamerGuy/share-ai-engineering-patterns/issues) — additions are welcome.

---

## A

**Agent.** An LLM running in a loop with access to tools. The loop is: read context → decide an action → execute the action → observe the result → repeat. A single LLM completion is not an agent; the loop with tool use is what makes it one. See [01-concepts/what-is-an-agent.md](./01-concepts/what-is-an-agent.md).

**Agentic engineering.** The practice of building software with AI agents as primary collaborators rather than as autocomplete assistants. The shift is in the unit of work: from "lines of code" (assistant model) to "tasks" (agent model).

**Agent council.** A multi-agent pattern where several personas debate a decision before a verdict is reached. Used for design questions and judgment calls. See [03-multi-agent-workflows/agent-council.md](./03-multi-agent-workflows/agent-council.md).

**Audit trail.** A persistent record of consequential actions taken by agents — file modifications, commits, external calls, deploys. Required for debugging, compliance, and forensics. See [07-tools-and-mcp/observability.md](./07-tools-and-mcp/observability.md).

**Autonomous control level.** A point on the spectrum from "agent asks before any action" (L0) to "fully autonomous, no human review" (L7). Different tasks belong at different levels. See [04-automation/autonomous-control-levels.md](./04-automation/autonomous-control-levels.md).

## C

**Cache (prompt cache).** A provider-side optimization where stable prompt prefixes are stored and billed at a fraction of normal rates on subsequent calls. The single biggest cost lever in agentic workflows. See [06-token-efficiency/prompt-caching.md](./06-token-efficiency/prompt-caching.md).

**Centaur model.** The design philosophy of "humans judge, agents build." At defined checkpoints, human judgment is required; between checkpoints, agents operate autonomously. The opposite of "AI does everything" *and* the opposite of "human reviews every line." See [01-concepts/trust-and-specs.md](./01-concepts/trust-and-specs.md).

**Compaction.** The process of summarizing older conversation history into a denser representation so the working context stays bounded. One of the three layers of context management. See [06-token-efficiency/context-management.md](./06-token-efficiency/context-management.md).

**Context window.** The maximum amount of text (measured in tokens) a model can consider in a single turn. Different models have different window sizes. The window includes both input (prompt + history + tool results) and reserved output space.

## E

**Eval / evaluation.** A measurement of how well an agent or model performs on a defined task. Distinguished from a benchmark by being task-specific to your work rather than vendor-published. The "your evals over vendor benchmarks" rule is about not trusting marketing numbers when you can measure performance on what you actually do. See [05-local-llms/benchmarking.md](./05-local-llms/benchmarking.md).

## F

**Frontier model.** The current top-tier model from a major provider. Examples vary over time but typically include flagship offerings like Anthropic's top Claude tier, OpenAI's top GPT tier, Google's top Gemini tier. Used loosely; precise meaning shifts as new models release.

## G

**Graph (in agent frameworks).** A computation structure where nodes are operations and edges define flow. A DAG (directed acyclic graph) only flows forward; a graph with cycles can loop back. Cycles are essential for genuine agentic behavior (reflection, retry, re-planning). See [03-multi-agent-workflows/framework-selection.md](./03-multi-agent-workflows/framework-selection.md).

**Guardrail.** A constraint that prevents an agent from taking certain actions. Implemented at the harness level (hooks), the model level (system prompts — weak), the tool level (permission scoping), or the environment level (sandboxing). The strongest guardrails are environmental, not behavioral.

## H

**Handoff.** The discipline of writing down enough state at the end of a session that another agent (or your future self) can pick up the work cleanly. Required for any task spanning multiple sessions. See [03-multi-agent-workflows/handoff.md](./03-multi-agent-workflows/handoff.md).

**Harness.** The software wrapping the LLM that handles tool calls, manages context, runs hooks, and presents the user interface. Claude Code, Cursor, Aider, Continue are all harnesses. Two agents using the same model in different harnesses behave very differently.

**Hook.** A harness-level event handler that runs automatically on a defined event (before/after a tool call, on session start, on completion). The model doesn't decide when hooks fire; the harness does. Used for guardrails, automation, and consistency enforcement. See [07-tools-and-mcp/hooks.md](./07-tools-and-mcp/hooks.md).

## I

**Indirect prompt injection.** A prompt injection where the malicious instruction is embedded in content the agent ingests later — a web page, a document, a tool output — rather than supplied by the attacker as direct input. Harder to defend against than direct injection because every external content source is an attack surface. See [04-automation/prompt-injection.md](./04-automation/prompt-injection.md).

## J

**Jagged frontier.** The observation that AI capability is uneven across tasks. A model that's brilliant at one task may be terrible at a similar-looking task. The frontier is "jagged" rather than smooth, so calibrating trust by task class — not by overall reputation — is critical.

**Judge (in plan/execute/judge).** The third role in the workhorse multi-agent pattern. Reads the spec and the executor's diff; produces a verdict (approve / reject) with specific reasons. Doesn't fix issues — just judges. See [03-multi-agent-workflows/plan-execute-judge.md](./03-multi-agent-workflows/plan-execute-judge.md).

## L

**Local model.** A model running on your own hardware (laptop, server, on-prem GPU) rather than hosted by a provider. Tradeoffs: privacy, cost predictability, no rate limits — versus capability gap with frontier and operational complexity. See [05-local-llms](./05-local-llms/).

## M

**MCP (Model Context Protocol).** An open protocol for connecting AI agents to tools and data sources. Defines a wire format and conventions so a tool author can write one MCP server and have it work with any MCP-aware agent. "USB for agents." See [01-concepts/mcp-explained.md](./01-concepts/mcp-explained.md).

**Multi-agent workflow.** A pattern where multiple specialized agents coordinate on a task rather than one agent doing everything. Closes the structural limits of single agents (no internal critic, single perspective, sequential thinking). See [03-multi-agent-workflows](./03-multi-agent-workflows/).

**Multi-LLM review.** Running the same task across two providers' models and comparing the outputs. Differences highlight risks no single model would catch. See [03-multi-agent-workflows/multi-llm-review.md](./03-multi-agent-workflows/multi-llm-review.md).

## P

**Plan / Execute / Judge.** The workhorse multi-agent pattern. Three roles: planner (reads task, produces plan), executor (reads plan, makes changes), judge (reads task and diff, verdicts). Roles are kept narrow for discipline. See [03-multi-agent-workflows/plan-execute-judge.md](./03-multi-agent-workflows/plan-execute-judge.md).

**Prompt caching.** See "Cache."

**Prompt injection.** An attack class where malicious input causes the model to act against its intended guidelines. Direct (attacker types instructions) or indirect (instructions embedded in content the model later ingests). The structural vulnerability of every agentic system. See [04-automation/prompt-injection.md](./04-automation/prompt-injection.md).

## R

**RAG (Retrieval-Augmented Generation).** Pattern where the agent retrieves relevant content from a corpus (via vector search, keyword search, or graph) before generating a response. The retrieved content is injected into the prompt as context. RAG corpora are also injection surfaces — every retrieved document is potential indirect injection.

**Reflection.** A pattern where the model critiques its own output and revises based on the critique. Generate → reflect → revise. Implemented as multiple sequential prompts or as a loop in a graph framework. See [03-multi-agent-workflows/reflection-loops.md](./03-multi-agent-workflows/reflection-loops.md).

## S

**Sandbox.** An isolated environment where an agent can run with reduced blast radius. Containers, VMs, ephemeral cloud instances, dedicated branches in version control — all are sandbox patterns. The choice depends on what you're isolating *from*. See [04-automation/sandbox-environments.md](./04-automation/sandbox-environments.md).

**Skill.** A packaged capability the agent can discover and invoke as a unit. Bigger than a tool, smaller than a workflow. Typically combines a focused prompt, references to tools, and templates or examples. See [07-tools-and-mcp/skills.md](./07-tools-and-mcp/skills.md).

**Spec.** A written description of a task, specific enough that a different agent (or person) can pick up and execute it. The connective tissue of multi-agent work — agents communicate through specs rather than long shared conversations. See [03-multi-agent-workflows/spec-driven.md](./03-multi-agent-workflows/spec-driven.md).

**Spine (agentic spine).** The small set of files and conventions that turn a regular project into one any agent can navigate productively. Typically includes `CLAUDE.md` / `AGENTS.md`, a Makefile or justfile, `.env.example`, `specs/`, and an MCP config. See [02-platform/agentic-os-spine.md](./02-platform/agentic-os-spine.md).

**Subagent.** An agent spawned by another agent for a specific sub-task. The parent coordinates; the subagent specializes. Each subagent typically has its own context window, tools, and permissions.

## T

**Token.** The unit of LLM input and output. Approximately 0.75 of a word in English. Cost and context window are measured in tokens. Tokenization differs by model family; the same text becomes different token counts on different models.

**Tool.** A function the model can call to affect the world or learn about it. Each tool has a name, a description, an input schema, and returns a result. The set of available tools defines what the agent can do. See [07-tools-and-mcp/mcp-server-patterns.md](./07-tools-and-mcp/mcp-server-patterns.md).

**Tool description as prompt.** The principle that tool descriptions are read by the model as prompt content, not as documentation. The description determines whether the model uses the tool correctly. See [07-tools-and-mcp/tool-description-as-prompt.md](./07-tools-and-mcp/tool-description-as-prompt.md).

**Tiering (model tiering).** Routing different roles or tasks to different model tiers (frontier / mid-tier / small) based on task difficulty. Defaulting to the best model for everything is the most common and most expensive cost mistake. See [06-token-efficiency/model-tiering.md](./06-token-efficiency/model-tiering.md).

**Trust calibration.** Adjusting how much autonomy you give an agent based on its track record for the specific task class. Not the same as "trust the model" or "don't trust the model." It's task-specific and changes over time. See [01-concepts/trust-and-specs.md](./01-concepts/trust-and-specs.md).

## W

**War story.** A specific, concrete account of a failure mode encountered in real practice and how it was resolved. The most credible content in a technical reference. See the sidebar callouts in `01-concepts/what-is-an-agent.md`, `03-multi-agent-workflows/plan-execute-judge.md`, and three other sections.

**Working set.** The portion of the agent's context that's actually load-bearing for the current step. Distinguished from total context (which can be larger — cached prefix, project conventions, handoff doc). Keeping the working set tight reduces cost and improves attention. See [06-token-efficiency/context-management.md](./06-token-efficiency/context-management.md).

---

*Snapshot: May 2026. Definitions evolve as the field does. See [CHANGELOG](./CHANGELOG.md) for revisions.*
