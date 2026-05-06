# Token Efficiency Overview

Agentic workflows have a unit cost. At small scale you don't notice. At team scale, the bill can quietly become a meaningful operating expense, and the same workflow that costs $0.30 per task can cost $3.00 per task with no change in output quality — just bad context discipline.

This section covers the patterns that drop cost (and latency) without sacrificing quality.

## The patterns

| Pattern | What it does |
|---------|--------------|
| [Model Tiering](./model-tiering.md) | Route work to the right model. Frontier for hard reasoning, mid-tier for execution, small/local for batch. |
| [Prompt Caching](./prompt-caching.md) | Make stable context cheap. Pin the durable parts; let the variable parts be small. |
| [Context Management](./context-management.md) | Three layers: caching, compaction, retrieval. Keep the working set small. |
| [Cost Estimation](./cost-estimation.md) | How to budget, monitor, and forecast. What "expensive" actually looks like. |

## The mental model

A token is the unit of LLM input and output. You pay per input token (cheaper) and per output token (more expensive). Tasks differ by an order of magnitude in token consumption, and most teams have no idea which tasks are expensive until they read the bill.

Three structural drivers:

1. **Model choice.** Frontier models cost 10-30x more per token than mid-tier models from the same provider, and 50-200x more than small local models. The capability gap is real but smaller than the price gap suggests for many tasks.
2. **Context volume.** Long conversations, large file loads, and verbose tool results all add input tokens to every subsequent turn. A loop that reads 50 files into context pays for those 50 files on every step until they age out.
3. **Loop length.** A single completion is cheap. A 30-step agent loop is 30 completions, each one carrying the accumulated history. Loops that don't terminate are billing problems.

Optimize all three, in that order.

## The rough cost shape

For a typical multi-agent engineering task (spec → plan → execute → judge):

- **Single frontier model, no caching, verbose context:** baseline (call it 1.0x)
- **Single frontier model with prompt caching:** 0.3-0.5x
- **Tiered models (frontier for plan/judge, mid-tier for execution) with caching:** 0.15-0.25x
- **Same with retrieval pruning instead of full file loads:** 0.10-0.15x

A 6-10x cost reduction is achievable with no quality loss if you apply all three patterns. Most teams capture none of it because they default to "the best model on every call with everything in context."

## The latency angle

Cost and latency are correlated but not identical:

- **Cached input is cheap and fast.** A 100K-token cached prefix loads in milliseconds; a 100K-token fresh prefix takes seconds.
- **Smaller models are faster.** A mid-tier model returning in 2 seconds beats a frontier model returning in 12 seconds for many tasks.
- **Parallel agents finish faster than sequential ones, even at the same total cost.** Running plan and research in parallel is a UX win even when it doesn't save money.

If you're optimizing for "feels fast," cache aggressively and route to smaller models where you can. If you're optimizing for "costs less," same answer.

## Where to start

If you're hitting unexpected bills: read [cost-estimation.md](./cost-estimation.md) first. You can't optimize what you can't measure.

If you're trying to make agentic workflows feel snappy: read [prompt-caching.md](./prompt-caching.md). It's the single biggest lever.

If you're scaling to a team and want sustainable economics: read [model-tiering.md](./model-tiering.md). Defaulting to frontier on every call is the most common and most expensive mistake.

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
