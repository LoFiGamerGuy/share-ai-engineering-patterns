# Dev Environment

The shell, editor, and tooling configured so an agent can run commands without surprises. This is unglamorous and matters a lot.

## What an agent expects

When an agent runs a shell command, it expects:
- The command to exist on `PATH`
- The right version of the tool (no major version mismatches)
- Any required environment variables to be set
- The working directory to be sensible
- Output to be parseable (text, ideally with structured options)

Anything that violates these expectations becomes an error the agent has to recover from. Recovery costs context and tokens, and sometimes fails entirely.

## The minimum viable environment

For any project the agent will work in:

**Shell.** A modern shell (`bash` 4+ or `zsh`). Set a clear prompt that doesn't break parsing. Don't use prompts with embedded color codes that confuse output capture.

**Language runtimes.** Pinned versions via a manager (`nvm`/`volta` for Node, `pyenv`/`uv` for Python, `rustup` for Rust, etc.). The version file should be in the repo (`.nvmrc`, `.python-version`, `rust-toolchain.toml`). The agent reads it, the manager resolves it.

**Package manager.** Whatever the language uses (`npm`, `pnpm`, `pip`, `uv`, `cargo`, `go`). Agent should be able to run `<install command>` and have it work.

**Build/test runner.** A standard entry point — usually a `Makefile` or `justfile` — so the agent doesn't need to memorize project-specific incantations. See "scripts directory" below.

**Editor-agnostic.** Don't put critical workflow inside an IDE plugin. Agents work from the shell. If your build only works through a "click here in VS Code" workflow, agents are blocked.

## The scripts directory

A `scripts/` (or `bin/`) directory with one executable per common task. The agent is told (in `CLAUDE.md`) that these exist.

Examples:
```
scripts/
  setup        # idempotent first-time setup
  dev          # start the dev server
  test         # run all tests
  test-unit    # run unit tests only
  lint         # run lint
  format       # auto-format
  build        # production build
  ci           # what CI runs
```

Why scripts and not just `Makefile` targets: scripts are more flexible (can be Python, Bash, anything), they show up cleanly in process trees, and they're easier for an agent to read when it wants to understand what a command does.

You can have a `Makefile` that delegates to scripts. That's fine.

## Environment variables

Two practices:

**`.env.example`.** A committed file showing every environment variable the project needs, with placeholder values. The agent reads this to understand configuration.

```
# .env.example
DATABASE_URL=postgres://localhost:5432/myapp
API_KEY=your-key-here
LOG_LEVEL=info
```

**Loaders.** The project loads `.env` automatically (via `dotenv` or equivalent). Don't make the agent figure out how to source variables manually.

Never commit a real `.env`. Add it to `.gitignore` from day one.

## Shell hygiene for agents

A few things to set in your `.bashrc`/`.zshrc` for environments where agents run:

- **Disable interactive prompts.** No `set -o vi`-style modes that change behavior. No prompts that wait for user input on every command.
- **Disable color in pipes.** Many tools output ANSI codes when stdout is a TTY but plain text when piped. This is mostly automatic but check.
- **Set `LANG`/`LC_ALL`.** Avoid locale-induced surprises in sort, awk, etc.
- **Don't auto-activate things that change behavior.** `direnv` can be useful but make sure agents understand when they've entered a directory whose env is different.

## Containerization

For shared team environments and CI, a Dev Container (`devcontainer.json`) or Dockerfile gives every agent the same starting point. Worth it for medium-and-up projects.

For solo work, a container is often overkill. A clear `setup` script that installs the right versions is usually enough.

## Sandboxing

When you let an agent run unattended, you want a sandbox — a place where the agent's actions are bounded. The bounds can be:
- A throwaway VM
- A container with no network access to production
- A WSL distribution dedicated to agent work
- A separate user account with limited permissions

See [04-automation/sandbox-environments.md](../04-automation/sandbox-environments.md) for patterns.

For interactive use where you're watching, a sandbox is less critical. The agent can ask before doing something risky, and you can say no.

## What goes wrong

- **"It works on my machine" but the agent's machine differs.** Pin versions in the repo. `.nvmrc`, `.python-version`, lockfiles committed.
- **Hidden state in the user's shell.** Aliases or functions defined in `.bashrc` that the agent doesn't know exist. Avoid them for project commands; put them in `scripts/` instead.
- **Missing tools.** Agent assumes `jq` is installed, but it isn't. Document required tools in `CLAUDE.md` or detect-and-install in `setup`.
- **Slow setup.** First-time setup takes 20 minutes; agent gives up. Cache aggressively. Make `setup` idempotent so the agent can re-run without harm.

## Practical checklist

For any project where an agent will be working:

- [ ] `setup` script that gets a fresh checkout to "ready to develop"
- [ ] Standard tasks accessible via short commands (`make test` or `./scripts/test`)
- [ ] Pinned tool versions in repo
- [ ] `.env.example` with all required variables
- [ ] Common errors documented (in `CLAUDE.md`) with their fixes
- [ ] Lint/format runnable in one command
- [ ] Tests runnable in one command, with subset commands for fast feedback

If you can hand a fresh laptop to a new team member and they can run `./scripts/setup && ./scripts/test` and it works, your environment is agent-ready.
