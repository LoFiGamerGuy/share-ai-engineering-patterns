# Contributing

This repo is meant to grow. Patterns evolve, tools change, new failure modes surface, and the collective field is moving fast. Contributions of all sizes are welcome — from a one-line typo fix to a whole new section.

## What I'm looking for

In rough priority order:

1. **Corrections.** If something is wrong, say so. The repo is opinionated; opinions can be wrong. A correction with reasoning beats a correction without.
2. **War stories.** Real "we tried X, it broke because Y, we now do Z" anecdotes. The most credible content in the repo. Anonymized to whatever degree your context requires.
3. **Pattern additions.** New patterns that have earned their place through actual practice. Not theoretical.
4. **Tool / repo additions** to the ecosystem map and tool evaluations. Especially for tools you've used at scale.
5. **Reading recommendations.** Books that genuinely changed how you work.
6. **Diagrams.** Mermaid-rendered. Aim for legibility over completeness.
7. **Translations.** If you want to translate a section into another language, open an issue first so we can discuss structure.

## What I'm not looking for

- **Speculation about the future of AI.** This repo is about what's working now. Forecasts age in months.
- **Tool partisanship.** "X is better than Y because I like X" doesn't move the patterns. "X is better than Y because of [specific tradeoff]" does.
- **Generic content from blog posts.** If a pattern can be assembled by reading three Medium articles, it doesn't need to live here.
- **AI-generated content with no review.** This whole repo is co-authored with AI, so I'm not anti-AI-content — but everything that ships here was reviewed and edited. Drive-by AI submissions will get the same scrutiny.
- **Marketing for your tool.** Tool mentions are evaluated on merit, not promotion. Self-promotion is fine; promotion *as if it weren't* is not.

## How to contribute

### For small changes (typo, broken link, factual correction):

1. Open a PR with the fix
2. In the PR description, briefly say what changed and why
3. That's it

### For new patterns or substantial additions:

1. Open an issue first describing what you want to add
2. We'll discuss whether it fits the structure and how it should be scoped
3. Then open a PR with the content

This avoids the "spent 10 hours writing a page that doesn't fit the repo" failure mode.

### For war stories:

These are the highest-credibility content in the repo. Format: ~80-150 words, sidebar-callout style (use `>` blockquote markdown), placed in the relevant section page. See existing war stories in `01-concepts/what-is-an-agent.md`, `03-multi-agent-workflows/plan-execute-judge.md`, etc. for the shape.

What makes a good war story:
- Specific (real numbers, real artifacts; not "we ran into issues")
- Anonymizable (the lesson generalizes; the project specifics don't need to)
- Has a clear "what we changed because of this" outcome
- Honest about what was wrong before, including if it was your fault

What doesn't make a good war story:
- Vague gripes about a tool
- "Just trust me, we hit X" without specifics
- Tool-bashing dressed as a war story

## How to disagree productively

This repo is opinionated and the opinions might not match yours. If you think a recommendation is wrong:

1. **State your alternative.** "I think X" is more useful than "X is wrong."
2. **Bring evidence.** Personal experience, links to sources, benchmarks, anything concrete.
3. **Acknowledge the tradeoff.** If your alternative wins on dimension A but loses on dimension B, say so. The repo is full of tradeoffs; pretending yours doesn't have any weakens the argument.
4. **Open it as an issue.** Discussion in PR comments is fine for narrow technical points; broader disagreements about direction belong in issues so others can join.

If we disagree and can't resolve it, that's fine. The repo will reflect the maintainer's view; your view can live in a fork or your own writing. The point is honest exchange, not unanimity.

## Style conventions

To keep the voice consistent:

- **Short sentences.** "X happens because Y" beats "It is the case that X happens, which is attributable to Y."
- **Concrete over abstract.** Name the actual tool, version, behavior, or number. Avoid "various tools" or "many teams."
- **No hype.** "Game-changing," "revolutionary," "best-in-class" — all banned.
- **No padding.** Skip "great question," "I hope this helps," "let me know if you have questions."
- **Provider-agnostic where possible.** If a pattern works across providers, write it that way. If it's provider-specific, label it.
- **Snapshot date the content if it's likely to age.** Most pages have a `Snapshot: <month>` footer.
- **Cite sources when borrowing specifics.** Books, papers, blog posts — credit where due.
- **Use tables and lists for scannable content.** Walls of prose are harder to use as reference.

## Format expectations

- **Markdown** for everything except the rendered HTML in `docs/`
- **Mermaid** for diagrams (renders natively on GitHub)
- **CC BY 4.0 license** applies to all contributions
- **Internal links use relative paths** (e.g., `[link](./page.md)` not absolute URLs to GitHub)
- **External links to other repos / sites** are fine and encouraged

## Code of conduct

Be the kind of person you'd want a contributor to be. Specifically:

- Disagree with ideas, not people
- Ask before assuming bad faith
- Thank people who correct your mistakes
- Don't be precious about your contributions; the repo's voice is consistent because edits happen

## Recognition

Substantive contributors will be acknowledged in the README. War story contributors can choose to be named or anonymous (just say which in the PR).

## Questions

- For repo-specific questions: open an issue
- For broader questions about the patterns themselves: open a discussion (or an issue if discussions aren't enabled)
- For private feedback: see the contact links in the [docs/leaders-memo.html](./docs/leaders-memo.html) signature block

## Maintainer

Ryan Gosnell — [GitHub @LoFiGamerGuy](https://github.com/LoFiGamerGuy)

---

*This file follows the same CC BY 4.0 license as the rest of the repo.*
