# Observability

You cannot improve what you cannot see. Agentic workflows are particularly opaque by default — the agent makes dozens or hundreds of decisions per session, each involving an LLM call, a tool call, or a context shift. Without instrumentation, when something goes wrong (or right), you have no way to understand why.

Observability is the discipline of capturing the right signals from agent runs so you can debug, improve, and trust the system.

## Three layers

Observability for agents operates at three layers:

1. **The session trace.** The full sequence of LLM calls, tool calls, and decisions for one agent run.
2. **Aggregate metrics.** Cost, latency, success rate, error patterns across many runs.
3. **Audit trail.** A permanent record of consequential actions (writes, deploys, sends) for compliance and forensics.

Each answers different questions. Build all three.

## The session trace

For any single run, you should be able to reconstruct:

- The initial prompt and context
- Every LLM call (input, output, model, latency, token cost)
- Every tool call (name, arguments, result, latency)
- Every hook fire (event, action taken)
- Final state (success / failure / aborted, duration, total cost)

This is the debugging primitive. When an agent does something surprising, you go to the trace, find the decision point, see what context the model had, and understand why it chose what it chose.

Most harnesses produce something trace-like by default. Read it; learn its format; make sure it captures what you need. If it's missing something important, instrument that yourself.

### Trace storage

Traces add up. A long agent session can produce 1MB+ of trace data. For a team running thousands of sessions, you need a strategy:

- **Hot storage** (last 30 days): full traces, indexed for search.
- **Warm storage** (last year): summarized traces, key decisions and outcomes preserved.
- **Cold storage / archive** (longer): aggregate metrics only, raw traces deleted unless flagged.

For traces involving consequential actions (production touches, sensitive data), keep the full trace longer for compliance.

## Aggregate metrics

The metrics that matter for agentic workflows:

**Cost.** Per session, per task type, per developer, per team. Detect outliers; spot bills before they surprise.

**Latency.** Time-to-first-token (responsiveness), time-to-completion (throughput). Both matter.

**Success rate.** What percentage of agent runs achieve their stated goal? Define "success" carefully — completion isn't success if the output is wrong.

**Tool call distribution.** Which tools get called most? Which never? Tools that are never called are either useless, badly described, or in the wrong harness for the job.

**Loop length.** Number of LLM calls per session. Watch for outliers — runaway loops or unexpectedly short sessions.

**Cache hit rate.** Percentage of input tokens served from cache. Direct signal of whether your prompt structure is working.

**Error rates by type.** Tool errors, model errors, hook blocks. Patterns reveal what to fix.

Pipe these to whatever observability stack you already have. Don't build a separate one for AI.

## Audit trail

For consequential actions, you need a record that survives independent of the agent harness:

- File modifications (with diff)
- Code commits and PRs
- External calls (API requests, emails sent, tickets created)
- Deployments and rollbacks
- Anything touching production

Audit trail entries should be immutable, timestamped, attributable to the agent run that produced them, and linked back to the trace for context.

For regulated environments, audit requirements may dictate retention periods and required fields. Comply with those before adding nice-to-haves.

## What "good" looks like

A team with good observability can answer questions like:

- "What did the agent do during the 4am incident?" — pull the session trace.
- "Why did our AI bill spike on Tuesday?" — drill into per-task cost; find the long-running session.
- "Which tools are getting used and which aren't?" — query the tool-call distribution.
- "Has cache hit rate degraded since we changed the system prompt?" — compare metrics across the change.
- "Show me every production deploy made by an agent in the last 30 days, with the trace for each." — query the audit trail; cross-reference with traces.

A team without observability can't answer any of these confidently. They guess.

## Practical setup

A reasonable starting setup:

- **Structured logging on every LLM call and tool call.** JSON, with consistent field names. Pipe to a log aggregator.
- **Per-session metrics emitted at session end.** Cost, duration, success/failure, key counts.
- **Audit log table.** A separate, append-only store for consequential actions. Linked to session ID.
- **A small dashboard** with the top-level metrics: daily cost, success rate, p50 / p95 latency, top error types.

You don't need a fancy "AI observability platform" to start. The core needs are met by the same tools you use for everything else. Specialized tooling becomes worth it when you're running enough sessions that custom analysis becomes the bottleneck.

## What to alert on

Three classes of alerts worth setting up early:

1. **Cost.** Per-session ceiling exceeded → kill the session. Daily team budget exceeded → page someone.
2. **Errors.** Sustained error rate above threshold → investigate. A single tool failing 100% of calls → it's broken.
3. **Audit anomalies.** A production deploy outside business hours, an agent making changes without a corresponding ticket, an unusual sequence of writes — anything that should be reviewed.

Don't alert on every individual failure. Agents fail at things constantly; that's normal. Alert on patterns and on consequential exceptions.

## Privacy and PII

Traces capture everything the agent saw and produced. That can include:

- Code (potentially proprietary)
- Customer data (potentially PII)
- Credentials (if the agent accessed them)
- Conversations (potentially sensitive)

Three rules:

1. **Treat traces as sensitive.** Same access controls as production logs.
2. **Redact known PII patterns** at trace-write time, not after. Easier than retroactive cleanup.
3. **Don't ship traces to third-party services without thinking.** "Send everything to vendor X for analysis" is a data-exposure decision, not just a tooling decision.

## The long view

Most teams under-instrument early and regret it. The cost of adding observability later — re-running sessions, reconstructing decisions from incomplete data, debating what happened — is much higher than the cost of capturing it from day one.

If you're starting, capture more than you think you need. Storage is cheap. Lost context is expensive.

If you've been running a while without good observability, prioritize getting it in place. The next hard incident is when you'll wish you had it.
