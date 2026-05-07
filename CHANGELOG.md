# Changelog

All notable changes to this repo. The patterns themselves are durable; specific tool names, flags, and provider behaviors are snapshots and may drift over time.

## [0.6.1] — 2026-05-06

### Added
- "Related public repos" section in main README pointing at `five-register-design-system` (the design system this repo's catalogue and leaders memo were built with), `terminal-stack`, and `dotfiles`.
- "Related public repos" line in the Foyer catalogue colophon for the same purpose.

This establishes a consistent cross-linking pattern that future related-repo releases will join.

## [0.6.0] — 2026-05-06

### Added
- **`QUICKSTART.md`** — first-5-minutes entry path with four audience-specific reading routes (engineer / leader / platform builder / safety-focused). Surfaced prominently at the top of the main README and as the lead feature in the Foyer catalogue.
- **`GLOSSARY.md`** — ~30-term single-page lookup for the load-bearing vocabulary used across sections (agent, centaur model, harness, hook, MCP, prompt injection, reflection, spec, spine, etc.).
- **`CONTRIBUTING.md`** — how to suggest fixes, add war stories, contribute new content; how to disagree productively; style conventions.
- **Cloudflare Web Analytics** snippet in both HTML pages (`docs/index.html`, `docs/leaders-memo.html`) — placeholder token, awaiting one-time activation. Free, cookieless, GDPR-compliant.
- **Feedback form** on the Foyer catalogue (Formspree-backed) for non-GitHub readers — placeholder endpoint, awaiting one-time activation.
- **`docs/SETUP-NOTES.md`** — documents the 5-minute activation steps for analytics and feedback form (both genuinely free at expected volumes).

### Changed
- Main `README.md` now leads with a prominent QUICKSTART callout block; adds a "Reference" section linking QUICKSTART, GLOSSARY, CHANGELOG, CONTRIBUTING.
- Foyer catalogue (`docs/index.html`): new "start here" feature box at the top, before the leaders-memo feature; Quick Links footer expanded with QUICKSTART, GLOSSARY, CONTRIBUTING.

## [0.5.0] — 2026-05-06

### Added — seven new pages

**Section 02 (Platform):**
- `cross-platform-parity.md` — Mac/Linux/Windows/WSL parity discipline; the patterns that prevent "works on my machine" failures at the OS layer

**Section 03 (Multi-agent workflows):**
- `reflection-loops.md` — generate → critique → revise pattern; promoted from a sub-pattern inside plan/execute/judge to its own page
- `framework-selection.md` — decision matrix for LangGraph / CrewAI / AutoGen / LangChain / Haystack from a multi-agent workflow perspective; complements the broader tool evaluation in section 08

**Section 04 (Automation):**
- `prompt-injection.md` — the structural vulnerability of every agentic system. Direct vs indirect injection, attack vectors, defenses that work, defenses that don't. Cites Wilson's *Developer's Playbook for LLM Security* (O'Reilly 2024)

**Section 07 (Tools and MCP):**
- `tool-description-as-prompt.md` — promoted from a paragraph inside mcp-server-patterns to its own page; the single most important sub-pattern of tool design

**Section 08 (Resources):**
- `shell-and-terminal-tools.md` — deep reference for shell/terminal tooling specifically for AI engineering; the modern Unix toolchain, multiplexers, runtime managers, fzf widgets, security-first scripting headers
- `cloud-and-deployment-tools.md` — patterns for hosting agents, MCP servers, sandbox envs; always-free gateway, sleeping compute, VPN-only access, container orchestration, serverless

### Changed
- All five affected section READMEs updated with new page entries
- README and leaders guide updated to reflect new content

## [0.4.1] — 2026-05-06

### Changed
- Expanded `docs/index.html` (Foyer catalogue): every section shelf now lists individual pages with descriptions (was previously collapsed to a single entry per section for sections 02-08; only section 01 was fully expanded). The catalogue now functions as a genuine navigation hub from the published Pages site.
- Added a "What's new in v0.4" feature box with direct anchor links to the six new diagrams, the five war stories, and the four pages of section 08.

## [0.4.0] — 2026-05-06

### Added
- Mermaid diagrams in six high-traffic pages: agent loop (sec 01), MCP architecture (sec 01), agentic spine layout (sec 02), plan/execute/judge flow (sec 03), autonomy spectrum L0–L7 (sec 04), three-layer context model (sec 06)
- Five "war story" callouts drawn from real session work (reference-library extraction pipeline, runaway-cost incident, 100× compression discovery, link-audit script bug catch, implicit-context-in-supposedly-generic-docs lesson) — sections 01, 03, 04, 06, 07

### Changed
- Replaced two ASCII-art diagrams with Mermaid equivalents (agent loop in sec 01; MCP architecture in sec 01)

## [0.3.1] — 2026-05-05

### Added
- `08-resources/ecosystem-and-plugins.md` — reconnaissance map of notable open-source repos (Everything Claude Code, Graphify, Graphiti, Mem0, Letta, observability platforms, MCP server catalogs, structured-output libraries, DSPy)

## [0.3.0] — 2026-05-05

### Added
- Section 08 — Resources: dev environment patterns, recommended reading list (curated from a ~150-book reference library), and honest tool evaluations (agent harnesses, multi-agent frameworks, local LLM runtimes, token-saving utilities)
- README and leaders guide updated to reference the new section

## [0.2.0] — 2026-05-05

### Added
- Section 06 — Token efficiency: model tiering, prompt caching, context management, cost estimation
- Section 07 — Tools and MCP: MCP server patterns, hooks, skills, observability
- `LICENSE` (CC BY 4.0)
- Per-section `README.md` landing pages (replaces `overview.md` so GitHub renders folder views cleanly)
- Author attribution in repo README
- This changelog

### Changed
- Renamed `overview.md` to `README.md` in sections 02-05; added new `README.md` to section 01
- Repo README updated to reference all seven sections (was missing 06 and 07)
- Repo README "How to read this" section updated to reflect README.md naming

## [0.1.0] — Initial commits

- Section 01 — Concepts (what is an agent, context and memory, MCP explained, trust and specs)
- Section 02 — Platform (overview, agentic OS spine, dev environment, project template)
- Section 03 — Multi-agent workflows (overview, plan/execute/judge, agent council, multi-LLM review, spec-driven, handoff, failure modes)
- Section 04 — Automation (overview, autonomous control levels, sandbox environments, overnight runs)
- Section 05 — Local LLMs (overview, model selection, benchmarking)
- `GUIDE-FOR-LEADERS.md`
- Initial repo `README.md`
