# Examples

The patterns in the rest of this repo describe artifacts in the abstract. This folder shows what they actually look like as files you'd hand to an agent.

These are illustrative, not copy-paste-ready — your team's conventions will differ. Treat them as starting points.

## What's here

| File | What it demonstrates | Referenced in |
|------|----------------------|---------------|
| [example-spec.md](./example-spec.md) | A real-shape spec for a single feature task | [03-multi-agent-workflows/spec-driven.md](../03-multi-agent-workflows/spec-driven.md) |
| [example-CLAUDE.md](./example-CLAUDE.md) | A project-level context file for an agent | [02-platform/agentic-os-spine.md](../02-platform/agentic-os-spine.md) |
| [example-plan-execute-judge.md](./example-plan-execute-judge.md) | Three prompts for a plan/execute/judge workflow | [03-multi-agent-workflows/plan-execute-judge.md](../03-multi-agent-workflows/plan-execute-judge.md) |

## How to read these

Read the corresponding pattern page first; the example is the concrete instance of the abstract idea. Then look at the example and notice:

- What's *specified* and what's left implicit
- Where the file boundaries fall
- What the agent is told vs. what's left for it to figure out
- How short or long the file is for the work it describes

Most teams' first attempts at these artifacts are too long, too vague, and too generic. The shape that works is shorter and more specific than people expect.
