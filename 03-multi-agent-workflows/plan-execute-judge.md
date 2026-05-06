# Plan / Execute / Judge

The workhorse pattern. Three roles, three agent runs (or three agent passes), one task.

## The pattern

1. **Planner.** Reads the task, produces a plan. The plan is a written artifact: a list of changes, a sequence of steps, files to touch, tests to add.
2. **Executor.** Reads the plan, makes the changes. Doesn't second-guess the plan; if the plan is wrong, that surfaces as a failed test or judge rejection.
3. **Judge.** Reads the original task and the executor's diff. Critiques the diff against the task. Approves or sends back with notes.

If the judge rejects, you either loop (back to executor with the notes) or escalate (back to planner if the plan was the problem).

## Why this works

The planner and executor are doing different cognitive work. Planning is high-level reasoning: what to change, in what order, what to be careful of. Execution is mechanical: read the file, write the change, run the test.

A single agent doing both tends to either:
- Plan poorly because it's eager to start writing code
- Execute poorly because it's still reasoning about the plan

Splitting them gives each role focused attention.

The judge is the critical addition. The judge reads the diff fresh, without having gone through the work of producing it. This is roughly equivalent to a code reviewer who didn't write the code — they catch things the author won't.

## Concrete example

Task: "Add a rate limit to the public API endpoints, 100 req/min per IP, return 429 with a Retry-After header."

**Planner output:**
```
Plan: API rate limiting

1. Add a rate-limit middleware in src/middleware/rate-limit.ts
   - Use a token bucket per IP
   - 100 tokens / 60s
   - Return 429 with Retry-After when exhausted
2. Wire the middleware into the public routes only
   - src/api/public/* gets the middleware
   - src/api/internal/* does not
3. Add tests in src/middleware/rate-limit.test.ts
   - Allows 100 requests in a window
   - Rejects the 101st with 429
   - Retry-After is approximately seconds-until-reset
4. Update CLAUDE.md with the rate-limit conventions

Files to touch:
- src/middleware/rate-limit.ts (new)
- src/middleware/rate-limit.test.ts (new)
- src/api/public/index.ts (wire middleware)
- CLAUDE.md (document)

Risks:
- In-memory storage won't work across multiple servers; flag this for follow-up
- Need a clock abstraction for testability
```

**Executor:** reads the plan, writes the files, runs `make test`, confirms green.

**Judge:** reads the spec ("rate limit on public API, 100/min/IP, 429 + Retry-After") and the diff. Notices that:
- The middleware was added but the test for "internal routes are NOT rate-limited" is missing — adversarial test
- The Retry-After header is in seconds (correct) but the test asserts approximate (good)
- In-memory storage is flagged in code comments (good)

Sends back: "Add a test that verifies internal routes are unaffected by the rate limit."

Executor adds the test. Judge re-reviews. Approves.

## How to implement it

You have three options for how the three roles run.

**Option A: Three separate sessions.**
Manually run a planner agent, save its output as a spec file, run an executor agent against the spec, run a judge agent against the diff and spec. Highest fidelity (each agent has clean context) but most manual.

**Option B: One session, three turns.**
Same agent plays all three roles in sequence within a single session. Simpler to run; the agent has to consciously context-switch. Works for smaller tasks; degrades for larger ones because the executor's work pollutes the judge's perspective.

**Option C: Orchestrated.**
A harness or script dispatches sub-agents for each role automatically. The planner runs as one sub-agent, hands its output to an executor sub-agent, then to a judge sub-agent. Highest leverage when you do this often; requires harness support.

Many modern harnesses support Option C natively (e.g., spawning sub-agents with their own context windows). When the harness supports it, use it.

## Picking models for each role

You don't have to use the same model for all three.

- **Planner.** Benefits from the smartest model. Planning is the most cognitively demanding step.
- **Executor.** Mid-tier is often fine. The plan is doing the heavy reasoning; the executor follows instructions.
- **Judge.** Benefits from a *different* model than the executor. Different training data, different blind spots. A judge that's the same model as the executor will share its biases.

A common cost-effective setup: frontier model (planner + judge), mid-tier model (executor). The mid-tier model can be from the same provider or a different one.

## What the judge looks for

The judge isn't just checking "does the code work." It's checking the diff against the task description. Useful judge prompts include:

- Does the diff address everything the task asked for?
- Does it do anything the task didn't ask for? (Unwanted scope creep)
- Are there missing tests?
- Are there obvious correctness issues?
- Are there violations of project conventions (read `CLAUDE.md`)?
- Is there anything that looks correct but isn't?

The judge should be told to be specific and adversarial — vague approval is useless.

## When to loop, when to escalate

If the judge has small notes (missing test, naming nit, missed edge case): loop back to the executor with the notes.

If the judge identifies a structural problem (wrong approach, missed major requirement): escalate back to the planner. The plan was wrong; trying to patch the execution will compound the issue.

Set a max loop count (3-5). If you're hitting it, the task is probably under-specified.

## What this pattern doesn't catch

- **Plan errors that look right.** If the plan is plausible-but-wrong, the judge may approve a correct execution of the wrong plan. Counter with multi-LLM review (different models judge differently) and tests that exercise actual behavior.
- **Untested code paths.** If the plan didn't ask for a test, neither the executor nor the judge will demand one. Counter with test coverage requirements in `CLAUDE.md`.
- **Subtle bugs the model can't see.** If both planner and judge have the same blind spot, the bug ships. Counter with multi-LLM review and human review for high-stakes changes.

## The minimum useful version

If you want to start with the simplest possible plan/execute/judge:

1. Write a one-paragraph spec by hand
2. Ask the agent: "Plan this, then execute, then critique your own diff against the spec"
3. Read the critique
4. If the critique flags something real, instruct the fix

That's it. Adding actual sub-agents and orchestration improves quality, but even the same-agent-three-passes version produces noticeably better results than direct execution.

## See also

- [examples/example-plan-execute-judge.md](../examples/example-plan-execute-judge.md) — three example prompts, one per role
- [03-multi-agent-workflows/spec-driven.md](./spec-driven.md) — specs are the contract these roles communicate through
- [06-token-efficiency/model-tiering.md](../06-token-efficiency/model-tiering.md) — route the executor to a cheaper model than the planner/judge
