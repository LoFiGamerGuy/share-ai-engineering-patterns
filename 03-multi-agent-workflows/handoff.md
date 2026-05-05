# Handoff

How work moves between sessions, between agents, or between agent and human without losing the thread.

Handoff is the part where multi-session and multi-agent setups break. Get it right and long-running work compounds. Get it wrong and every pickup-from-yesterday is a reboot.

## What handoff means

A handoff is the transfer of in-flight work from one entity to another:
- Agent → human (you're stopping the run; want to resume tomorrow)
- Human → agent (you've decided what to do; want to dispatch implementation)
- Agent → agent (planner hands to executor; executor hands to judge)
- Session → session (the agent crashed or hit a context limit; you start fresh)

In every case, what gets transferred is **state**. State that lives in conversation (volatile) is lost on handoff. State that lives in files (durable) survives.

## The core principle

**If it isn't in a file, it doesn't survive.**

Conversation history is volatile. Reasoning that happened in the chat doesn't carry forward. The next session won't have it. The next agent won't have it. The next reviewer won't have it.

To make handoffs clean: write things down. Specifically:

- **The current spec.** What we're trying to do.
- **Progress notes.** What's been done, what's next.
- **Decisions made.** With reasoning, so they're not re-litigated.
- **Open questions.** With proposed answers if you have them.
- **Known broken state.** If the build is failing, document why and what's expected to fix it.

## A handoff document

For longer running work, keep a `HANDOFF.md` (or similar) in the repo or task directory. Contents:

```markdown
# Handoff: <task name>

## Status
<one-line summary: "in progress, planning phase" / "executing step 3 of 5" / "blocked on X">

## What's done
- <bullet>
- <bullet>

## What's next
1. <next concrete step>
2. <step after that>

## Decisions made
- <decision>: <reason>
- <decision>: <reason>

## Open questions
- <question> — proposed answer: <answer>

## How to pick this up
<concrete instructions: which spec to read, which command to run, where the agent left off>
```

This document is read at the start of every resumed session. It's updated at the end. If the work spans days, it's the artifact that keeps continuity.

## Agent → agent handoff (same task)

In plan/execute/judge, the planner hands to the executor hands to the judge. The handoff artifact is the **plan** (or the spec, plus the plan).

For this to work cleanly:
- The plan must be standalone. Reading the plan should be enough to execute. If the executor needs to ask the planner clarifying questions, the plan was incomplete.
- The plan must be in a file, not a chat message. Files survive handoff; chat doesn't.
- The plan should reference the spec, not duplicate it. The executor reads both.

## Agent → human handoff (pause)

You're stopping for the day. The agent was halfway through. To pick up tomorrow:

1. Have the agent write a brief status update to `HANDOFF.md` (or append to the spec).
2. Note the last action and the next intended action.
3. Note any state in the working tree (uncommitted changes, partial implementation).
4. Optionally, commit-as-WIP with a clear message so `git log` reflects state.

Tomorrow, you read the handoff doc and the recent diff before starting a new session. Don't try to "resume the conversation" — start fresh, oriented by the handoff.

## Human → agent handoff (dispatch)

You've decided what should happen; you want the agent to do it.

The handoff artifact is the **spec**. Specifically, a spec that's complete enough that the agent doesn't need to ask follow-up questions for clarification.

Common failure: the spec sounds clear to you because you have all the context. The agent doesn't have your context. To check, read the spec as if you were a stranger to the project. Anything ambiguous? Add detail.

## Session → session handoff (context limit)

Long-running tasks hit context limits. You can't keep one session going forever — eventually it gets too noisy and the model degrades.

The pattern:
1. Have the agent summarize state to a file before context fills up.
2. Start a new session.
3. The new session reads the summary and continues.

Some harnesses do this automatically (compacting the conversation periodically). Others require you to drive it. Either way, the file is the medium.

## Across-tool handoff

You might do exploration in Claude Code, then implementation in Cursor, then review with a custom agent. The work moves across harnesses.

For this to work, the handoff artifacts must be harness-agnostic:
- Plain markdown for specs and handoff docs
- Plain text in `git log` for what was done
- File-based context (`CLAUDE.md`/`AGENTS.md`) that any reader can use

Don't lock state inside a specific harness. Tool-specific state is a handoff hazard.

## What to avoid

- **"Just continue from where you left off."** No agent can do this reliably across sessions. Always re-orient with a written summary.
- **Long context dumps in the new session.** If you find yourself pasting paragraphs of explanation at the start of every session, the missing context belongs in `CLAUDE.md` or a handoff doc.
- **Implicit state in the working tree.** Half-applied changes that "you'll remember tomorrow" — you won't. Either commit-as-WIP with a clear message or revert and re-do.
- **Multiple half-finished tasks in one branch.** One branch per task. Easier to hand off, easier to cancel, easier to ship.

## Handoff hygiene as a habit

The teams that operate well across long-running work treat handoff documentation as part of the work, not as overhead. The pattern:

- End of every session: 5 minutes updating the handoff doc.
- Start of every session: 2 minutes reading it.
- Net cost: small.
- Net benefit: every session picks up cleanly; nothing is "in someone's head."

Treat handoff like commit messages. They feel like overhead until you've been bitten a few times by their absence.
