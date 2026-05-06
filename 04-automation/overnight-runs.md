# Overnight Runs

Patterns for letting agents work while you sleep. Done well, it's the highest-leverage form of automation. Done poorly, it's an expensive way to wake up to a mess.

## What works overnight

- **Batch processing of small, well-specified tasks.** "Apply this refactor across these 30 files." "Add tests for these 20 untested functions." "Update imports across the codebase."
- **Long-running explorations.** "Evaluate these three approaches to X, run benchmarks, write up findings."
- **Iterative loops with clear convergence criteria.** "Fix lint errors until none remain." "Resolve all type errors in this directory."
- **Periodic maintenance.** "Update dependencies, run tests, open PRs for safe upgrades."

## What doesn't work overnight

- **Tasks that need clarification mid-run.** If the agent is going to get stuck on an ambiguity, you'll come back to a wasted run.
- **High-stakes one-off changes.** The cost of a bad overnight run on critical infrastructure is enormous.
- **Anything time-sensitive.** Overnight is slow by definition. Don't put fires there.
- **Open-ended creative work.** "Make the codebase better" produces unhinged output without bounds.

## The overnight checklist

Before kicking off:

- [ ] **Spec is clear.** A spec you can defend if the agent does exactly what it asked.
- [ ] **Bounded scope.** A definable end state. "Until tests pass" or "for these 30 files" not "forever."
- [ ] **Sandboxed environment.** No production credentials, no destructive access to shared systems.
- [ ] **Budget cap.** Hard limit on token spend. ~$50 unless you really mean it.
- [ ] **Step cap.** Hard limit on agent iterations. "Stop after 100 steps" prevents pathological loops.
- [ ] **Logging.** Output goes to a file you can read in the morning. Don't lose the trail.
- [ ] **Snapshot.** Git commit the starting state so you can compare diffs cleanly.

If any of these is missing, don't run overnight. Run interactively and address the gap.

## Loop patterns

**Single-shot.** Dispatch one task, agent runs to completion or failure, exits.
- Simplest. Best for well-defined batch work.

**Iterate-until-done.** Loop runs the agent against a goal repeatedly, terminating when goal is met or max iterations hit.
```bash
for i in {1..10}; do
  agent-cli --task "fix all lint errors" || break
  if make lint; then break; fi
done
```
- Good when the goal is checkable but not single-step achievable.

**Queue-driven.** A queue of tasks; the agent pulls one, completes it, pulls the next.
- Scales to many small tasks. Set up the queue carefully — the agent can't tell good queue items from bad.

**Watch-and-respond.** The agent watches for events (PR comments, CI failures) and responds.
- Useful for "babysit-prs" style flows. Make the response surface narrow — don't give it open-ended capability.

## What "Ralph" looks like

The simplest overnight loop:

```bash
#!/bin/bash
set -e

LOG_DIR="logs/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$LOG_DIR"

for i in $(seq 1 50); do
  echo "=== Iteration $i ===" | tee -a "$LOG_DIR/run.log"
  
  # Check if we're done
  if make test &>/dev/null && make lint &>/dev/null; then
    echo "Convergence reached at iteration $i" | tee -a "$LOG_DIR/run.log"
    break
  fi
  
  # Run the agent
  agent-cli --task specs/current-task.md \
    --max-tokens 50000 \
    --output "$LOG_DIR/iter-$i.md" \
    2>&1 | tee -a "$LOG_DIR/run.log"
  
  # Commit progress
  git add -A
  git commit -m "agent: iteration $i" || true
done
```

This is intentionally simple. It loops, checks convergence, runs the agent, commits, repeats. Variations add error handling, smarter convergence checks, escape hatches.

## Convergence criteria

The most common reason overnight runs go bad: no clear stop condition. Patterns that work:

- **Tests pass.** Run the tests; if they pass, done.
- **Lint clean.** Run the linter; if zero issues, done.
- **Goal-state file exists.** The agent writes a `DONE` marker when it believes itself complete.
- **No diff produced.** If the agent's last run made no changes, it's stuck or finished. Either way, stop.
- **Manual signal.** The agent watches for a file (`STOP`) and exits if present.

Combine multiple criteria. "Stop if tests pass AND lint clean AND no diff in last iteration."

## Recovery from failure

The agent will fail. Plan for it.

- **Each iteration should be a clean checkpoint.** Commit (or branch) after each. If the next iteration corrupts the tree, revert to the last good state.
- **Errors should be loud and final.** A failed step that gets silently retried 50 times is the recipe for token spend with no progress.
- **Catastrophic failures should kill the loop.** If the build is fundamentally broken, the agent shouldn't keep iterating against it.

## Reading the morning report

When you check in, you want to scan in 5 minutes. Structure the output so this is possible:

- A summary at the top: convergence reached / max iterations hit / failed at iteration N
- A list of commits made
- A diff against the starting commit
- The full log linked but not in your face

If the run converged: review the diff, run tests yourself, decide whether to merge. Often this is 10 minutes of work for hours of agent time.

If the run failed or hit max iterations: read the last few iterations' logs to understand why. Common patterns: ambiguous spec, missing tool, wrong assumption that needed correction.

## Cost discipline

Overnight runs are how teams discover that automation has unbounded cost.

Default budget caps:
- **Per task:** $5-50 depending on size.
- **Per night:** $100 unless you've explicitly approved more.
- **Per loop iteration:** Some maximum that ensures the loop can't run away.

Actual spend is often much lower. The cap is a safety net, not a target.

Monitor in real time if your budget allows. A simple webhook from the model provider's billing API → Slack notification → you wake up to "your overnight run hit $200, paused." Better than waking up to a $2000 bill.

## When not to bother

For tasks that take 30 minutes interactively, overnight is rarely worth the setup. The break-even is when:
- The task would take you many hours interactively
- The agent can plausibly make progress without you
- You can verify the result faster than you could have done the work

Below that threshold, just do it interactively. Above that threshold, set up properly and let it cook.

## See also

- [06-token-efficiency/cost-estimation.md](../06-token-efficiency/cost-estimation.md) — the budget caps that protect against runaway loops
- [04-automation/sandbox-environments.md](./sandbox-environments.md) — where overnight runs should actually live
- [07-tools-and-mcp/observability.md](../07-tools-and-mcp/observability.md) — what to capture so the morning review is fast
