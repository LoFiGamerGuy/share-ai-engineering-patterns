# Prompt Caching

Prompt caching is the single biggest cost lever in agentic workflows. Most providers now offer some form of it: you mark a stable prefix of your prompt as cacheable, and on subsequent calls within a window (typically minutes), that prefix is billed at 10-30% of normal rates and loads in milliseconds rather than seconds.

The catch: caching only works if you structure your prompts to maximize stable prefix length. Most prompts are accidentally structured to defeat caching, and most teams don't realize it.

## The mental model

LLMs process prompts left to right. The model's internal state after reading the first N tokens is a function of those N tokens. If you send the same first N tokens twice, the model can re-use that state instead of recomputing it.

This means:

- **Stable content goes first.** System prompt, tool definitions, project context, conventions.
- **Variable content goes last.** The current task, the latest tool result, the user's new message.
- **Even one differing token early in the prompt breaks the cache.** A timestamp at the top of the system prompt invalidates the cache on every call.

If your prompt is `[system + tools + context + task]` and you only change `task`, you cache `[system + tools + context]` once and pay full price only for the new task. Multi-turn conversations can keep extending the cache through the cached prefix as long as nothing earlier changes.

## What to put in the cached prefix

In order, from front to back:

1. **System prompt.** Stable across all calls in a session.
2. **Tool definitions.** Change only when tools change (rarely mid-session).
3. **Project context / conventions.** Things like CLAUDE.md, AGENTS.md, repo conventions.
4. **Reference material.** Spec docs, design decisions, relevant file contents the agent needs to know about.
5. **Conversation history.** Older turns, in order.

Then the variable parts (current turn, latest tool result) come after.

## What breaks the cache

- **Timestamps or session IDs at the top of the prompt.** Always different, always invalidates.
- **Reordering content.** The same content in different order is a different prefix.
- **Inserting a new file in the middle.** Anything after the insert point is no longer cached.
- **Provider-side cache eviction.** Caches expire (typically 5 minutes). Long pauses lose the cache.
- **Switching models.** Each model has its own cache.

## Practical patterns

### The append-only conversation

Don't rebuild the full conversation each turn. Append new turns to the end of the existing prompt. The model harness should handle this if it's well-designed; if you're rolling your own, build append-only.

### The pinned context block

Define a `<context>` block early in the system prompt that contains everything stable about this session. Keep it stable for the session lifetime. If you need to add information, add it to a separate, later block that is allowed to grow — don't edit the pinned block.

### Cache-aware retrieval

If you're injecting retrieved chunks into the prompt, retrieve them once and pin them. Re-retrieving every turn (with possibly different chunks) breaks caching. Acceptable strategies:

- Retrieve once at session start, pin the result, accept that it's stale
- Retrieve incrementally and append (cache extends through new appends)
- Re-retrieve only when explicitly needed; rebuild the cache

### The 5-minute heartbeat

Caches expire on inactivity. If a session naturally has long pauses (overnight runs, human review gates), expect cache misses on resume. For continuous work, the cache stays warm.

## How much it actually saves

Realistic numbers for a typical agentic engineering session:

- **No caching:** every call pays for the full input prefix (often 50-200K tokens of context)
- **With caching, warm cache:** input cost drops 70-90% on each cached call
- **End-to-end cost reduction:** 40-60% across a typical session, more for long sessions with stable context

The savings are larger than they sound because input tokens dominate output tokens by 5-10x in most agentic work. Cutting input cost cuts the bigger half.

## When caching doesn't help

- **One-shot calls.** Single completions don't benefit; there's no second call to hit the cache.
- **Highly varied prompts.** If every call has a different system prompt or different early context, nothing caches.
- **Very short prompts.** The overhead of cache management exceeds the savings.
- **Cross-session work.** Caches typically don't persist across sessions or processes.

## What to measure

Two metrics tell you if caching is working:

- **Cache hit rate.** What percentage of input tokens were served from cache. Healthy: 60-90% during active sessions.
- **Cost per task.** End-to-end cost should drop noticeably when caching is on. If it didn't, your prompt structure is defeating the cache.

Most providers expose cache hit metrics in the API response. Log them. If a session that should be cache-friendly is showing low hit rates, your prefix isn't stable — find what's varying and pin it.

## A common anti-pattern

Teams sometimes "freshen" their prompts by inserting the current date and time at the top of the system prompt, thinking the model needs it. This invalidates the cache on every call. If the model needs the time, put it at the end of the prompt, not the start. The same goes for any per-call metadata: it goes after the cached prefix, not in it.
