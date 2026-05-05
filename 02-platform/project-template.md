# Project Template

Every new project starts the same way: a checklist of files and conventions that make the project agent-ready from day one. Codify this as a template you can scaffold from.

## Why a template

Without a template, every new project re-discovers the same setup decisions:
- Where does the context file go?
- What do we name the standard scripts?
- How do we structure secrets?
- Where do specs live?
- Where do agent-generated artifacts go?

Each re-discovery is a chance to be inconsistent. After three projects, your repos all feel different and each requires its own context to navigate. With a template, every project has the same shape.

## Minimum template contents

A baseline template has:

```
.
├── README.md                  # Human-facing introduction
├── CLAUDE.md                  # Agent context (or AGENTS.md)
├── .env.example               # Required environment variables
├── .gitignore                 # Standard ignores plus .env
├── Makefile                   # Standard task entry points
├── scripts/
│   ├── setup                  # First-time setup
│   ├── dev                    # Start dev environment
│   ├── test                   # Run tests
│   ├── lint                   # Lint
│   └── format                 # Format
├── specs/                     # Where specs live
│   └── README.md              # How to write a spec for this project
├── docs/                      # Human-facing documentation
└── src/                       # Source code (or app/, lib/, etc.)
```

Adjust per language. Python projects might have `pyproject.toml` and a `src/` layout. Node might have `package.json` and `src/`. Rust gets a `Cargo.toml` and `src/`. The template lives at the layer above the language specifics.

## CLAUDE.md template

The agent context file is the most valuable part of the template. A starter:

```markdown
# Project Name

One-paragraph description of what this project does and who uses it.

## Stack

- Language: <e.g., TypeScript with Node 20>
- Framework: <e.g., Next.js 15>
- Database: <e.g., PostgreSQL 16>
- Tests: <e.g., Vitest>

## Commands

- `make setup` — install dependencies
- `make dev` — start dev server (port 3000)
- `make test` — run all tests
- `make lint` — check formatting and types
- `make build` — production build

## Layout

- `src/` — application code
- `src/api/` — API routes
- `src/lib/` — shared utilities
- `tests/` — test files (mirror `src/` structure)
- `specs/` — written specs for ongoing work
- `scripts/` — operational scripts

## Conventions

- TypeScript strict mode is on. Don't disable.
- Imports use absolute paths from `src/`.
- One component per file, file name matches export.
- Tests live next to source: `foo.ts` and `foo.test.ts`.

## Common tasks

When asked to add a new API endpoint:
1. Add the route handler in `src/api/<name>/route.ts`
2. Add a test in `src/api/<name>/route.test.ts`
3. Update the OpenAPI spec in `specs/api.yaml`

## Known landmines

- Don't run migrations directly. Use `make migrate`.
- The `legacy/` directory is being deprecated. Don't add to it.
- Cache invalidation in `src/cache.ts` has subtle ordering. Read the file before changing.

## How we ship

- Branch from `main`. Branch names: `feat/...`, `fix/...`, `chore/...`.
- Commits use conventional commits format.
- PRs need lint + test green and one reviewer.
```

This is a starting point. Each project will have its own specifics. The template's job is to ensure these sections all exist; the project owner fills them.

## Specs directory

A committed `specs/` directory does two things:
1. Gives a clear home for written task descriptions (so agents can be pointed at them: "implement `specs/2026-04-search-revamp.md`")
2. Builds an institutional memory of what was built and why

Naming convention that works: `YYYY-MM-<short-name>.md`. Sortable, dated, descriptive.

A `specs/README.md` should explain how specs are written here — link to a template, list any required sections, etc. See [01-concepts/trust-and-specs.md](../01-concepts/trust-and-specs.md) for spec content patterns.

## Standard `Makefile` shape

A simple, consistent `Makefile` makes commands predictable across projects:

```makefile
.PHONY: setup dev test lint format build clean

setup:
	./scripts/setup

dev:
	./scripts/dev

test:
	./scripts/test

lint:
	./scripts/lint

format:
	./scripts/format

build:
	./scripts/build

clean:
	rm -rf dist/ .cache/
```

Targets delegate to scripts so the actual logic lives in one place.

## Variations

**Library vs. application.** Libraries don't need `dev`; they do need `publish`. Adjust the template.

**Monorepo.** The template applies to each package; the root has its own `CLAUDE.md` describing the monorepo layout.

**Polyglot.** A `CLAUDE.md` per language directory plus a root one is fine. Layered context handles this naturally.

## Bootstrapping a template

You can store the template as:
- A separate Git repo you `clone --template` from
- A `cookiecutter` / `degit` / `npx create-*` style scaffolder
- A directory in your dotfiles you `cp -r` from

Whichever, the goal is "new project takes <60 seconds to be ready for an agent."

## What not to template

Resist the urge to template:
- Specific tools or libraries (they go stale fast)
- Detailed code patterns (they vary too much)
- Workflow choices that depend on team size (CI configs, deploy patterns)

Template the *shape*, not the *content*. The shape is durable. The content is project-specific.

## Update cadence

Templates rot. Once a quarter, run through your template against a current project and update anything that's drifted. Add new patterns you've found valuable. Remove patterns that turned out to be overhead.
