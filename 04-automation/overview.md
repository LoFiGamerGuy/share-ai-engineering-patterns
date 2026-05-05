# Automation Overview

Letting agents run unattended — overnight, in CI, in sandboxes, in production loops. This is where the leverage curve gets steep and the failure modes get expensive.

## Why automate

Interactive use of agents is bounded by your attention. You watch each step; you approve each action; you iterate in real time. That's high-quality but it doesn't scale beyond what you can personally attend to.

Automation lifts that ceiling. An agent that runs overnight can chew through a backlog of well-specified tasks. An agent that runs in CI can review every PR. An agent that runs in a loop ("Ralph", "openclaw", "babysit") can keep working on a problem while you do something else.

The catch: every step away from "human watching" is a step toward "agent doing something dumb at scale." Automation amplifies both leverage and mistakes.

## The dimensions

This section organizes automation along three axes:

**[Sandbox environments.](./sandbox-environments.md)** Where the agent runs. The choice of execution environment determines blast radius. A fully isolated VM lets the agent do anything; a production server means every action is potentially live.

**[Overnight runs.](./overnight-runs.md)** When the agent runs unattended. Patterns for dispatching long-running work, recovering from failures, and reviewing the result the next morning.

**[Autonomous control levels.](./autonomous-control-levels.md)** How much the agent decides for itself. From "asks permission before any tool call" to "runs in a tight loop without supervision." The right level depends on the task and the safeguards.

## The trust-blast-radius tradeoff

Two questions for every automation setup:

1. **How much do I trust the agent to do the right thing?** Based on track record for this kind of task.
2. **What's the blast radius if it does the wrong thing?** Based on the environment and tools.

The right configuration is one where these match. High trust + small blast radius: fine, but maybe over-engineered. Low trust + large blast radius: actively dangerous. The diagonal is what you want: trust calibrated to the consequences.

A practical rule: **the higher the autonomy, the smaller the blast radius should be.** Fully autonomous loops should run in disposable sandboxes. Agents with production access should require human approval for every change.

## The "openclaw / Ralph / babysit" pattern

Several names for the same idea: an agent running in a loop that keeps working on a problem while a human checks in periodically.

Variants:
- **Ralph.** A Bash loop that keeps invoking an agent against the same task until convergence.
- **openclaw.** A pattern where you let the agent run with an objective (e.g., "fix all lint errors in this directory") and check in at intervals.
- **babysit-prs.** A loop that watches a PR, responds to review comments, fixes CI failures.

These all live on the autonomy axis between "interactive" and "fully autonomous." The agent runs without your continuous attention; you're checking on it, not driving it.

For these to work safely, you need:
- A clear, bounded objective ("fix lint errors in src/", not "make the codebase better")
- A sandboxed environment (no production credentials)
- A budget cap (max iterations, max tokens)
- A clear stop condition (objective met, max attempts hit, or human intervention)

## What automation is good for

- **Backlog burning.** A queue of small, well-specified tasks. Each one is too small to attend to individually; collectively they're a chore. Dispatch overnight, review in the morning.
- **CI augmentation.** Lint, format, test, type-check — these are existing automation. Add: agent review on every PR, automatic generation of test cases for changed code, automatic doc updates.
- **Continuous improvement loops.** Agents running periodically against codebase metrics — flag dead code, suggest refactors, find unused dependencies. Output is suggestions, not changes.
- **Long-running explorations.** A research-style task ("evaluate three approaches to X, write up findings") can run for hours without supervision.

## What automation is bad for

- **High-stakes one-off work.** Migrations, security changes, anything irreversible. The savings from automation aren't worth the risk.
- **Under-specified work.** If you can't write a clear objective, the agent will pick its own, and it'll be the wrong one.
- **Anything time-critical.** Automated agents are not real-time. If you need it now, do it interactively.

## The morning-after question

The right test for any automation setup: when you check in (next morning, end of week, whatever the cadence), can you tell whether things went well?

Concretely:
- Is there a clear log of what the agent did?
- Are the changes summarized somewhere readable?
- Are failures flagged distinctly from successes?
- Can you scan in 5 minutes whether to dig in or just merge?

If the answer is no, your automation is producing work you can't evaluate, which means you'll either rubber-stamp it or stop using it. Both are failure modes.

## Reading order

Start with [autonomous-control-levels.md](./autonomous-control-levels.md) — the framework for thinking about where on the autonomy spectrum your work belongs.

Then [sandbox-environments.md](./sandbox-environments.md) — how to constrain blast radius.

Then [overnight-runs.md](./overnight-runs.md) — the practical patterns for unattended operation.
