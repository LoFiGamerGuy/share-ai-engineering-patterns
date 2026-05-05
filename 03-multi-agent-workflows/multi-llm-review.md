# Multi-LLM Review

Run the same task across multiple LLM providers and compare. Differences highlight blind spots, agreements increase confidence.

This is more expensive than single-provider review but the highest-fidelity check available short of human review.

## Why multiple providers

Models from different providers have different training data, different RLHF emphases, different blind spots. When two independent models agree, that's stronger evidence than one model being confident. When they disagree, the disagreement itself is information — it points at the parts of the work that are ambiguous or risky.

A judge that's the same model as the executor shares blind spots with the executor. A judge from a different provider is a more independent check.

## Where to apply it

Not everywhere. The cost is real (you're paying multiple providers) and the latency is real (you're waiting for multiple completions).

Worth it for:
- **Production-bound changes.** The diff is going to ship. Better to spend $0.50 catching a bug than ship it.
- **Security-sensitive code.** Auth, crypto, payment handling, anything where bugs become incidents.
- **API contract changes.** Public interfaces are expensive to roll back. Get them right.
- **Migrations and one-way changes.** Database schema changes, data migrations, anything irreversible.
- **Large diffs.** Above some threshold (200+ lines? 500+?), the chance of a single-reviewer miss goes up.

Skip for:
- Internal refactors with full test coverage
- Documentation-only changes
- Trivial fixes
- Anything you're going to manually review carefully anyway

## How to set it up

Three patterns by complexity:

**Pattern 1: Manual cross-check.** After a change is made, paste the diff into a different-provider chat and ask for a critique. Read both sets of feedback. This requires no setup; works today.

**Pattern 2: Scripted dual-review.** Wire a script that, on PR open or `make review`, sends the diff to two providers and gathers responses into a single comment. Works for a single team or project; takes a day to build.

**Pattern 3: Reviewer-as-tool.** Build an MCP server that exposes a `review_diff` tool which fans out to multiple providers internally. Any agent can call it. Useful when many agents in many projects need it.

## Crafting the review prompt

The review prompt matters as much as the model choice. A weak prompt gets generic feedback regardless of model.

Good review prompts include:

- **The original task** (or spec). Reviewers need to know what was supposed to happen.
- **The diff or full file before/after.**
- **Project conventions** (a pointer to `CLAUDE.md` or the relevant section).
- **What kinds of issues to look for.** Be specific: "Look for missing error handling on async boundaries, incorrect error propagation, missed null cases."
- **The format you want the response in.** Numbered list of issues, severity-tagged, file:line references.

Generic prompts ("review this diff") get generic responses ("looks good!"). Specific prompts get specific responses.

## Reconciling disagreements

When reviewers disagree, you have signal. Three common patterns:

**One reviewer flags an issue, the other doesn't.** Read the flagged issue carefully. Often the silent reviewer just missed it; the concern is real. Sometimes the flagging reviewer is hallucinating an issue that doesn't apply to your codebase. You decide.

**Both reviewers flag the same issue.** High confidence the issue is real. Fix it.

**Reviewers disagree on the same code.** ("This is fine" vs. "This is broken.") The most interesting case. Read both arguments and decide — possibly with a third reviewer, possibly with a quick test.

Don't try to algorithmically resolve disagreements. The point is to surface them so a human can adjudicate.

## Pairing reviewers

Picking which providers to use:

- **Two strong frontier models from different vendors.** Highest signal. Highest cost.
- **One frontier + one cheap fast model.** Good balance. The fast model catches surface-level issues quickly; the frontier model handles the harder critique.
- **Same vendor, different models.** Cheaper, but reduces the diversity benefit. Useful when you want speed/cost over independence.
- **Frontier model + local model.** Adds a fully independent check at near-zero marginal cost. Local model accuracy is lower but it's still a different perspective.

Avoid: the same model twice. You're not getting independence; you're spending money to get correlated opinions.

## What to feed each reviewer

Two choices:

**Same prompt to both.** Easier. Each reviewer sees the same task and diff. Differences in feedback come from model differences alone.

**Differently framed prompts.** "Reviewer A: focus on correctness. Reviewer B: focus on maintainability." Forces different lenses. More work to set up but can yield higher coverage of the issue space.

For most cases, same prompt is fine. The frontier models cover their bases reasonably well on a generic critique prompt.

## Reading the review

When you get back two reviews:

1. Read both before deciding on any single point.
2. Group their concerns. Issues both flag → high priority. Issues one flags that look real → medium. Issues that look like noise → drop.
3. If either reviewer says "I'm not sure" or asks a clarifying question, read that carefully — uncertainty is real signal.
4. Don't fix everything. Some "issues" are matters of taste and you don't have to address them. The point is informed decision, not blind compliance.

## What multi-LLM review doesn't catch

- **Issues outside the visible diff.** Reviewers don't have access to the rest of the codebase unless you give it to them. A change that's correct in isolation but breaks something three files away may pass review.
- **Domain-specific bugs.** If the model doesn't know your business logic, it can't catch business logic errors. Tests catch these; reviews don't.
- **Performance issues.** Reviewers can sometimes flag "this looks O(n²)" but they can't measure. Profile separately.
- **Subtle race conditions.** Concurrency bugs are hard for any reviewer, model or human. Targeted testing matters more.

Multi-LLM review is one layer. Pair it with tests, type checking, and (for the high-stakes changes) human review. No single layer catches everything.

## Cost calibration

Rough order of magnitude for a 500-line diff with two frontier reviewers: $0.10–$1.00 per review depending on prompt and provider. A team running this on 50 PRs/week is spending $25-250/month on reviews. Compare against the cost of one bug shipping to production.

For most teams, this is one of the highest-ROI uses of token budget.
