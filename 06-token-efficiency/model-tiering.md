# Model Tiering

Not every step in an agentic workflow needs a frontier model. The most common cost mistake is defaulting to the best available model on every call. The second most common is the opposite — using a small model for work it can't actually do.

Tiering is the discipline of matching model capability to task difficulty.

## The three tiers

Think of available models as falling into three rough tiers:

**Frontier.** Top-tier models from major providers. The current best at long-horizon reasoning, complex code, novel problem solving. Expensive per token. Use when the answer quality is load-bearing.

**Mid-tier.** Solid general-purpose models — second-string from major providers, or top-tier from a tier below. Good at execution, summarization, structured tasks, most code edits. Roughly 5-15x cheaper than frontier.

**Small / local.** Local models on your hardware, or the smallest hosted offerings. Fast, cheap, sometimes free. Limited reasoning depth but adequate for narrow tasks: classification, extraction, formatting, simple lookups. See [05-local-llms](../05-local-llms/).

The tier names don't matter. The decision matters: which tier is right for this call.

## What each tier is good for

**Frontier:**
- Architecture and design decisions
- Writing specs from ambiguous requirements
- Multi-file refactors with cross-cutting impact
- Code review where missed issues are expensive
- The "judge" role in plan/execute/judge when stakes are high
- Novel problems where you don't know the right approach

**Mid-tier:**
- Implementing a clear spec
- Following a written plan
- Single-file edits with bounded scope
- Test generation against existing code
- Documentation and summary writing
- The "execute" role in most agentic workflows

**Small / local:**
- Classification and triage (is this issue a bug, feature, or question?)
- Extraction (pull the error message out of this log)
- Formatting (turn this into JSON)
- Routing (which agent should handle this?)
- Simple Q&A against well-defined corpora
- High-volume batch tasks where per-call cost dominates

## Tiering in practice

A plan/execute/judge workflow with naive routing uses one model for all three roles. A tiered version routes by need:

- **Plan** → frontier (planning is reasoning-heavy; mistakes here cost the most downstream)
- **Execute** → mid-tier (the plan tells it what to do; it just needs to do it correctly)
- **Judge** → frontier OR a different mid-tier than the executor (different perspective + sufficient capability to spot errors)

This single change typically cuts total cost 3-5x with no measurable quality loss.

## When NOT to tier down

- **The model can't do the task.** Switching to mid-tier and getting wrong code is not cheaper. It's free, and it's also useless.
- **You don't have evals.** Tier changes should be validated. Without an eval suite you can't tell if quality dropped, and you'll only discover it when something ships broken.
- **Trust calibration is mid-flight.** If your team is just learning to direct agents, fix the model variable first. Vary it later.

## Routing logic

The simplest router is a hard-coded rule per role: planner uses model A, executor uses model B. This is fine and you should start here.

More sophisticated routers branch on task signal:
- Length / complexity heuristics ("if the task description is over 200 words OR mentions architecture, escalate to frontier")
- Confidence flags from the calling agent ("planner says this is high-risk → judge with frontier")
- Cost budgets ("this session has burned $5; route remaining work to mid-tier unless escalated")

Build the simple version first. Add adaptive routing only when the simple version's cost or quality is the actual bottleneck.

## Provider mixing

Tiering across providers (frontier from one, mid-tier from another) is sometimes worth it for cost, but adds operational complexity:

- Different providers have different rate limits, auth, billing, and downtime
- Latency can vary by 2-3x between providers for similar tier
- Caching mostly doesn't cross providers — a switch breaks your cached prefix

Default to single-provider tiering until you have a real reason to mix. Multi-provider review (see [03-multi-agent-workflows/multi-llm-review.md](../03-multi-agent-workflows/multi-llm-review.md)) is a different pattern from tiering and is worth the complexity for different reasons.

## Common mistakes

- **All-frontier defaults.** "We use the best model" sounds responsible. It's usually 3-10x more expensive than necessary.
- **All-cheap defaults.** Tier-2 results that are subtly wrong erode trust faster than they save money.
- **Tiering without measurement.** If you can't tell whether quality changed, you don't know whether the tiering worked.
- **Tiering by author preference.** "I like model X" is not a routing rule. Match tier to task, not to taste.

## The decision

For a given call, ask three questions:

1. **What's the cost of being wrong here?** High → frontier. Low → mid-tier or smaller.
2. **Is the task well-specified?** Yes → mid-tier can usually execute. No → frontier helps disambiguate.
3. **Is this in the loop's hot path?** Yes (called every step) → push down a tier if quality holds. No (called once per session) → tier matters less; just pick the obvious one.

That's enough for 90% of routing decisions. The other 10% benefit from evals.
