# Spec-Driven Workflows

The connective tissue of multi-agent work. Every multi-agent pattern degrades without good specs; every multi-agent pattern improves dramatically with them.

## What "spec-driven" means in practice

Instead of giving an agent a task in chat ("hey can you add rate limiting"), you write the task as a markdown file (`specs/2026-04-rate-limit.md`) and point agents at the file. The spec becomes:

- The input to the planner
- The reference for the executor (when uncertain about scope, re-read the spec)
- The benchmark for the judge (does the diff match the spec?)
- The artifact that survives the work — readable months later

Specs replace ephemeral chat with durable text. That single change is what makes most other patterns work.

## Anatomy of a useful spec

```markdown
# <Title — short, descriptive>

## Goal
Why this work exists. One paragraph.

## Background
Context the agent needs but won't infer. Links to related docs, prior decisions, relevant tickets.

## Scope
What's included. Be specific: file paths, modules, behaviors.

## Out of scope
What might look related but isn't. Prevents scope creep.

## Approach
How to do it, if you've decided. If not, "investigate options" is fine, but say so.

## Constraints
Things that must hold. Compatibility, performance, dependencies.

## Acceptance
How we know it's done. Tests, behaviors, metrics.

## Open questions
What's unresolved. Either you decide before starting, or you flag for the agent to investigate first.
```

Length: 30-200 lines for most tasks. Longer = the task is too big.

## Why every section earns its place

**Goal.** Without it, the agent optimizes for the wrong thing. "Add rate limiting" without "to prevent abuse of the public API" leaves the agent free to add it places where it doesn't belong.

**Background.** Saves the agent (and you) from making decisions that contradict prior decisions you've forgotten about.

**Scope.** The agent will draw boundaries somewhere. If you don't tell it where, it'll guess, and the guess will be wrong sometimes.

**Out of scope.** Preempts the most common failure mode — the agent doing more than asked. "Don't refactor unrelated code" prevents 80% of unwanted-scope-creep.

**Approach.** If you have one in mind, share it. If you don't, say "investigate first." Don't make the agent guess whether you have a preference.

**Constraints.** The hidden requirements that aren't obvious from the goal. "Backward compatible with v1 clients" is the kind of thing that should be in the spec, not discovered when v1 clients break.

**Acceptance.** The judge needs this. Without it, "is the work done?" is a vibe. With it, it's a checklist.

**Open questions.** The honest section. The things you don't know, so you can't decide them in advance. The agent investigates and proposes, you confirm.

## Where specs live

A `specs/` directory at the repo root, with files named `YYYY-MM-<short-name>.md`. Sortable, dated, easy to find.

After a spec is implemented, you have options:
- Leave it in `specs/` as historical record
- Move to `specs/done/` if the directory gets noisy
- Delete it (lose institutional memory; not recommended)

Either of the first two is fine. The point is the spec exists and can be referenced ("we did rate limiting per the 2026-04 spec").

## Specs as agent dispatch

A spec-driven workflow turns task assignment into:

```
1. Write spec to specs/<name>.md
2. Run agent with prompt: "Implement specs/<name>.md"
3. Agent reads the spec, reads relevant code, plans, executes, judges
4. Review result against spec
```

You can dispatch the same way to:
- Different agents (Claude, Codex, Cursor) — pick whichever works best for this task
- Sub-agents (the planner, executor, judge each read the same spec)
- Multiple parallel agents (run two implementations from the same spec, compare)

The spec is the universal interface. Anything that reads markdown can be the executor.

## Iteration patterns

**Spec-first iteration.** You iterate on the *spec*, not on the implementation. Spec changes are cheap; implementation changes are expensive. When something feels off, ask: should I adjust the spec, or accept this implementation?

**Spec-as-output.** For exploratory work, the agent's first run is to produce a spec, not to implement. ("Investigate the options for X. Output a spec.md describing the recommended approach.") You review the spec, edit it, then dispatch implementation against it.

This pattern — explore → spec → implement — is much higher quality than "explore + implement in one pass."

## Specs vs. tickets

Specs and tickets aren't the same thing. A ticket is "what." A spec is "how, with constraints."

You might:
- Have a Jira ticket "Add rate limiting"
- Write a spec at `specs/2026-04-rate-limit.md` linked from the ticket
- Implement against the spec

Or:
- Skip the ticket entirely; the spec is the ticket

Both work. The key is that the spec is a different artifact from a chat message — it's structured, referenceable, and writable.

## Specs at different sizes

**Tiny tasks.** Don't bother. "Rename foo to bar across the codebase" doesn't need a spec.

**Small tasks (a few hours of work).** A 20-50 line spec.

**Medium tasks (a day or two).** A 100-200 line spec.

**Large tasks.** Either break into smaller specs, or write a 200-line spec plus a `decisions/` doc that captures the design choices in detail.

If you find yourself writing 500+ lines of spec, the task is too big. Decompose.

## What specs don't replace

- **Tests.** Specs describe intent; tests verify behavior. Both.
- **Documentation.** Specs describe a task; docs describe the system. They might overlap, but a spec written for an agent isn't user-facing docs.
- **Design docs.** A design doc is upstream of a spec. Big architectural decisions get a design doc; specs implement against decisions.

## When specs are wrong

Specs go stale. The codebase moves; the spec was written against the old shape; the implementation against the spec produces nonsense.

Two practices:
- **Read recent code first.** Either you do it before writing the spec, or you instruct the agent to do it before implementing. ("First, read src/api/* to understand the current structure. Then implement against the spec.")
- **Treat the spec as a draft.** When the executor surfaces a contradiction between the spec and reality, that's good. Update the spec, then continue.

A spec that the agent argues with is more valuable than a spec the agent silently ignores.
