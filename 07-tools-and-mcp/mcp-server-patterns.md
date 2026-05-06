# MCP Server Patterns

Building an MCP server is technically easy — most ecosystems have SDKs that get you to a working server in an hour. Building a server that agents *use well* is the actual craft, and most poorly-performing agentic workflows trace back to badly-designed tools, not bad models.

This page is the design discipline.

## The first principle

**The tool description is a prompt.**

The model reads tool descriptions to decide which tool to call and how. Vague descriptions get random calls. Precise descriptions get the right calls. A model is not "deciding which tool fits" through deep reasoning — it's pattern-matching on the description against the current task. Write descriptions that match the way the model thinks about the task.

Bad: `search` — "Search for things."

Better: `search_engineering_wiki` — "Search the internal engineering wiki for runbooks, onboarding guides, architecture decisions, and team-specific conventions. Returns the top 10 matching pages with snippets. Use this when the user asks about how something is done at this company, or when you need internal context that isn't in the codebase."

The second one tells the model when to call it, what it returns, and what scenarios it fits.

## Naming

Tool names are the model's first signal. Conventions that work:

- **Verb + object.** `read_file`, `query_database`, `create_ticket`. The verb tells the model it's an action; the object tells it what kind.
- **Specific over generic.** `query_user_database` beats `query_db`. The model is less likely to call a specific tool inappropriately.
- **Avoid abbreviations.** `gh_create_pr` is harder for the model to reason about than `github_create_pull_request`.
- **Distinguish read from write.** `get_*` for reads, `update_*`/`create_*` for writes. Models are calibrated to expect this asymmetry.

If you have multiple similar tools (search wiki, search code, search tickets), name them in parallel: `search_wiki`, `search_code`, `search_tickets`. Parallel naming helps the model pick the right one.

## Scoping

Each tool should do one thing. Composite tools that "do a lot" are harder to use correctly:

**Bad:** `manage_ticket(action, ticket_id, fields)` where `action` can be `create`, `update`, `close`, `comment`. The model has to remember which fields apply to which action.

**Better:** Four tools — `create_ticket`, `update_ticket`, `close_ticket`, `comment_on_ticket`. Each with a focused signature.

The exception: when the operations are genuinely interchangeable and the discriminator is obvious, a single tool with a `mode` parameter is fine. The test is whether the model can predict which call shape is correct from the task alone.

## Input schemas

The schema is part of the prompt too. Three rules:

1. **Type every parameter.** No raw `object` or `any`. The model uses types as hints.
2. **Describe every parameter.** Even obvious-seeming ones. "id (string): the ticket ID, formatted as TICKET-1234" is better than just "id".
3. **Mark required vs optional explicitly.** Models will sometimes invent values for required fields and skip optional ones. Make the distinction unambiguous.

For complex inputs (nested objects, arrays of structured items), prefer flatter schemas. The model is more reliable with `assignee_email` and `assignee_name` than with `assignee: {email, name}`.

## Outputs

Tool outputs feed back into the model's context. Structure them for the model, not for human eyes.

- **Return JSON or structured text.** Free-form prose is fine for narrative tools (search, summarize); structured output is better for queries.
- **Include enough metadata to act on.** A search result with just titles is hard to follow up on. Include URLs/IDs/snippets.
- **Truncate aggressively.** A tool that returns 50K tokens of raw data on every call is a cost and confusion problem. Paginate, summarize, or cap.
- **Surface errors clearly.** "Error: ticket TICKET-1234 not found in project FOO" is far more useful than "404 Not Found."

The 80/20: a well-shaped output that's 1K tokens beats a "complete" output that's 20K tokens almost every time.

## Errors and edge cases

Models handle errors well if errors are descriptive. They handle errors poorly if errors are stack traces or generic messages.

**Bad:** `Error: ValueError: invalid literal for int() with base 10: 'abc'`

**Better:** `Error: parameter 'limit' must be a positive integer; received 'abc'. Try calling again with limit=10.`

Tell the model what went wrong and how to fix it. The model will, in most cases, retry correctly.

For destructive operations, include confirmation in the response: "Deleted 3 tickets: TICKET-1234, TICKET-1235, TICKET-1236." If the operation was a no-op, say so explicitly: "No tickets matched the filter; nothing was deleted."

## Destructive operations

Tools that modify state (create, update, delete) need extra care:

- **Mark them clearly in descriptions.** "DESTRUCTIVE:" prefix or a clear flag in the description.
- **Consider requiring an approval step.** Many harnesses support per-tool approval gates. Use them for delete, deploy, send-email-style operations.
- **Make them idempotent where possible.** If the agent calls `create_ticket` twice with the same content, return the existing ticket rather than creating a duplicate.
- **Return enough context to undo.** A delete should return what was deleted (so the agent can describe the action accurately or, if needed, recreate).

## Server scope: one per logical system

A common mistake is building a "universal" MCP server that exposes everything. This is hard to maintain, hard to secure, and produces a confusing tool surface.

Better: one server per logical system. Tickets, monitoring, deployments, knowledge base — each its own server. Compose them at the harness level.

Benefits:
- Each server has a clear owner and lifecycle
- Security boundaries align with system boundaries
- Tool naming stays clean (no `tickets_search` and `wiki_search` competing in one namespace)
- Servers can be added or removed per-agent based on what they need

## Security

The MCP server runs the actual operations. The agent is just deciding when to call. Auth, scoping, and safe operations are the server's responsibility:

- **Authenticate the caller.** Don't trust that the agent harness is the only thing connecting.
- **Apply principle of least privilege.** A read-only agent gets a server that only has read tools.
- **Validate inputs.** SQL injection, command injection, path traversal — all the usual web concerns apply to MCP server inputs.
- **Audit destructive operations.** Log who called what with what arguments. Mandatory for production-facing servers.
- **Rate limit.** Agents can call tools enthusiastically. Cap calls per minute per session.

A poorly-secured MCP server is a serious vulnerability. Treat them with the same discipline as you'd treat a public API.

## Iteration

You won't get tool design right the first time. The signal that a tool is well-designed:

- The model picks the right tool reliably (low rate of "wrong tool called")
- The model uses the tool with correct arguments (low rate of retries on validation errors)
- The tool's outputs feed productively into next steps (the model knows what to do with what came back)

If any of these is failing, iterate the description, schema, or output shape — usually all three.

The cost of iteration is low. Tool descriptions are config, not code. Change a description, restart the server, watch the next session. A good description saves 10x its weight in tokens over a bad one across a session.
