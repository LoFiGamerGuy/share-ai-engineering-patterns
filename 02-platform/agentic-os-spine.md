# Agentic OS Spine

The "spine" is the small set of files and conventions that turn a regular project into one that any agent can navigate productively. Once you've worked with the spine in place, working without it feels like coding without an editor — possible, but obviously worse.

## What the spine is

A consistent set of files at known paths, with known purposes:

| Path | Purpose |
|------|---------|
| `CLAUDE.md` (or `AGENTS.md`) | Primary agent context |
| `README.md` | Human-facing introduction |
| `Makefile` (or `justfile`) | Standard task entry points |
| `.env.example` | Required environment variables |
| `scripts/` | Operational scripts |
| `specs/` | Written task descriptions |
| `docs/` | Long-form human documentation |
| `.mcp.json` (or equivalent) | MCP server configuration |

Every project has the same shape. Every agent run starts by reading the same files. Every team member knows where to look.

## Why "spine"

The metaphor: it's the thing every agent run hangs off of. If the spine is healthy, the rest of the project's body works. If the spine is misaligned, everything else is awkward and slow.

Concretely, the spine exists so that:
- A fresh agent run can self-orient in <30 seconds
- Cross-project consistency is high (an agent that knows one of your projects knows them all)
- Onboarding (human or AI) doesn't need a tour guide
- Critical operational knowledge is in version control, not in someone's head

## The two file types

Spine files are either **context files** or **operation files**.

**Context files** describe the project. The agent reads them. They're prose-heavy.
- `CLAUDE.md` — project-level context
- `README.md` — public-facing summary
- `docs/*.md` — deep dives, design docs

**Operation files** make things happen. The agent runs them. They're action-heavy.
- `Makefile`, `scripts/*` — task entry points
- `.env.example` — configuration template
- `.mcp.json` — tools the agent can use

A healthy spine has both. Context without operations: agent knows what to do but can't do it. Operations without context: agent can do things but doesn't know which.

## CLAUDE.md as the keystone

The single most important file. It should be:

- **Short.** 50-300 lines. If it's longer, factor parts into `docs/` and link.
- **Current.** Updated when the project structure changes. Stale = harmful.
- **Specific.** "Tests are run with vitest" is good. "We use a testing framework" is useless.
- **Honest.** Document the messy parts, not just the clean ones. Known landmines belong here.

Sections that earn their place:
1. **Project description** (1 paragraph)
2. **Stack** (languages, frameworks, services)
3. **Commands** (the entry points)
4. **Layout** (where things live)
5. **Conventions** (style, patterns, decisions)
6. **Common tasks** (recipes for things you do often)
7. **Known landmines** (things to be careful of)
8. **How we ship** (PR/branch/release conventions)

Sections that usually waste space:
- Long history / "how we got here"
- Aspirational goals not yet implemented
- Detailed API documentation (link to the actual docs)
- Tool advocacy ("we love TypeScript because...")

## Layered context

Big projects benefit from layered context files:

```
CLAUDE.md                 # Top-level, applies to whole repo
src/api/CLAUDE.md         # API-specific conventions
src/web/CLAUDE.md         # Web-specific conventions
docs/CLAUDE.md            # How docs are written here
```

When the agent works in `src/api/`, it loads both the root and the local context. The local file should be focused — only what's specific to this subdirectory. Don't repeat content from the root.

## .mcp.json and tool configuration

If you're using MCP, a project-level config declares which servers the agent should load:

```json
{
  "mcpServers": {
    "github": { "command": "...", "args": [...] },
    "postgres": { "command": "...", "args": [...] },
    "internal-tickets": { "command": "...", "args": [...] }
  }
}
```

This is part of the spine because it determines what tools the agent has. Two projects with different MCP configs are different "operating systems" for the agent.

Don't put real credentials here. Reference environment variables that come from the user's machine, not the repo.

## Specs as institutional memory

A `specs/` directory accumulates the written task descriptions used to drive agent work. Over time it becomes:
- A history of significant changes
- A reference for "how we usually describe this kind of work"
- A starting point for similar future work

Even if you don't religiously write specs for every task, having the directory and writing them for the bigger ones is high-leverage.

## Hooks and skills

Some agent harnesses support hooks (callbacks that fire on certain events) and skills (reusable workflows). When you have these, the spine extends to:

```
.claude/
  hooks/         # Hook definitions
  skills/        # Skill definitions
  commands/      # Custom slash commands
```

These are harness-specific. Keep them in a clearly-named directory and document what they do in `CLAUDE.md`. See [07-tools-and-mcp/hooks-and-skills.md](../07-tools-and-mcp/hooks-and-skills.md).

## Drift and decay

Spines decay. The most common decay patterns:

- **Stale commands.** `CLAUDE.md` lists `npm test` but the project moved to `pnpm`.
- **Outdated layout.** The directory map references files that were renamed.
- **Convention drift.** The team adopted a new pattern but never updated the conventions section.
- **Phantom landmines.** Old warnings about issues that have since been fixed.

The single best practice: **update the spine in the same PR that changes the underlying thing.** If you reorganize directories, the layout section gets updated. If you change build commands, `CLAUDE.md` and `scripts/` move together.

When you start a new session and the agent does something based on stale context, that's the signal. Fix the context first, then the task.

## Spine as portability layer

Because the spine is harness-agnostic, it's also portability. A project with a clean spine works with Claude Code, Cursor, Codex, custom agents — they all read `CLAUDE.md` (or are pointed to read `AGENTS.md`), they all run the same `make test`, they all use the same `.env.example`.

This matters because the agent landscape moves fast. The harness you're using today may not be the one you're using in six months. A spine that doesn't depend on a specific harness is a hedge against that volatility.

## Minimum viable spine

If you're starting from nothing and want the smallest useful spine:

1. Write `CLAUDE.md` — even just 50 lines covering "what is this, how do I run it, what conventions matter"
2. Add a `Makefile` with `setup`, `dev`, `test`, `lint`
3. Add `.env.example` if you have any env vars
4. Add `specs/` directory (can be empty initially)

That's it. Maybe 30 minutes of work. The payback starts on the first agent run.
