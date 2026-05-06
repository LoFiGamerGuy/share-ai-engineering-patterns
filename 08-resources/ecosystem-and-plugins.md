# Ecosystem and Plugins

A curated reconnaissance map of the open-source projects, plugin collections, and ecosystem tools worth knowing about when you're working in agentic AI. This is different from [tool-evaluations.md](./tool-evaluations.md) — that page helps you commit to a tool; this page helps you orient.

Format per entry: name + link + one-line "what it is" + one-line "when to look at it." Tighter on purpose. The goal is to know what exists; deeper evaluation comes later if a tool earns it.

The picks are opinionated and incomplete. The criterion: would I tell a friend "look at this before you build something similar." If the answer is no, it's not on the list.

---

## Plugin collections and marketplaces

### Everything Claude Code (ECC)
**[github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code)**
The most comprehensive plugin collection for Claude Code in the wild. Skills, agents, hooks, commands, rules, MCP configs — evolved over 10+ months of intensive daily use. Cross-platform (Windows, macOS, Linux). Anthropic hackathon winner.
**Look at it when:** you're using Claude Code seriously and want to skip the "what should I configure" phase. Install via `/plugin marketplace add https://github.com/affaan-m/everything-claude-code`.

### Anthropic Plugins Official
**[github.com/anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)**
Anthropic's officially-managed directory of high-quality Claude Code plugins. Smaller and more vetted than community marketplaces.
**Look at it when:** you want plugins with an institutional quality bar; sensible default for production-adjacent work.

