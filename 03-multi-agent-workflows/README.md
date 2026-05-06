# Multi-Agent Workflows Overview

The single highest-leverage move in agentic engineering is moving from one agent to multiple coordinated agents. Single-agent workflows top out quickly. Multi-agent workflows scale further because they parallelize work, catch errors single agents miss, and let you specialize roles.

This section covers the patterns. Each pattern has its own page.

## The patterns

| Pattern | When to use |
|---------|-------------|
| [Plan / Execute / Judge](./plan-execute-judge.md) | Default for non-trivial tasks. One agent plans, one executes, one critiques. |
| [Agent Council](./agent-council.md) | Decisions and design questions. Multiple personas debate; you (or an arbiter) decide. |
| [Multi-LLM Review](./multi-llm-review.md) | Important changes. Same task across providers; differences highlight risks. |
| [Spec-Driven](./spec-driven.md) | Anything multi-step. Spec is the artifact agents work against. |
| [Handoff](./handoff.md) | Long tasks that span sessions or move between agents. |
| [Failure Modes](./failure-modes.md) | What goes wrong with multi-agent setups, and how to recognize it. |

## Why multi-agent

Single agents have three structural limits:

1. **No internal critic.** They don't reliably catch their own mistakes. The same model that wrote the bad code is unlikely to flag it as bad.
2. **Single perspective.** They approach problems one way. Alternative approaches require external prompting.
3. **Sequential thinking.** They can't truly parallelize. Multiple chains of reasoning need multiple agents.

Adding a second agent dedicated to critique closes (1). Adding a third with different framing closes (2). Running independent agents on parallel sub-tasks closes (3).

## The cost question

Multi-agent workflows cost more in tokens. A plan/execute/judge pattern uses 2-3x the tokens of a single agent doing the same task.

Two reasons it's still worth it:

- **The output quality difference is large.** Catching an error before it ships saves more than the cost of the extra critique pass.
- **You can use cheaper models for some roles.** The judge doesn't need to be a frontier model; a smaller model can effectively spot common errors. See [06-token-efficiency/model-tiering.md](../06-token-efficiency/model-tiering.md).

## When NOT to use multi-agent

- **Trivial tasks.** Renaming a function. Adding a missing import. Fixing a typo. The overhead is more than the work.
- **Exploratory work where you're already iterating.** If you're going to manually correct after every step anyway, you're already the critic.
- **Time-critical hotfixes.** Multi-agent adds latency. When seconds matter, single agent + careful review is faster.

For everything else — anything that produces real code that matters — multi-agent is the default.

## Coordination overhead

Adding agents adds coordination cost. Two practices keep it manageable:

- **Make the spec the contract.** Agents communicate through the spec, not through a long shared conversation. Each agent reads the spec, does its job, writes its result. The next agent reads the result. Conversations don't compound.
- **Keep roles narrow.** A judge that's just judging is fast. A judge that also revises is slow and confusing. One responsibility per agent.

## Common combinations

In practice, the patterns combine:

- **Spec-driven plan/execute/judge.** Most common. Write a spec, agent plans against it, agent implements the plan, agent judges against the spec.
- **Council into spec.** Use council for decisions; output is a spec; spec drives plan/execute/judge.
- **Multi-LLM review on top of plan/execute/judge.** After the judge approves, run a second judge from a different provider on changes that warrant it.

You don't pick one pattern. You compose them.

## Where to start

If you've never run multi-agent: start with [plan-execute-judge.md](./plan-execute-judge.md). It's the highest-leverage single change you can make to your workflow.

After that: [multi-llm-review.md](./multi-llm-review.md) for important changes, [agent-council.md](./agent-council.md) for design decisions you're stuck on.

Always: [spec-driven.md](./spec-driven.md). Specs are the connective tissue.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
