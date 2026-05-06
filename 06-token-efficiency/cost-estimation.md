# Cost Estimation

Most teams running agentic workflows have a vague sense that "AI costs money" and a precise sense of the bill arriving at the end of the month. Between those two states is the work of actually understanding where the money goes and what it should cost.

This page is the bridge.

## The unit economics

Three numbers drive every bill:

1. **Input token rate.** Cost per million input tokens. Frontier models: $3-15/M. Mid-tier: $0.30-3/M. Small/local: ~$0 if self-hosted.
2. **Output token rate.** Typically 3-5x the input rate. Frontier: $15-75/M output.
3. **Cache discount.** Cached input tokens cost 10-30% of normal input. Often the biggest practical lever.

These rates change. The shape of the math doesn't.

## How to estimate a workflow

Take a typical workflow and decompose:

```
total_cost = sum_over_calls(
    (input_tokens * input_rate) +
    (output_tokens * output_rate)
)
```

For a single plan/execute/judge cycle on a moderate task, rough numbers:

- **Plan call:** 20K input (system + tools + context + task), 2K output (the plan)
- **Execute calls:** ~10 calls × 30K input each (growing with context), 1K output each (tool calls + reasoning)
- **Judge call:** 50K input (full context + diff), 1K output (verdict)

Without caching, frontier model:
- Plan: 20K × $3/M + 2K × $15/M = $0.06 + $0.03 = $0.09
- Execute: 10 × (30K × $3/M + 1K × $15/M) = $1.05
- Judge: 50K × $3/M + 1K × $15/M = $0.165
- **Total: ~$1.30 per task**

With caching (assume 70% cache hit on input):
- Effective input rate ≈ $1.20/M instead of $3/M
- Total drops to ~$0.55 per task

With model tiering (mid-tier execute):
- Execute calls drop from $1.05 to ~$0.20
- **Total: ~$0.40 per task**

A 3x reduction without changing the workflow, just by applying caching and tiering correctly.

## Multiplying out to the team

A team of 10 engineers, each running 20 agentic tasks per day, 20 working days per month:

- 10 × 20 × 20 = 4,000 tasks per month
- At $1.30/task (naive): $5,200/month
- At $0.40/task (tiered + cached): $1,600/month

These numbers are illustrative — your tasks will be larger or smaller. But the *shape* holds: a team-wide AI bill is the per-task cost times the task volume, and both factors are tunable.

## What "expensive" actually looks like

A few patterns produce surprise bills:

- **Long-running unattended loops.** An agent in a retry loop with no termination condition can burn through $100 in an hour.
- **Aggressive parallel agent dispatch.** Running 10 parallel agents on a 30-minute task means 10x cost over the same 30 minutes.
- **Re-reading large files every step.** A 50K-token file read on every loop step in a 30-step loop is 1.5M tokens of avoidable cost.
- **Large context dumps.** Pasting a full log file or full test output adds 50-200K tokens per call. At scale this dominates.
- **Frontier model on trivial tasks.** Using a $15/M model to format JSON is a 50x markup over what it should cost.

If your bill surprised you, one of these is almost certainly the cause.

## What to monitor

Three metrics, in order of importance:

**1. Cost per task.** Tag each agent run with a task identifier and sum the cost. Distribution matters — average can hide a long tail of expensive tasks.

**2. Tokens per call.** Input and output, separately. Watch for inputs that grow over time (sign of bad caching or no compaction).

**3. Cache hit rate.** Should be 60-90% on cache-friendly workflows. If it's 0% or fluctuating, your prompt structure is breaking the cache.

Add cost alerts at meaningful thresholds: per-developer daily, per-team monthly, and a hard ceiling that kills any session. The hard ceiling is what protects you from the runaway loop.

## Budgets per agent run

Set a hard token or dollar budget per agent run:

- Soft warning at 50% (log a metric)
- Hard stop at 100% (kill the run, surface a clear error)
- Per-task budgets sized to the task class (a one-line edit shouldn't cost more than $0.10; a multi-file refactor might budget $5)

This is the single most important guardrail for autonomous workflows. Without it, one bad loop can outspend a team for a week.

## The provider math

Different providers price differently:

- Some charge separately for prompt caching reads vs writes
- Some have per-token pricing that varies by region
- Some bundle features (search, vision, tool use) with different rates
- Some have volume discounts at high commit levels

The actual rates are documented in each provider's pricing page. Build your estimator off those rates, not memory. Rates change.

## Forecasting

For a six-month forecast:

1. Estimate tasks per developer per day (talk to the team — likely 10-50, varies by maturity)
2. Estimate average cost per task (start with $0.50; refine after measuring)
3. Multiply by team size, working days, growth rate

Be skeptical of teams claiming dramatic numbers in either direction. "We don't use much AI" usually means "we haven't measured." "AI costs us a fortune" usually means "we don't use caching or tiering."

## What to ask vendors

If your team is buying agentic tooling from a vendor (an agent platform, an IDE, a CI integration), ask:

- **What's our usage rate at current activity?** Per-developer, per-day, per-task.
- **What's the cost shape under 2x or 5x growth?** Linear, sublinear, with a step function?
- **Where does caching apply?** Some platforms wrap providers in ways that defeat caching.
- **What controls do we have over model selection?** A platform that locks you to frontier-only is expensive at scale.
- **What's the observability story?** Can you see per-task costs, cache hit rates, model-level breakdowns?

If the vendor can't answer these, you don't know what you're spending or why.

## The takeaway

Cost is controllable. Most of the cost reduction available isn't from picking different vendors or negotiating rates — it's from prompt structure, model routing, and context discipline applied within whatever provider you're using. A team that applies the patterns in this section can typically cut their AI bill by 5-10x with no quality regression. A team that doesn't will pay the full price.
