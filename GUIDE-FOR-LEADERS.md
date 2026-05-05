# Guide for Leaders

A non-technical primer on how AI is actually being used for engineering work, what the patterns are, and what to look for when you're trying to understand whether your teams are getting leverage from it.

This is written for managers, directors, and execs who are not coding day-to-day but need to make decisions about tooling, hiring, headcount, and strategy. You don't need to know the syntax. You need to know the shape of the work.

## The shift

Three years ago, AI in engineering meant autocomplete: a developer typed code, the IDE suggested the next few lines. Useful, but the human was driving every keystroke.

Today the work pattern is different. A developer writes a description of what they want — a paragraph, a checklist, a spec — and an agent does the implementation. The developer reviews the result, adjusts the spec, runs again. The unit of work moved from "lines of code" to "tasks."

This is the single most important thing to understand: **the bottleneck moved from typing to specifying**. Engineers who are getting the most leverage from AI are the ones who got good at writing precise specs and reviewing AI-generated work efficiently. Engineers who are still using AI as autocomplete are leaving most of the value on the table.

## What an "agent" actually is

An agent is an LLM running in a loop with access to tools. The loop looks like:

1. The agent reads the current task and any context.
2. It decides what to do next — usually call a tool (read a file, run a command, search the codebase, post a comment).
3. The tool returns a result.
4. The agent reads the result and decides the next action.
5. Repeat until the task is done or the agent gets stuck.

That's it. Everything sophisticated comes from how you set up the context, which tools are available, and what guardrails exist on the loop.

## What the patterns look like

The repo this guide lives in describes seven categories of patterns. The short version:

**Concepts.** The vocabulary: agents, context windows, MCP (the protocol that lets agents talk to your real tools), trust calibration. If you've ever sat in a meeting and people were saying "MCP" and "context" and you weren't sure exactly what they meant, the [01-concepts](./01-concepts/) section is the place.

**Platform.** Setting up a project so agents can be productive in it. This is the boring infrastructure layer — environment configs, project templates, the "agentic spine" that makes a repo agent-ready. Teams that skip this step end up with agents that flail because they don't know how to run tests, find the build script, or understand the codebase conventions. See [02-platform](./02-platform/).

**Multi-agent workflows.** The real leverage. Instead of one agent doing everything, you have multiple agents specialized for different roles: one plans the work, one executes it, one reviews it. You can also run the same task across multiple AI providers and have them critique each other. This is where teams go from "AI helps me code faster" to "AI does most of the implementation while I direct it." See [03-multi-agent-workflows](./03-multi-agent-workflows/).

**Automation.** Letting agents run unattended — overnight, in sandboxes, in CI. This is where you have to think hard about trust and blast radius. There are levels of autonomy and you should know which level your teams are operating at. See [04-automation](./04-automation/).

**Local LLMs.** Running models on your own hardware instead of paying per-token to a provider. Useful for privacy-sensitive work, very high-volume tasks, or experimentation. The capability gap between local and frontier is real, and shrinking, but not closed. See [05-local-llms](./05-local-llms/).

**Token efficiency.** AI work has a unit cost. At small scale you don't notice. At team scale, the bill matters, and there are well-known patterns (model tiering, prompt caching, context management) that can drop costs by 5-10x without sacrificing quality. See [06-token-efficiency](./06-token-efficiency/).

**Tools and MCP.** The plumbing that lets agents reach beyond text into your actual systems — your codebase, your databases, your ticketing system, your monitoring. MCP is the open protocol for this. See [07-tools-and-mcp](./07-tools-and-mcp/).

## Questions to ask your teams

Not "how much AI are you using" — that question doesn't reveal anything. Instead:

- **What's your spec workflow?** If they say "I just type questions into chat," they're early. If they say "I write a markdown file describing the task, then dispatch an agent against it, then review the diff," they're operating at the modern pattern.
- **Are you using multiple agents or just one?** Single-agent is fine for small tasks. Multi-agent (plan/execute/judge, council, multi-LLM review) is where the leverage is for non-trivial work.
- **What's your trust calibration?** Do they let agents commit directly? Open PRs? Run tests? Run anything destructive? "Always reviews every line" and "lets it run overnight in a sandbox" are both legitimate answers — but they should know where on that spectrum they are and why.
- **What does it cost?** They should be able to give you a rough monthly per-developer number. If they have no idea, they're probably either over-spending or under-using.
- **What broke recently?** Every team running serious agentic workflows has stories about an agent doing something dumb. Teams that don't have these stories aren't pushing hard enough. Teams that have *too many* of these stories don't have good guardrails. You want a moderate number of "we caught this in review" stories.

## Where the leverage is

The single highest-leverage move for most teams: **getting good at multi-agent review workflows** ([03-multi-agent-workflows](./03-multi-agent-workflows/)). Having one agent draft, a second agent critique, and a third agent reconcile catches a category of errors that no single-agent setup catches, and it parallelizes well.

The second highest-leverage move: **investing in the platform layer** ([02-platform](./02-platform/)). A well-structured repo with clear context files, working scripts, and known conventions is one where any agent can be productive immediately. A messy repo is one where every agent run starts from zero.

## Where the risk is

- **Trust drift.** Teams that start by reviewing every line slowly stop reviewing. The quality of agent output is high enough that careful review feels redundant. It isn't. Set explicit norms about what gets reviewed and stick to them.
- **Cost surprises.** Without monitoring, costs can 10x in a week when someone leaves a long-running loop unattended. Monthly budget alerts are the minimum.
- **Skill atrophy.** Junior engineers who only ever direct agents may not develop the deep mechanical skill that makes them good at directing agents. Teams should think about how juniors get reps on the actual mechanics, not just on supervision.
- **Context staleness.** Agents work off context files (`CLAUDE.md`, `AGENTS.md`, etc.). Those files rot. A team that hasn't updated its context file in three months is a team whose agents are working off bad assumptions.

## What to read next

If you have 30 minutes: read the seven `overview.md` files, one per section.

If you have 5 minutes: read [01-concepts/what-is-an-agent.md](./01-concepts/what-is-an-agent.md) and [03-multi-agent-workflows/overview.md](./03-multi-agent-workflows/overview.md). Those two together give you a working model of what's happening.

If you want to understand cost: [06-token-efficiency/cost-estimation.md](./06-token-efficiency/cost-estimation.md).