### Awesome Claude Plugins
**[github.com/quemsah/awesome-claude-plugins](https://github.com/quemsah/awesome-claude-plugins)**
Automated tracking of Claude Code plugin adoption metrics across GitHub. Useful for spotting what the community is actually using vs what's just published.
**Look at it when:** you want signal on which plugins have traction beyond the README claims.

### Claude Marketplace directories
**[claudemarketplaces.com](https://claudemarketplaces.com/)** · **[claudepluginhub.com](https://www.claudepluginhub.com/marketplaces)** · **[aitmpl.com/plugins](https://www.aitmpl.com/plugins/)**
Browsable directories of Claude Code plugins, skills, and MCP servers. Useful for discovery; quality varies.
**Look at it when:** you have a specific need ("I want a skill for X") and want to check whether something exists before building.

---

## Codebase knowledge graphs and code understanding

### Graphify
**[github.com/safishamsi/graphify](https://github.com/safishamsi/graphify)** · **[graphify.net](https://graphify.net/)**
Builds a queryable knowledge graph from a codebase (plus docs, SQL schemas, scripts, papers) using Tree-sitter for static analysis and LLM-driven semantic extraction with Leiden clustering. MIT licensed. Works across Claude Code, Codex, OpenCode, Cursor, Gemini CLI, Copilot, Aider, and others. Reportedly returns context for typical code questions in ~1.7k tokens vs ~123k for naive grep-and-read.
**Look at it when:** your codebase is large enough that token cost on context loading is dominating your bill, or when agents are flailing because they can't see the cross-file structure.

### Code-Review-Graph
**[github.com/tirth8205/code-review-graph](https://github.com/tirth8205/code-review-graph)**
Local knowledge graph specifically for Claude Code, focused on code review. Persistent map of your codebase so the agent reads only what matters. Reports 6.8× fewer tokens on reviews, up to 49× on daily coding tasks.
**Look at it when:** Graphify is too broad and you want a code-review-specific alternative; or you want to compare two implementations of the same idea.

---

## Memory and persistent state for agents

### Graphiti
**[github.com/getzep/graphiti](https://github.com/getzep/graphiti)**
Temporal knowledge graph framework for AI agents from Zep. Captures facts about entities and relationships *with timestamps*, so agents can reason about what was true *when*. Different from a static knowledge graph; the temporal dimension is the differentiator.
**Look at it when:** your agent needs persistent memory of an evolving domain (customer state, project state, anything that changes over time) and "latest snapshot" memory isn't enough.

### Mem0
**[github.com/mem0ai/mem0](https://github.com/mem0ai/mem0)**
Multi-tier memory layer for AI agents. Combines vector search, graph memory, and key-value storage. Provider-agnostic.
**Look at it when:** you want a memory layer that abstracts across vector / graph / KV without committing to one upfront.

### Letta (formerly MemGPT)
**[github.com/letta-ai/letta](https://github.com/letta-ai/letta)**
Stateful agent server with persistent memory and tool calling. The "MemGPT" model of treating context as a virtual memory hierarchy with explicit paging.
**Look at it when:** you need genuinely long-running agents with state that persists across sessions and tool calls; the explicit memory hierarchy is worth understanding even if you don't adopt the framework.

---

## Agent observability and evaluation

### LangSmith
**[smith.langchain.com](https://smith.langchain.com/)**
Tracing, eval, and monitoring for LangChain/LangGraph applications. Native integration is the easiest if you're already in that stack.
**Look at it when:** you're building on LangChain/LangGraph and want first-class observability without instrumentation work.

### Langfuse
**[github.com/langfuse/langfuse](https://github.com/langfuse/langfuse)**
Open-source LLM observability and analytics platform. Self-hostable. Works across providers and frameworks.
**Look at it when:** you want LangSmith-style observability without LangChain lock-in, or you need self-hosted for compliance.

### Phoenix (Arize)
**[github.com/Arize-ai/phoenix](https://github.com/Arize-ai/phoenix)**
Open-source ML observability platform with strong LLM tracing and evaluation. OpenTelemetry-based.
**Look at it when:** you want OTel-native instrumentation, or you're already in the Arize ecosystem.

### Helicone
**[github.com/Helicone/helicone](https://github.com/Helicone/helicone)**
Proxy-based observability for LLM API calls. Drop-in: change the base URL, get logging, caching, and analytics. Self-hostable.
**Look at it when:** you want zero-code-change observability; especially useful when wrapping multiple internal applications.

---

## MCP server collections and tooling

### Official MCP server catalog
**[github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)**
The reference collection of MCP servers maintained alongside the protocol itself. GitHub, filesystem, Postgres, Slack, and many more — first-party quality.
**Look at it when:** you need any common integration; check here before building.

### Awesome MCP Servers
**[github.com/punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)**
Community-curated list of MCP servers across categories. Quality varies; useful for discovery.
**Look at it when:** the official catalog doesn't cover what you need and you want to see what the community has built.

### MCP Inspector
**[github.com/modelcontextprotocol/inspector](https://github.com/modelcontextprotocol/inspector)**
Web-based debugger for MCP servers. Connect to a server, inspect its tools, test calls, see responses.
**Look at it when:** you're building an MCP server and want to verify behavior without running through a full agent harness.

---

## Open-source agent harnesses worth knowing

These complement the harnesses already covered in [tool-evaluations.md](./tool-evaluations.md). Listed here because they're notable for specific reasons.

### OpenDevin / OpenHands
**[github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)**
Open-source agent platform aiming at the "AI software engineer" use case. Sandboxed code execution, browser automation, multi-step task planning.
**Look at it when:** you want a fully open-source alternative to commercial agent harnesses, or you're researching how complete agent platforms are architected.

### Continue
**[github.com/continuedev/continue](https://github.com/continuedev/continue)**
Open-source IDE assistant for VS Code and JetBrains. Pluggable with any model provider. Strong customization story.
**Look at it when:** you want IDE integration without vendor lock-in to a specific harness.

### Goose (Block)
**[github.com/block/goose](https://github.com/block/goose)**
Block's open-source agent framework. CLI-first, MCP-native, model-agnostic.
**Look at it when:** you want a CLI agent with strong MCP integration and don't want to commit to one of the larger ecosystems.

---

## Specialized utilities and notable individual repos

### LiteLLM
**[github.com/BerriAI/litellm](https://github.com/BerriAI/litellm)**
Unified interface to 100+ LLM APIs. Use the OpenAI SDK shape against any provider. Routing, fallback, and cost tracking built in.
**Look at it when:** you're switching providers often, building tooling that needs to support multiple, or want centralized cost/latency observability across providers.

### Outlines
**[github.com/dottxt-ai/outlines](https://github.com/dottxt-ai/outlines)**
Structured generation library. Forces LLM outputs to conform to JSON schemas, regex, or grammars. Works at the token-sampling level, not just prompt-level.
**Look at it when:** you need genuinely guaranteed structured output (not "asked nicely and hoped"), especially for local model serving where provider-side structured output isn't available.

### Instructor
**[github.com/jxnl/instructor](https://github.com/jxnl/instructor)**
Structured output library for the OpenAI SDK and compatible providers. Pydantic-first, validation-driven.
**Look at it when:** you want clean structured output from hosted providers without the lower-level work Outlines requires.

### DSPy
**[github.com/stanfordnlp/dspy](https://github.com/stanfordnlp/dspy)**
Framework for "programming, not prompting" LLMs. Compose modules, define metrics, optimize prompts via search rather than hand-tuning.
**Look at it when:** you have a measurable task and want to systematically optimize prompts/pipelines instead of iterating manually; especially valuable when the "right" prompt is non-obvious.

### Aider Chat (already in tool-evaluations.md)
Cross-referenced; see that page for the full evaluation.

---

## How to use this list

This page is a starting point, not a destination. Two failure modes to avoid:

1. **Installing everything to "try it out."** Each tool you install adds maintenance load and decision fatigue. Pick one in a category, use it for two weeks, then decide if it earned its slot.
2. **Treating absence as judgment.** A tool not on this list isn't necessarily bad; it just didn't make my personal cut. The ecosystem moves fast; check current GitHub stars, recent commit activity, and community discussion before assuming a project is settled.

If you find a notable tool that should be here, the repo accepts contributions — see [README.md](../README.md).

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
