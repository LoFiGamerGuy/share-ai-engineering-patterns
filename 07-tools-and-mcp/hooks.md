# Hooks

Hooks are event handlers in the agent harness. They run automatically on defined events — before a tool call, after a file write, on session start, on completion. The model doesn't choose when they fire. The harness does.

This makes hooks the right tool for guardrails, automation, and consistency enforcement that should not depend on the model remembering to do something.

## Why hooks exist

Every agentic workflow has things that should happen *every time*, regardless of what the model decides. Examples:

- Run the linter after every file edit
- Block any shell command containing `rm -rf`
- Log every file write to an audit trail
- Refuse tool calls that touch files outside the project root
- Inject the latest project README into the system prompt at session start
- Trigger a notification when the agent completes a task

Asking the model to do these reliably is fragile. Hooks make them automatic.

## Common hook events

Different harnesses define different events; the common ones across most:

- **Session start.** Load context, check environment, register skills, validate auth.
- **Before tool call.** Inspect the call. Approve, deny, or modify before it runs.
- **After tool call.** Log it. Update an audit trail. Trigger downstream actions.
- **Before file write / edit.** Validate the change. Format the content. Check for secrets.
- **After file write / edit.** Run the linter. Run the formatter. Trigger tests.
- **On user prompt.** Inspect the message. Inject context. Block on policy.
- **On completion / stop.** Generate a summary. Update status. Notify someone.
- **Pre-compaction.** Capture state before context is summarized away.

## Pre-tool-use hooks: the enforcement layer

The single most valuable hook is the pre-tool-use guardrail. It runs before any tool call executes, sees what the model is about to do, and decides whether to allow it.

Patterns:

- **Block destructive shell commands.** Pattern-match the command; refuse if dangerous.
- **Restrict file operations to allowed paths.** Reject writes outside project boundaries.
- **Require approval for production touches.** Any tool call hitting prod waits for human sign-off.
- **Rewrite for safety.** Add `--dry-run` flags to commands that support them, in confirmation modes.

The harness should support both **blocking** (refuse the call, return an error to the model) and **modifying** (let the call proceed but with adjusted arguments). Both are useful in different scenarios.

## Post-tool-use hooks: the automation layer

After a tool call, you have ground truth: what the agent did and what came back. This is the right time to:

- **Log the action.** Build an audit trail. For PR reviews, security reviews, debugging.
- **Run validators.** File was edited → run the formatter, the linter, the type checker.
- **Update derived state.** File was added → update the file index. Test passed → mark a checkpoint.
- **Trigger external systems.** Code was committed → notify CI. Ticket was closed → update the dashboard.

Post-tool hooks should be fast. If they're slow, they bottleneck the agent loop. For expensive operations, fire-and-forget — kick off the work asynchronously and let the agent continue.

## Session start hooks: the context layer

Session start is the right time to load stable context that should be available throughout the session:

- Project conventions (read CLAUDE.md / AGENTS.md)
- Current branch, recent commits, in-progress work
- Outstanding tasks, handoff notes from prior sessions
- User preferences and active skills

The discipline: load this once, pin it in the prompt, let it cache. Don't re-load on every turn.

## Hooks and prompts

A subtle and powerful hook pattern: hooks can inject content into the model's prompt, not just modify tool calls.

- A pre-tool-use hook detects an unfamiliar API call and injects the relevant API docs before the call executes
- A user prompt hook detects a question the model isn't well-equipped for and injects a routing recommendation ("this question would benefit from model X")
- A session start hook injects the project's full context block

Prompt-injection-style hooks let the harness handle context discovery so the model doesn't have to.

## When hooks are wrong

Hooks are wrong when:

- **The decision genuinely belongs to the model.** Hooks are deterministic; if the right behavior depends on judgment, leave it to the model.
- **The hook is just papering over a bad tool.** If you keep adding hooks to prevent a tool from being misused, the tool needs redesigning.
- **The hook makes the loop too slow.** Synchronous hooks are added latency on every relevant event. Be deliberate about what runs synchronously.
- **The hook is doing what a tool should.** If the agent might want to choose whether to do something, expose it as a tool. Don't force it via hook.

## A useful taxonomy

Hooks fall into four practical categories:

1. **Guardrails.** Block bad behavior. Pre-tool-use is the natural fit.
2. **Quality enforcement.** Run linters, formatters, validators. Post-tool-use is the natural fit.
3. **Context management.** Load, inject, compact. Session start and pre-prompt are the natural fits.
4. **Observability.** Log, metric, trace. Post-tool-use and session end.

If you're not sure where a hook belongs, ask which of these four jobs it's doing. If it's doing more than one, split it.

## Composition

Hooks compose. A single tool call might trigger:

1. Pre-tool-use guardrail (allow / deny / modify)
2. The tool actually runs
3. Post-tool-use logger (record what happened)
4. Post-file-edit linter (run if the tool was an edit)
5. Post-tool-use metrics (capture token cost, latency)

Order matters; the harness should run them in a defined sequence. Most harnesses default to "guardrails first, then automation, then observability" — which is the right default.

## Versioning hooks

Hooks are code that runs on every relevant event. They should be:

- **Version controlled.** In the project repo, not someone's local config.
- **Tested.** A broken hook can crash every agent run.
- **Documented.** Future-you (or a teammate) needs to know what's intercepting their tool calls.
- **Easy to disable.** When debugging, you'll want to turn one off temporarily.

Treat hooks as production code. They are.
