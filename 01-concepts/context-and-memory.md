# Context and Memory

Agents have no memory between sessions. Everything they know about your project comes from the **context** assembled at the start of each run plus whatever they discover during the loop.

This is the most important thing to get right. Context quality dominates almost every other variable.

## The two memory layers

**Working memory (the context window).** Everything in the current conversation: the system prompt, the task, all the tool calls and results so far, all the files that have been read. Models have a fixed limit (often 200K tokens; some 1M). When you exceed it, things fall out or the run dies.

**Persistent memory (files on disk).** Anything you write to a file persists across sessions. The standard pattern is to keep durable knowledge in markdown files at known paths and have agents read them at the start of each run.

There is no third layer. Models do not "remember" your project. If you didn't put it in context, it isn't there.

## The context file pattern

Most agent harnesses read a project-level context file by default. Common names:

- `CLAUDE.md` — Claude Code
- `AGENTS.md` — Codex, generic
- `.cursorrules` — Cursor
- `GEMINI.md` — Gemini CLI

Same idea: a markdown file at the repo root that the agent loads automatically. It should contain:

1. **What this project is** — one paragraph
2. **How to run it** — install, build, test, run commands
3. **Conventions** — language, formatting, style decisions that aren't obvious from the code
4. **Where things live** — high-level directory map
5. **What to avoid** — known landmines, deprecated paths, things that look right but aren't
6. **How to ship** — commit message format, PR conventions, branch naming

A good context file is between 50 and 500 lines. Longer than that and it competes with the actual task for attention.

## Context decay

Context files rot. They were correct when written, then the codebase moved, and now they describe a project that no longer exists. An agent reading a stale context file confidently makes wrong moves.

Two practices help:
- **Update with structural changes.** When you reorganize directories, refactor build scripts, change conventions — update the context file in the same PR.
- **Periodically re-read with fresh eyes.** Every quarter, read your `CLAUDE.md` as if you'd never seen the project. Anything that's wrong, fix.

## Layered context

Beyond the project-level file, you can have layered context:

- **User-level** (`~/.claude/CLAUDE.md`) — preferences that apply to all your work: tone, default languages, things you always want.
- **Project-level** (`./CLAUDE.md`) — the project specifics.
- **Subdirectory-level** (`./src/api/CLAUDE.md`) — local conventions for a specific area, loaded when the agent works there.
- **Task-level** — the prompt for the current run.

Layers are additive. The agent sees the union. Use this to keep each layer focused: don't put project specifics in user-level config, don't put global preferences in project files.

## Working memory hygiene

Within a single run, the context window fills up as the agent reads files and accumulates tool results. Practices that keep it useful:

- **Read the right files, not all the files.** An agent that reads the entire repo on every run wastes context. Encourage targeted reads (specific paths, grep first).
- **Summarize before delegating.** When dispatching a sub-agent, hand it a summary, not the full conversation.
- **Compact periodically.** Some harnesses support summarizing earlier parts of the conversation when memory gets tight. Use it.
- **Start fresh for unrelated tasks.** Don't keep one long-running session for everything. Discrete tasks get discrete sessions.

## What to put in context vs. retrieve from a tool

Static, frequently-needed info → put in the context file. (Build commands, conventions, key paths.)

Large reference material → leave on disk, point the agent at it. (API specs, design docs, schemas.)

Live data → expose via a tool. (Current PR status, ticket details, monitoring metrics.) Don't paste them into context — they go stale within seconds.

## The pitfall of "more context = better"

Tempting heuristic: load as much as possible into context, the agent will figure out what's relevant. Doesn't work. Long contexts:

- Cost more (you pay per token in)
- Are slower (latency scales with context length)
- Confuse the model (relevant info gets diluted by noise)
- Hit limits sooner (less room for tool results)

The skill is **selecting** what context the agent needs for *this* task, not maximizing volume. A 500-line `CLAUDE.md` plus a precise 30-line task description outperforms a 5000-line context dump in almost every case.

## Memory as a design problem

When you find an agent doing the same wrong thing repeatedly, the answer is rarely "use a smarter model." It's usually:

- The context file doesn't say not to do that thing
- The context file says to do it but the wording is ambiguous
- The right tool isn't available so it improvises
- The task description didn't make the constraint explicit

Treat memory and context as a first-class design problem. Most "the agent is dumb" complaints are actually "we didn't tell it the thing it needed to know."
