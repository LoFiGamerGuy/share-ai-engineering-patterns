# Trust and Specs

Two ideas that have to be paired: **how much you trust an agent's output**, and **how precisely you describe what you want**. They control each other. Get the calibration wrong and either the agent does the wrong thing or you waste time reviewing work you could have trusted.

## The trust spectrum

Possible levels of trust in an agent's output, from least to most:

1. **Suggest only.** The agent proposes; a human applies every change manually.
2. **Diff review.** The agent makes changes; a human reads every diff before commit.
3. **Test-gated review.** The agent makes changes and runs tests; a human reads diffs only if tests pass.
4. **Spec-gated review.** The agent works against a spec; a human reviews against the spec, not the diff.
5. **Autonomous with audit.** The agent commits and ships; a human samples output and audits periodically.
6. **Autonomous.** The agent runs unattended; failures are caught by downstream signals (errors, complaints, monitoring).

Most teams should not pick a single level for everything. Different tasks warrant different trust levels. Refactoring a typo: level 5. Modifying a payment flow: level 2. The skill is matching task to level.

## Why specs matter

A **spec** is a written description of the work that's specific enough for someone else (human or agent) to execute without follow-up questions.

Specs do four things:

1. **Force you to think.** Writing the spec exposes the parts you haven't decided. You'd rather find them now than mid-implementation.
2. **Constrain the agent.** A clear spec reduces the search space the model has to explore. It makes worse implementations less likely.
3. **Enable review.** You review the diff against the spec, not against your in-head model. This is faster and catches more.
4. **Enable handoff.** Specs travel between sessions, between agents, between people.

Without a spec, you're effectively asking the agent to guess at the work. Sometimes it guesses well. Sometimes it doesn't. You won't know which until you read the diff.

## What a good spec looks like

A spec is not a paragraph of prose ("make the login page faster"). A useful spec has:

- **Goal.** What outcome are we after, and why.
- **Scope.** What's in, what's out.
- **Approach.** How to do it. If you don't know, say so explicitly — "investigate options" is a valid spec line.
- **Constraints.** Things that must hold. ("Don't change the public API." "Maintain backward compatibility through 2026.")
- **Acceptance.** How we'll know it's done. Tests pass, manual scenarios work, metrics improve, etc.
- **Out-of-scope.** Things the agent might try to do that we don't want.

Length: 20-200 lines is normal. 500+ usually means the task is too big and should be split.

## Trust ratchets

Trust calibrates over time. The pattern that works:

- **Start tight.** New agent, new model, new task type → diff review.
- **Earn the next level.** After 5-10 successful diff reviews on similar tasks, move to test-gated review.
- **Earn the next.** After confidence builds, move to spec-gated.
- **Don't skip levels.** Going from "I've never let this agent do anything autonomously" to "let it run overnight" is how teams get burned.

Equally important: **ratchet down when something breaks.** If an agent at level 5 produces a serious bug, drop it back to level 3 for that task type until you understand what went wrong.

## The trust-spec pairing

Here's the rule: **the lower the trust level, the more the diff is the contract. The higher the trust level, the more the spec is the contract.**

At level 2 (diff review), the spec just kicks off the work; review happens against the actual change.

At level 5 (autonomous with audit), there is no diff review. The spec *is* the agreement. If the spec is vague, the agent has license to do whatever it interprets the spec to mean. Vague specs at high trust = bugs in production.

This is why teams that operate at high autonomy invest heavily in spec quality. They've moved their review effort from "looking at code" to "writing precise descriptions."

## Verification, not just inspection

Trust isn't only "did I read the diff." Strong verification includes:

- **Tests.** The agent runs them; you require they pass.
- **Static analysis.** Lint, type check, format check.
- **Property checks.** Invariants that should hold (no secrets in code, no `console.log` in production paths).
- **Multi-agent review.** A second agent critiques the first agent's work. (See [03-multi-agent-workflows](../03-multi-agent-workflows/).)
- **Production canaries.** Ship to a fraction of traffic, watch for regressions.

Trust at level 4+ requires multiple verification layers. "I read the diff carefully" is one layer; it's not enough on its own for fast-moving work.

## When to write specs vs. when to just ask

For tiny tasks (rename a function, fix a typo, add a missing import) — don't bother with a spec. Just ask. The overhead of writing a spec exceeds the value.

For anything multi-step, anything that touches multiple files, anything with constraints — write the spec. Even a bad spec is better than no spec, because the act of writing it forces clarity.

Rough rule of thumb: if the agent will produce more than ~50 lines of changed code, write a spec.

## The hidden cost of skipping specs

Teams that "just chat with the agent" tend to:
- Iterate more (the agent guesses; you correct; repeat)
- Produce more inconsistent code (no specification means each iteration drifts)
- Have less reusable artifacts (the spec is also documentation; without it you have nothing)
- Miss the parallelism opportunity (with a spec, multiple agents can work in parallel; without, only the original chat session can continue)

The 10 minutes you "save" by skipping the spec costs you 30 minutes in iteration and produces a worse result. Specs are not overhead — they're the work.
