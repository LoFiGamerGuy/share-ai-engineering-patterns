# AI Engineering Patterns

A practitioner's reference for building with AI agents — focused on the patterns, infrastructure, and tradeoffs that make AI useful for real engineering work rather than demos.

This repo collects the durable lessons from building agentic systems: how to structure context, when to trust which model, how multiple agents coordinate without falling over, how to keep cost and latency under control, and how to instrument the whole thing so you can tell when it's broken.

> ## ⚡ Start here: [QUICKSTART](./QUICKSTART.md)
>
> Eight sections is a lot. The quickstart routes you to the **three pages worth reading first** based on whether you're an engineer, an engineering leader, a platform builder, or focused on safety. **5 minutes to a working mental model.**
>
> Already know what you're looking for? The [Foyer catalogue](https://lofigamerguy.github.io/share-ai-engineering-patterns/) lists every page in every section with a one-line description.

## Who this is for

- **Engineering leaders** who want to understand how AI is actually being used by their teams and where the leverage is. Start with [GUIDE-FOR-LEADERS.md](./GUIDE-FOR-LEADERS.md).
- **Engineers** who have used Copilot or ChatGPT and want to move into agent-driven workflows, multi-agent orchestration, and autonomous loops. Start with [01-concepts](./01-concepts/).
- **Platform builders** who are putting an agentic spine into a project or organization and want to skip the dead ends. Start with [02-platform](./02-platform/).

## Sections

| # | Section | What's in it |
|---|---------|--------------|
| 01 | [Concepts](./01-concepts/) | Agents, context, MCP, trust models — the vocabulary |
| 02 | [Platform](./02-platform/) | Dev environment, project scaffolding, the agentic spine |
| 03 | [Multi-agent workflows](./03-multi-agent-workflows/) | Plan/execute/judge, councils, multi-LLM review, handoff |
| 04 | [Automation](./04-automation/) | Sandboxes, overnight runs, autonomous control levels |
| 05 | [Local LLMs](./05-local-llms/) | Model selection, benchmarking, when local makes sense |
| 06 | [Token efficiency](./06-token-efficiency/) | Model tiering, prompt caching, context management, cost |
| 07 | [Tools and MCP](./07-tools-and-mcp/) | MCP server patterns, hooks, skills, observability |
| 08 | [Resources](./08-resources/) | Dev environment, recommended reading, tool evaluations, ecosystem map |

## How to read this

Each section has its own `README.md` that orients you. If you're getting the lay of the land, read the eight section overviews — that's roughly an hour and gives you the full landscape. Then drill into specific files based on what you're trying to do.

Nothing here is theoretical. Every pattern came from running into the failure mode and figuring out the way around it.

## Conventions

- "Agent" means an LLM running in a loop with tools, not a single completion call.
- "Frontier model" means the current top-tier model from a major provider. "Local model" means something you run on your own hardware.
- "Spec" means a written description of the work that's specific enough for a different agent (or a different person) to pick up and execute.
- Code samples use shell, Python, or TypeScript depending on what's idiomatic for that pattern. They're illustrative — not copy-paste-ready.

## Examples

Concrete artifacts that show what the patterns look like as files: a real-shape spec, a project CLAUDE.md, and a full plan/execute/judge prompt set. See [examples/](./examples/).

## Rendered version

For a polished, hand-around version, the [`docs/`](./docs/) folder contains:

- [`docs/index.html`](./docs/index.html) — the catalogue / front door for the seven sections
- [`docs/leaders-memo.html`](./docs/leaders-memo.html) — the leaders guide as a one-page memo

Open either file in a browser locally, or host the `docs/` folder via GitHub Pages or any static host.

## Caveat

The patterns here reflect the state of the tools as of writing (May 2026). Models, harnesses, and protocols are moving fast. Treat the *pattern* as durable; treat the specific tool names and flags as snapshots. See [CHANGELOG.md](./CHANGELOG.md) for the version history.

## Reference

- [QUICKSTART](./QUICKSTART.md) — the three pages worth reading first, by audience
- [GLOSSARY](./GLOSSARY.md) — single-page lookup for the load-bearing terms used across sections
- [CHANGELOG](./CHANGELOG.md) — what's been added and when
- [CONTRIBUTING](./CONTRIBUTING.md) — how to suggest fixes, add war stories, or contribute new content

## Related public repos

This repo is part of a small family of public reference material. The patterns here describe *how to work*; the others give you the artifacts, infrastructure, and complementary frameworks.

- **[council-of-five](https://github.com/LoFiGamerGuy/council-of-five)** — Multi-perspective decision framework. Five voices, one Chair, structured deliberation. *Complementary* to the multi-agent patterns documented here — plan/execute/judge parallelizes execution; Council of Five parallelizes perspective. CC BY 4.0.
- **[five-register-design-system](https://github.com/LoFiGamerGuy/five-register-design-system)** — Design system for the artifacts agentic work produces (decks, docs, memos, dashboards, catalogues, chapbooks). MIT licensed. The catalogue and leaders memo for *this* repo were built using it.
- **[terminal-stack](https://github.com/LoFiGamerGuy/terminal-stack)** — Opinionated terminal kit for Git Bash on Windows. 24 themes, modern Unix tools, fzf widgets, multi-pane launchers. Cross-platform parity with the Mac/Linux side.
- **[dotfiles](https://github.com/LoFiGamerGuy/dotfiles)** — Personal dotfiles. Shell config, prompt, tool integrations. Idempotent installer.

More repos in this family will be released over time. The current set will be expanded with explicit cross-links as they ship.

## Author and license

Written by **Ryan Gosnell**. Licensed under [CC BY 4.0](./LICENSE) — share, adapt, build on; just credit the source.

If you find errors, have suggestions, or want to add patterns from your own work, contributions are welcome. See [CONTRIBUTING](./CONTRIBUTING.md). The repo is meant to grow.
