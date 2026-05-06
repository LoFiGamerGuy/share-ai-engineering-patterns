# Skills

A **skill** is a packaged capability the agent can discover and invoke. It's bigger than a single tool and smaller than a whole workflow. A well-built skill encapsulates a chunk of expertise — a set of instructions, references to tools, sometimes a reusable prompt — that the agent loads on demand when the task fits.

The key word is *discover*. Skills are surfaced to the model so the model can decide when to use them. Tools and hooks are always-on. Skills are opt-in.

## What a skill typically contains

A skill packages some combination of:

- **A name and description.** What it does, when to use it.
- **Instructions.** A focused prompt or set of guidelines for the task.
- **References to tools.** Either tools the skill assumes are available, or specific tool calls it recommends.
- **Templates or examples.** Reusable artifacts (a code skeleton, a doc template, a checklist).
- **Sub-resources.** Files the skill references on demand (extended instructions for specific subcases).

The exact format is harness-specific. The pattern is universal.

## Skills vs tools

A tool is a verb — `read_file`, `query_database`. The model calls it with arguments and gets a result.

A skill is a recipe — "How to write a release note in our team's style", "How to triage a production incident". The model invokes it as a unit and follows its guidance.

Tools are atomic. Skills are composite. A skill often *uses* multiple tools internally — read the changelog, query the issue tracker, format the result — but presents as one capability to the model.

## Skills vs prompts

A prompt is something you write at the moment of use. A skill is something you *captured* from prior use so you don't have to write it again.

The classic moment to extract a skill: you've now solved the same kind of problem three times with slightly different prompts. The fourth time, write down the steps that worked, save it as a skill, and the next time the agent encounters the situation, the skill loads instead of you re-prompting.

Skills are how organizational knowledge accrues into the agent over time.

## When to build a skill

Build a skill when:

- **The task pattern repeats.** "Add a feature flag", "Write a migration", "Cut a release", "Triage an alert" — these are skill-shaped.
- **The right approach is non-obvious.** A junior engineer would do it wrong; the right way takes context. A skill captures the context.
- **Quality is uneven without explicit guidance.** The model produces inconsistent output across runs. A skill normalizes.
- **There's a checklist or sequence that matters.** Steps must happen in order, or one step is easy to forget.

Don't build a skill when:

- **The task is one-off.** A skill that runs once isn't worth packaging.
- **The model already does it well consistently.** No skill needed; the model has it.
- **It's just a tool.** If the work is "call this API with these args," that's a tool, not a skill.

## How skills get discovered

The challenge with skills is that the model only uses them if it knows about them. Two approaches:

**Pre-loaded.** All available skills are listed in the system prompt with names and descriptions. The model picks based on the descriptions. Works well for small skill libraries (< 20). Breaks down at scale (the prompt gets enormous and the model can't keep them straight).

**On-demand discovery.** A `list_skills` or `find_skill` tool lets the model search the skill library when it senses it needs help. Works at scale but requires the model to think to look.

In practice, most systems use a hybrid: a few highly-relevant skills are pre-loaded based on context (project type, current task), and the rest are discoverable via search.

## Naming and description discipline

A skill's name and description are how the model decides whether to use it. Same discipline as tool descriptions:

- **Verb + object in the name.** `triage-production-incident`, `write-release-notes`, `add-feature-flag`.
- **Be specific in the description.** When does this apply? What does it produce? What does it assume?
- **State assumptions explicitly.** "Assumes you have a recent deploy and access to the monitoring dashboard."
- **Include trigger signals.** "Use this when the user mentions an incident, an outage, or a production alert."

A poorly-described skill is a skill that won't get used, even when it should.

## Skill lifecycle

Skills should be:

- **Versioned.** As techniques change, skills evolve. Track changes.
- **Owned.** Someone is responsible for keeping each skill current.
- **Tested.** A skill that produces wrong output is worse than no skill.
- **Reviewed.** When a skill is added or significantly changed, another set of eyes catches the fragile parts.
- **Pruned.** Skills that nobody uses, or that conflict with newer skills, should be removed.

A skill library that grows monotonically becomes noise. A library that's curated stays useful.

## Personal vs team skills

Two scopes:

**Personal skills** are how an individual codifies their working style. "How I write commits", "How I format code reviews", "How I structure a brainstorming session." Small, opinionated, optimized for one person's workflow.

**Team skills** are shared agreements about how work happens. "How we structure incident reviews", "How we cut releases", "How we onboard a new service." Reviewed, versioned, applied consistently across team members.

Both are valuable. Personal skills compound individual leverage. Team skills compound organizational leverage. Build the personal ones first to learn the form; build team ones once you know what works.

## Skills as institutional memory

The most underrated property of skills: they preserve knowledge.

When a senior engineer figures out the right way to do something hard and writes it down as a skill, that knowledge is now available to every agent run by every team member. Nobody else has to re-derive it. Onboarding gets shorter. Best practices get applied consistently. The bus-factor problem softens.

This is the long-term promise of skills: not faster individual tasks, but a slow-and-steady accumulation of "the way we do things" that the team's agents can act on. A team that takes skill curation seriously builds a compounding asset. A team that doesn't keeps re-explaining the same things forever.

## Where skills meet specs

Specs (see [03-multi-agent-workflows/spec-driven.md](../03-multi-agent-workflows/spec-driven.md)) describe what to build for a particular task. Skills describe how to do recurring kinds of work.

A spec is per-task. A skill is per-pattern. A spec might invoke a skill ("write the release notes following the team release-notes skill"). The skill provides the *how*; the spec provides the *what*.

Both are forms of writing things down so agents (and people) can pick up the work cleanly. Both compound in value the more rigorously you treat them.
