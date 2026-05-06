# Quickstart

You don't have to read the whole repo. This page tells you what to read first depending on who you are and what you need.

If you have **5 minutes**: pick your role below, read the three linked pages, and you'll have a working mental model.

If you have **30 minutes**: read the eight section README files (one per section). That's the full landscape.

If you have **an afternoon**: pick a section that matches your current work and drill in.

---

## I'm an engineer who has used Copilot or ChatGPT

You want to move from "AI as autocomplete" to "AI as a collaborator that does the implementation while you direct it." The leverage is real and the patterns are not obvious.

**Read these three first** (~15 minutes):

1. **[What is an Agent](./01-concepts/what-is-an-agent.md)** — the loop, tools, where agents fail. The mental model the rest of the repo assumes.
2. **[Plan / Execute / Judge](./03-multi-agent-workflows/plan-execute-judge.md)** — the workhorse multi-agent pattern. The single most leveraged change you can make to your workflow.
3. **[The Agentic OS Spine](./02-platform/agentic-os-spine.md)** — the small set of files that turn a regular project into one any agent can navigate productively. Where the rest of the repo's value compounds from.

**If you're convinced and want more:**
- [Trust and Specs](./01-concepts/trust-and-specs.md) — the centaur model and how to calibrate trust
- [Reflection Loops](./03-multi-agent-workflows/reflection-loops.md) — the most under-applied pattern; pairs with everything
- [Prompt Caching](./06-token-efficiency/prompt-caching.md) — the single biggest cost lever; usually structured wrong without realizing it

---

## I'm an engineering leader

You don't need to know the syntax. You need to know the shape of the work, where the leverage is, and what to ask your teams.

**Read these three first** (~15 minutes):

1. **[Guide for Leaders](./GUIDE-FOR-LEADERS.md)** — non-technical primer. Five questions to ask your teams. Where the leverage is. Where the risk is. Roughly one-page read.
2. **[Multi-Agent Workflows Overview](./03-multi-agent-workflows/README.md)** — the structural reason why teams getting real leverage from AI use multiple coordinated agents, not single chatbots.
3. **[Autonomous Control Levels](./04-automation/autonomous-control-levels.md)** — the framework for thinking about how much autonomy to give agents. Includes a color-coded L0–L7 spectrum diagram.

**If you want a polished version to share:**
- [Leader's memo (HTML)](https://lofigamerguy.github.io/share-ai-engineering-patterns/leaders-memo.html) — the leaders guide as a one-page rendered memo. Send this to your peers.

**If you're trying to understand cost:**
- [Cost Estimation](./06-token-efficiency/cost-estimation.md) — the unit economics, what "expensive" looks like, what to monitor.

---

## I'm a platform builder putting an agentic spine into a project or org

Your job is to make the *infrastructure* work so the engineers using agents are productive. The work is mostly invisible when it's done right.

**Read these three first** (~20 minutes):

1. **[The Agentic OS Spine](./02-platform/agentic-os-spine.md)** — the file conventions every project should have for agents to be productive in it. Includes a layout diagram.
2. **[MCP Server Patterns](./07-tools-and-mcp/mcp-server-patterns.md)** — how to design tools agents will actually use well. The longest read in the repo because tool design has the most subtle craft.
3. **[Cross-Platform Parity](./02-platform/cross-platform-parity.md)** — keeping setups consistent across Mac / Linux / Windows / WSL so agents (and engineers) get the same behavior on every machine.

**If you're scaling to a team:**
- [Cloud and Deployment Tools](./08-resources/cloud-and-deployment-tools.md) — patterns for hosting agents and MCP servers cost-effectively
- [Hooks](./07-tools-and-mcp/hooks.md) — event-driven harness automation that doesn't depend on the model remembering
- [Observability](./07-tools-and-mcp/observability.md) — three-layer instrumentation that lets you tell when things are going wrong

---

## I'm worried about safety / security / putting agents into production

The risks are real and most of them don't have clean solutions yet. Be honest with yourself and your stakeholders about what's solved and what's still open.

**Read these three first** (~20 minutes):

1. **[Prompt Injection](./04-automation/prompt-injection.md)** — the structural vulnerability of every agentic system. What works, what doesn't, what's still unsolved. Required reading.
2. **[Autonomous Control Levels](./04-automation/autonomous-control-levels.md)** — the framework for thinking about where on the autonomy spectrum your work should live. Most general-purpose agentic engineering belongs at L2-L3, not higher.
3. **[Sandbox Environments](./04-automation/sandbox-environments.md)** — containment patterns for letting agents act with reduced blast radius.

---

## I'm just curious — show me the most credible content

The repo's whole premise is "every pattern came from running into the failure mode and figuring out the way around it." The five **war stories** are concrete, specific instances of that — real things that happened in real work, distilled into the lesson.

- [When "generic" isn't](./01-concepts/what-is-an-agent.md) — implicit context bleed in supposedly generic synthesis docs (sec 01, scroll to "War story" callout)
- [Pipeline patterns scale; single-pass doesn't](./03-multi-agent-workflows/plan-execute-judge.md) — a 150-book extraction job that taught the value of plan/execute/judge structure (sec 03)
- [Per-task budgets are not optional at scale](./04-automation/autonomous-control-levels.md) — a runaway extraction caught by a budget cap (sec 04)
- [Most of a book is not load-bearing](./06-token-efficiency/context-management.md) — a 100× compression ratio on a real bash cookbook synthesis (sec 06)
- [Audit scripts compound](./07-tools-and-mcp/observability.md) — a 30-line script that caught a real bug in this very repo (sec 07)

---

## After this

The Foyer catalogue at [lofigamerguy.github.io/share-ai-engineering-patterns](https://lofigamerguy.github.io/share-ai-engineering-patterns/) lists every page in every section with a one-line description. Use it as the navigation hub once you've finished a starting path and want to drill deeper.

The [GLOSSARY](./GLOSSARY.md) defines load-bearing terms in one place. Useful as a lookup when reading.

The [CHANGELOG](./CHANGELOG.md) shows what's been added recently if you've read this before and want to see what's new.

For corrections, additions, or war stories from your own practice: see [CONTRIBUTING](./CONTRIBUTING.md).

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time.*
