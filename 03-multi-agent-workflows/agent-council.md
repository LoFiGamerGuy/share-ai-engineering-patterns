# Agent Council

A pattern for *decisions*, not implementation. Multiple agents with different perspectives debate a question. You read the debate and decide.

## When to use it

- Architectural choices ("should this be a library or a service?")
- Tradeoff questions ("performance vs. simplicity here")
- Stuck problems ("we've tried X and Y, what else?")
- Pre-mortems ("what could go wrong with this plan?")
- Spec validation ("is this spec actually implementable?")

Don't use it for execution. The council deliberates; it doesn't write code. Once a decision is made, that's a separate plan/execute/judge run.

## The shape

```
[ Question ]
     |
     v
[ Persona A ] -> argument A
[ Persona B ] -> argument B  -> [ Synthesis ] -> [ Decision ]
[ Persona C ] -> argument C
```

Three to five personas is the sweet spot. Two devolves into a flat agreement-or-disagreement. Six or more becomes noise.

Each persona reads the question, optionally reads each other's arguments, and produces its position. A synthesis step (sometimes another agent, sometimes you) reconciles.

## Picking personas

The personas should disagree. If they all agree, you've designed the council badly.

Useful personas (mix and match):

- **The pragmatist.** "What's the smallest change that solves the actual problem?"
- **The architect.** "What's the cleanest design? What are the long-term consequences?"
- **The security skeptic.** "What does an adversary do here? What can be exploited?"
- **The operator.** "How does this run in production? Who pages at 3am?"
- **The user advocate.** "How does this affect the people using the product?"
- **The cost watcher.** "What does this cost to build, run, and maintain?"
- **The historian.** "What have we tried before that's similar? What did we learn?"
- **The contrarian.** Whatever the consensus seems to be, argues against it.

Pick 3-5 that are relevant to the question. A purely backend perf question doesn't need the user advocate. A UX question doesn't need the operator. Match the council to the decision.

## How to run it

**Manually.** Open a chat with the agent. "Play three personas: pragmatist, architect, security skeptic. Each gives their take on this question: <question>. Don't reconcile yet."

The agent produces three positions. Read them. Either decide, or ask a follow-up question, or run a synthesis pass: "Now reconcile these views. Recommend one path."

**With sub-agents.** Some harnesses let you dispatch sub-agents in parallel. Each sub-agent takes a persona, fills its context with relevant project info, produces its argument independently. Then a synthesis agent reads them all.

The parallel version produces stronger arguments because each persona's context is undiluted. The single-session version is faster.

## A worked example

Question: "We have a slow database query. The team is split on whether to add a cache or rewrite the query. Which?"

**Pragmatist:** "The query is slow once per page load and the data is stable for hours. Cache. 30 minutes of work, immediate fix, can revisit later."

**Architect:** "Caching covers up a structural issue. The query is slow because we're doing a four-table join in a hot path. Rewrite the query to use a denormalized read view. Fixes the cause, not the symptom."

**Operator:** "Cache invalidation will bite us. Last time we cached this kind of data we had three incidents from staleness. Rewrite if it's tractable."

**Cost watcher:** "Rewrite is two days, cache is a half day. But cache adds Redis to the dependency surface, which we don't currently have. Cache + new infrastructure = two days too. They're roughly equal cost."

**Synthesis:** "All three non-pragmatist voices push toward the rewrite. The pragmatist's case rests on speed-to-fix, but the cost analysis equalizes that. Recommend: rewrite. Time-box at three days; if tractable, ship it. If not, fall back to cache with a clear ticket to revisit."

You read this and decide. Maybe the synthesis is right. Maybe you have context none of the personas had ("we're shipping next week, can't afford a rewrite") and you override. The council's job is to surface the considerations, not to make the call.

## Avoiding common failures

**Same model, same answer.** If you're running all personas on the same model with similar prompts, you'll get correlated arguments. Mitigations: use *different* models for different personas, or use prompts that explicitly emphasize the persona's stance ("you are skeptical of cleverness; default to the simplest solution").

**Agreement collapse.** Sometimes the personas just agree, even when set up to disagree. This usually means the question is genuinely lopsided — one answer is clearly right. That's fine; trust the result.

**Persona slippage.** The architect drifts into pragmatic territory. Counter with persona descriptions that include what the persona explicitly *won't* consider. ("The architect doesn't worry about ship date; that's the pragmatist's job.")

**Synthesis hides disagreement.** A synthesis that papers over real splits is worse than no synthesis. The synthesis should explicitly note "X and Y disagree on Z; my recommendation is..." rather than smoothly resolving.

## When the council says "I don't know"

A useful output. If the council can't reach a recommendation because the question is under-specified or genuinely depends on information you haven't provided, that's the answer: refine the question, gather data, then come back.

A council that gives a confident answer when the inputs are insufficient is a worse council than one that flags the gap.

## Council vs. plan/execute/judge

These are not competitors; they handle different things.

- Council: "What should we do?"
- Plan/execute/judge: "Now do it."

Run a council to decide; run plan/execute/judge to implement.

A typical sequence: council outputs a recommendation → you turn it into a spec → plan/execute/judge implements the spec.
