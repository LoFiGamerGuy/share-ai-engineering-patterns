# Changelog

All notable changes to this repo. The patterns themselves are durable; specific tool names, flags, and provider behaviors are snapshots and may drift over time.

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
