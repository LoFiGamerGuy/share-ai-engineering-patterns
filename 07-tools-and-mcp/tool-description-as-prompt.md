# Tool Description as Prompt

A single principle that, applied consistently, will improve your agentic systems more than almost any other change to tool code: **the tool description IS the prompt**. The model decides whether to call your tool, and how, by reading what you wrote in the description. Bad descriptions get bad calls. Good descriptions get good calls.

This is broken out as its own page because it's the most subtle and load-bearing aspect of [mcp-server-patterns.md](./mcp-server-patterns.md). Every MCP server author should internalize it.

## The principle

When the LLM is deciding which tool to call, it doesn't reason from your tool's behavior. It can't — it has never run your tool. It reasons from the *description* in the tool definition: name, parameter schema, parameter descriptions, return value description, and any narrative description.

That metadata is, functionally, prompt content. The model reads it the same way it reads any other instruction. Vague metadata produces vague tool selection. Specific metadata produces correct tool selection.

The implication: **writing tool descriptions is prompt engineering**, not documentation. The audience is a language model deciding when and how to act, not a human reading your README later.

## Bad vs good descriptions

The bad version:

```
Tool: search
Description: "Search for things."
Parameter: query (string)
```

A model reading this has no signal about:
- What it's searching (the codebase? the web? a wiki?)
- When it should call this tool versus a different search tool
- What the output looks like

The model will call this tool randomly. Sometimes correctly, sometimes not.

The good version:

```
Tool: search_engineering_wiki
Description: "Search the internal engineering wiki for runbooks,
onboarding guides, architecture decisions, and team-specific
conventions. Returns the top 10 matching pages with title, URL,
and a 200-character snippet. Use this when the user asks about
how something is done at this company, or when you need internal
context that isn't in the codebase."
Parameter: query (string) — "Free-text search query. Best results
come from descriptive phrases (e.g., 'oncall rotation policy')
rather than single keywords."
```

A model reading this knows:
- What corpus is being searched (internal eng wiki)
- What kinds of content live there (runbooks, onboarding, conventions)
- When to use this tool vs alternatives ("internal context that isn't in the codebase")
- What the output shape looks like (top 10, with title/URL/snippet)
- How to formulate good queries (descriptive phrases)

The model will call it correctly far more often.

## What goes into a good tool description

A good description answers four questions:

1. **What is this tool?** A precise verb phrase: "Search the eng wiki" not "Wiki search functionality"
2. **When should I use it?** Trigger conditions — what kinds of tasks or situations call for this tool
3. **What does it return?** Output shape, sometimes with an example. The model uses this to plan downstream actions.
4. **What constraints / tips matter?** Rate limits, formatting requirements, performance notes, anything non-obvious

If your description is missing any of these four, you'll get tool calls that miss in predictable ways.

## Naming as the first signal

Names matter because they're the first thing the model sees in the tool list. Conventions that work:

- **Verb + object.** `read_file`, `query_database`, `create_ticket`. The verb says it's an action; the object says what kind.
- **Specific over generic.** `query_user_database` beats `query_db`. Specific names get specific use; generic names get used randomly.
- **Avoid abbreviations.** `gh_create_pr` is harder to reason about than `github_create_pull_request`. Tokens are cheap; ambiguity is expensive.
- **Distinguish read from write explicitly.** `get_*` for reads, `create_*` / `update_*` / `delete_*` for writes. Models are calibrated to expect this asymmetry.

Parallel naming for related tools helps the model pick correctly:

- `search_wiki`, `search_code`, `search_tickets` — clear which to use for what
- `wiki_search`, `code_grep`, `find_tickets` — same capabilities, harder to reason about

## Parameter schemas are part of the description

Every parameter has a name, a type, and (in well-designed schemas) a description. All three are prompt content.

**Type every parameter.** No raw `object` or `any`. The model uses types as hints. A parameter typed as `string` is interpreted differently from one typed as `integer` or `enum`.

**Describe every parameter.** Even obvious-seeming ones. "id (string)" tells the model nothing useful. "id (string) — the ticket ID, formatted as TICKET-1234" tells the model exactly what to pass.

**Mark required vs optional explicitly.** Models will sometimes invent values for required fields and skip optional ones. Make the distinction unambiguous in the schema.

