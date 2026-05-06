# Changelog

All notable changes to this repo. The patterns themselves are durable; specific tool names, flags, and provider behaviors are snapshots and may drift over time.

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
