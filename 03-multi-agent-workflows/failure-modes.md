# Failure Modes

What goes wrong with multi-agent setups, and how to recognize it before it costs you.

## Agreement collapse

**Symptom.** Multiple agents always agree. Reviews are uniformly positive. Council debates resolve too quickly.

**Cause.** Agents with similar models, similar prompts, or similar context will produce similar opinions. The "second opinion" isn't independent.

**Fix.** Use models from different providers. Frame prompts with explicit, divergent stances. If a council always agrees, replace one persona with one explicitly contrarian.

## Confident hallucination

**Symptom.** An agent (often a judge or reviewer) flags an issue that doesn't actually exist in the code. Cites a function that isn't there, or a behavior that isn't real.

**Cause.** Models pattern-match to plausible issues. Without grounding, "plausible" beats "true."

**Fix.** Always ground reviews in the actual code. Pass the diff with full context, not summaries. When an agent flags an issue, verify before acting on it. The cost of verification is low; the cost of "fixing" a non-existent problem is high.

## Context contamination

**Symptom.** Later turns in a session degrade compared to earlier ones. The agent confuses things, repeats itself, makes contradictory choices.

**Cause.** As context grows, signal-to-noise drops. The agent is reasoning over a conversation that includes earlier wrong turns, abandoned plans, and irrelevant tool results.

**Fix.** Start fresh sessions for distinct tasks. Use compaction when the harness supports it. Move durable state out of conversation into files. Recognize when a session has gone bad and abandon it rather than fighting through.

## Plan-execute mismatch

**Symptom.** The plan looks good, the execution looks good, but the result doesn't match the original task.

**Cause.** The plan misinterpreted the task; the executor faithfully executed the wrong plan. Each step looked locally correct.

**Fix.** Always include the original task (the spec) in the judge's context, not just the plan and diff. The judge compares against the task, not just internal consistency. Multi-LLM review catches this — different models will interpret the spec differently and flag the inconsistency.

## Judge capture

**Symptom.** The judge consistently approves work that turns out to have problems.

**Cause.** Several possibilities. The judge prompt is too lax ("looks good"). The judge is the same model as the executor (shared blind spots). The judge isn't being given enough context to evaluate.

**Fix.** Make judge prompts adversarial — explicitly ask for problems, missing tests, scope creep, convention violations. Use a different model for the judge. Provide the judge with project conventions (`CLAUDE.md`) so it can evaluate against the standard.

## Loop runaway

**Symptom.** Plan → execute → judge rejects → execute again → judge rejects again → ... — the agent thrashes. Costs escalate, no convergence.

**Cause.** The task is under-specified. Each iteration is interpreting the gap differently. Or the judge keeps moving the goalposts.

**Fix.** Set a max iteration count (3-5). When you hit it, stop. The task needs human input — usually a clearer spec or a clearer rejection. Iterating without resolution wastes tokens.

## Scope creep

**Symptom.** The diff includes changes the task didn't ask for. Refactors, "while I was here" cleanups, new tests for unrelated code.

**Cause.** Models are trained to be thorough. Without explicit out-of-scope guidance, "thorough" means "do extra things."

**Fix.** Specs include an explicit "out of scope" section. Judge prompts ask "does the diff do anything the task didn't ask for?" Reject diffs that include unrequested changes; have the agent split them into separate work.

## Spec rot

**Symptom.** The spec was written last week, the agent implements against it, the implementation doesn't fit the codebase anymore because the codebase moved.

**Cause.** Specs are written against a snapshot. The codebase keeps moving. By the time implementation runs, the spec is partially out of date.

**Fix.** Have the agent read recent code first, before implementing. Treat the spec as a draft; if it conflicts with reality, update it. Don't dispatch ancient specs without re-reading them.

## Cost runaway

**Symptom.** The token bill spikes. Multiple agents running at full context size, looping, retrying.

**Cause.** No budget caps. Agents calling agents calling agents. Long-context conversations not being compacted.

**Fix.** Hard budget per task. Hard step limit per agent. Per-loop spending alerts. Cheaper models for roles that don't need frontier (executor, simple reviewers). See [06-token-efficiency](../06-token-efficiency/).

## Trust drift

**Symptom.** The team starts at "review every line." Six months later, no one is reviewing. Bugs ship.

**Cause.** Quality is high enough that careful review feels redundant. Each individual decision to skim is fine. The accumulated drift isn't.

**Fix.** Make trust levels explicit and revisited. "What level are we at for X type of task?" should be a regular conversation. Ratchet down (more review) when something breaks; ratchet up (less review) only when there's evidence the agent has earned it.

## Tool sprawl

**Symptom.** The agent has 50+ tools available. It picks suboptimal ones, gets confused about which to use, hallucinates tools that don't exist.

**Cause.** Every additional MCP server adds tools to the prompt. Without curation, this grows monotonically.

**Fix.** Keep the active tool set small. Per-project MCP configs that only load what's relevant. Periodically prune servers that aren't being used. Description quality matters — tools with good descriptions get picked over ones with bad descriptions.

## Silent skill atrophy

**Symptom.** A team that used to be fast at low-level work now relies on agents for everything. When the agent is wrong, no one catches it because no one has fresh hands-on with that area.

**Cause.** Agents are good enough that humans stop practicing. The deep skill that lets you supervise an agent is the same skill that erodes when you stop doing the work yourself.

**Fix.** Mostly a team practice question, not a tooling one. Some teams designate "no agent days" or specific tasks done manually. Some pair humans through agent-driven work to keep skills warm. The right answer depends on team structure; the failure mode is real either way.

## Patterns for recognizing problems early

- **Read the diffs.** Even when you trust the agent. The first sign of any of these problems is in the diff.
- **Track loop counts.** If iterations on a task are climbing, something's off.
- **Watch the bill.** A doubling of token spend with no corresponding output increase = something's wrong.
- **Notice when reviewers always agree.** Disagreement is healthy; uniformity is suspicious.
- **Pay attention to "well, that's weird" reactions.** When you find yourself surprised by something the agent did, that's a signal. Don't suppress it.