**Prefer flat schemas over nested.** The model is more reliable with `assignee_email` and `assignee_name` than with `assignee: {email, name}`. Nested objects produce more parameter errors.

**Use enums when the value space is bounded.** A `status` parameter with `enum: ["open", "in_progress", "closed"]` is far more reliable than a free-text `status` field with the values listed in the description.

## Output descriptions matter too

The output description tells the model what it'll get back. This affects whether the model knows what to do next.

Bad: `Returns search results.`

Better: `Returns up to 10 results, ordered by relevance. Each result has: title (string), url (string), snippet (string, ~200 chars). Returns empty array if no matches.`

The model reading the better version knows it can access `.title`, `.url`, `.snippet` fields, knows the result limit, knows that empty-array means no matches. The bad version forces the model to inspect actual output to figure all this out — slower, more failure-prone, and produces worse follow-up calls.

## Describing when NOT to use a tool

Often more useful than describing when to use it. The model's failure mode is calling tools in situations where they don't fit. Saying so explicitly helps:

```
Description: "Get the current weather for a city.
NOT for historical weather (use get_weather_history instead).
NOT for weather forecasts (use get_weather_forecast instead)."
```

Three sentences eliminate a huge class of misroutes. The model that reads "NOT for forecasts" won't call this tool when the user asks about tomorrow.

## Describing destructive operations

Mark them clearly. Treat as both safety control and routing aid:

```
Description: "DESTRUCTIVE: Permanently deletes a ticket and all
its comments. Cannot be undone. Use only when explicitly
requested by the user, never as a side-effect of other work."
```

The "DESTRUCTIVE:" prefix is a strong signal. Many harnesses also use such prefixes to gate tools behind approval — see [hooks.md](./hooks.md). Even without harness gating, the prefix changes how the model reasons about calling.

## Iteration

You won't get descriptions right the first time. Three signals that a description is working:

1. **The model picks the right tool consistently.** Low rate of "wrong tool called." If the model keeps reaching for `search_code` when the user asked about wiki content, the descriptions need clarification.
2. **The model passes correct parameters.** Low rate of validation errors. If the model keeps passing wrong-format IDs, the parameter description needs an example.
3. **The model's follow-up actions make sense.** It uses the output productively. If it's making redundant calls or asking the user to clarify what came back, the output description was insufficient.

If any of these is failing, iterate the description. Tool descriptions are config, not code — change a description, restart the server, watch the next session. Cycle time is minutes.

## A useful test

Before deploying a tool, take its description (and only the description — not your knowledge of the implementation) and ask: *if this description were the only thing the model knew about this tool, would the model use it correctly?*

If you can't answer "yes" with high confidence, the description needs more work.

## When the model uses a tool wrong

The instinct is to blame the model. Usually it's the description. Audit:

- **Wrong tool called.** The description's "when to use" is unclear, or two tools have overlapping descriptions and the model can't distinguish them.
- **Right tool, wrong arguments.** Parameter descriptions are missing or examples are absent.
- **Right tool, right arguments, wrong follow-up.** Output description is missing or doesn't match actual output shape.

In all three cases, the fix is description, not model.

## Why this matters at the platform level

A team that takes tool description seriously builds a compounding asset. Every new agent the team spins up benefits from descriptions that have been refined across many sessions. A team that doesn't will keep encountering "the model used this tool wrong" and treating each instance as a model problem rather than a description problem.

For internal MCP servers, the descriptions are owned and editable. There's no excuse for shipping vague ones. For third-party MCP servers, the descriptions are what they are — but you can wrap them, add prefixes, narrow allowed parameters, and improve descriptions at the wrapper layer. Curate the tool surface; don't accept the defaults.

## See also

- [mcp-server-patterns.md](./mcp-server-patterns.md) — the broader page on MCP server design; this is the most important sub-pattern from it
- [hooks.md](./hooks.md) — pre-tool-use hooks that gate destructive operations marked in tool descriptions
- [skills.md](./skills.md) — skill descriptions follow the same prompt-engineering discipline
- [04-automation/prompt-injection.md](../04-automation/prompt-injection.md) — tool descriptions are part of the trust boundary; understand the risks before publishing tool surfaces

---

*Snapshot: May 2026. Patterns are durable; specific tool names, model versions, and provider behaviors are point-in-time. See [CHANGELOG](../CHANGELOG.md).*
