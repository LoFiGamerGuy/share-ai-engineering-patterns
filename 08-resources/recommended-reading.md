# Recommended Reading

A curated list of books that inform good agentic AI engineering work. AI is the primary focus, but the list extends to the adjacent categories that genuinely shape how you do this work well — psychology of decision-making, software craft, engineering leadership, writing.

The list is opinionated. Books that make the cut have one of three properties: they teach a durable mental model, they cover ground that's hard to assemble from blog posts, or they were genuinely useful when applied to real work. Books that read well but didn't survive contact with implementation are not here.

**A note on publisher diversity:** earlier versions of this list leaned heavily on one technical publisher. That was a curation failure, not a reading reality. The revised list spans many publishers because the field's important books span many publishers. If a book deserves to be here, it's here regardless of who publishes it.

**A note on format:** this list is **books only**. White papers, blog posts, conference talks, and academic publications are valuable but ephemeral and out of scope for a curated reading list. The discipline is "what 200-400 page treatment of this topic should be on your shelf?"

---

## Two books that should anchor this list

If you read nothing else from this list, read these two.

**Co-Intelligence: Living and Working with AI** — Ethan Mollick (Portfolio / Penguin, 2024)

Mollick is a Wharton professor who's been writing about AI's actual effect on knowledge work since GPT-3.5. *Co-Intelligence* distills three years of his research and experimentation into a working philosophy: how to think about AI as a collaborator, not a tool; the four rules ("always invite AI to the table," "be the human in the loop," "treat it like a person but tell it what kind of person it is," "assume this is the worst AI you'll ever use"); the jagged frontier; the centaur model. The single most useful framing for engineers building with AI in 2026.
**Read for:** the operating philosophy. Everything else on this list is downstream of getting this framing right.

**Thinking, Fast and Slow** — Daniel Kahneman (Farrar, Straus and Giroux, 2011)

The foundational text on how human reasoning actually works — System 1 (fast, intuitive, error-prone) vs System 2 (slow, deliberate, expensive). Reading it changes how you understand what LLMs are doing (mostly System 1) and what humans bring to the loop (System 2). Also: the source for many of the anti-groupthink mechanisms used in [council-of-five](https://github.com/LoFiGamerGuy/council-of-five).
**Read for:** the cognitive vocabulary. Everyone working with AI is doing applied cognitive science whether they know it or not.

---

## How to read this list

Most engineers read books wrong for technical work:

1. **Read the table of contents and the conclusion first.** Decide whether the book is worth the time investment.
2. **Skim the first chapter end-to-end** to calibrate the author's voice and depth.
3. **Read selectively.** Most technical books have 2-3 chapters that earn the price; the rest is filler. Find the load-bearing chapters and read them carefully.
4. **Distill as you read.** A 300-page book is worth a 2-page distillation; a 2-page distillation is worth more than the book sitting on a shelf.

The "Distillation" notes below come from this practice — pre-extracted insights from each book, focused on what was non-obvious or load-bearing.

---

## For [01 Concepts](../01-concepts/) — agents, context, MCP, trust

**Co-Intelligence** — Ethan Mollick (Portfolio, 2024) *(also featured above)*
The operating philosophy. Read first.

**Artificial Intelligence: A Guide for Thinking Humans** — Melanie Mitchell (Farrar, Straus and Giroux, 2019)
A computer-scientist-turned-Santa-Fe-Institute-faculty's accessible-but-rigorous tour of what AI actually is, what it isn't, and what the open questions are. Older than the LLM era but the conceptual foundations are the same; the gaps in current AI capability she identified in 2019 are mostly still gaps.
**Read for:** the calibration on what AI is good at vs. terrible at. Counter-acts the field's worst hype reflex.

**Building Generative AI Agents** — Tom Taulli & Gaurav Deshmukh (Apress, 2025)
The single best book for the technical foundation of agent architecture. The "six components of an agent" framing (Reflection, Tools, Memory, Planning, Multi-agent, Autonomy) is the most useful taxonomy I've seen — and the book is explicit that you don't need all six. Memory taxonomy (episodic, semantic, procedural, entity, contextual) is finer-grained than most treatments. Framework comparison table (LangGraph, CrewAI, AutoGen, LangChain, Haystack) is useful before you commit to any of them.
**Read for:** the agent-component decomposition, memory subtypes, framework decision matrix.

**Hands-On Large Language Models** (2nd ed.) — Jay Alammar & Maarten Grootendorst (O'Reilly, 2024)
The "what's actually happening in the model" book. Strong on tokenization, embeddings, attention, and how the pieces compose. Worth reading even if you'll never train an LLM, because it grounds the abstractions you use every day.
**Read for:** the mental model of why long context, prompt order, and token budgets work the way they do.

---

## For [02 Platform](../02-platform/) — environment, scaffolding, the agentic spine

**A Philosophy of Software Design** — John Ousterhout (Yaknyam Press, 2nd ed. 2021)
The modern definitive treatment of software design as a craft. Deep modules, tactical vs strategic programming, the cost of complexity. Short (~190 pages), unusually self-aware, no jargon. Ousterhout (Stanford, formerly Sun) writes from decades of teaching the craft.
**Read for:** the framework for evaluating "is this design good?" Applies directly to how you structure agentic project scaffolding.

**The Pragmatic Programmer** (20th Anniversary ed.) — David Thomas & Andrew Hunt (Addison-Wesley, 2019)
The foundational text on software craft. Tracer bullets, DRY (the original framing), the broken windows theory, programming by coincidence, plain text as universal format. Twenty years on, almost everything still applies.
**Read for:** the working vocabulary of software craft. Translates directly to building good repos for agents to work in.

**Agentic Coding with Claude Code** — practitioner-focused (2025)
The most directly relevant book for how Claude Code specifically shapes the work. Subagent architecture, tool-use hierarchy, `CLAUDE.md` as procedural memory.
**Read for:** subagent architecture, when to escalate to a sub-agent vs handle inline, permission model patterns.

---

## For [03 Multi-agent workflows](../03-multi-agent-workflows/) — coordination, council, review

**Designing Data-Intensive Applications** — Martin Kleppmann (O'Reilly, 2017)
Not about AI; about the mental models for distributed systems that multi-agent workflows quietly depend on. Replication, consistency, consensus, the dynamics of what happens when multiple processes coordinate. Kleppmann's writing is unusually clear for a technical book at this level.
**Read for:** the distributed-systems vocabulary that explains why multi-agent coordination is hard.

**Building Applications with AI Agents — Designing and Implementing Multi-Agent Systems** — Vlad Rișcuția (O'Reilly, 2025)
The most thorough multi-agent book currently in print. Covers role specialization, message passing, shared memory, conflict resolution.
**Read for:** the design choices that go into a multi-agent system, common failure modes.

**The Worlds I See** — Fei-Fei Li (Macmillan, 2023)
Memoir of the founder of ImageNet (and one of the people most responsible for modern AI). The technical content is light; the value is the perspective on how multi-system AI work actually emerges over years. Useful counter to the "an agent is one thing" intuition.
**Read for:** the long-arc view on how AI systems emerge from many narrower components.

---

## For [04 Automation](../04-automation/) — autonomy levels, sandboxes, overnight runs

**The Developer's Playbook for Large Language Model Security** — Steve Wilson (O'Reilly, 2024)
Wilson co-founded the OWASP Top 10 for LLM Applications. The book is the operational reference for prompt injection, tool abuse, data exfiltration, the agency-as-attack-surface argument. Required reading before letting an agent run unattended on anything sensitive.
**Read for:** the attack surfaces you didn't think of. See [04-automation/prompt-injection.md](../04-automation/prompt-injection.md) for the distillation.

**Human Compatible: Artificial Intelligence and the Problem of Control** — Stuart Russell (Viking, 2019)
Russell co-wrote the standard AI textbook. *Human Compatible* is his argument for why we're building AI wrong (objective-maximizing) and what the alternative looks like (uncertainty-preserving). Less practical than other entries here but the framing applies directly to how you think about agent autonomy levels.
**Read for:** the structural argument for keeping humans in the loop at consequential decision points.

**The Alignment Problem** — Brian Christian (W. W. Norton, 2020)
The history-and-current-state of ML alignment, told as narrative. Three sections: representation (what the model sees), agency (what the model does), normativity (what the model should do). Not a how-to; an explanation of why alignment is structurally hard.
**Read for:** context on why your agent does the wrong thing in ways you didn't predict, and why anti-sycophancy bindings have to be structural rather than just polite asks.

---

## For [05 Local LLMs](../05-local-llms/) — model selection, benchmarking

**The Coming Wave** — Mustafa Suleyman (Crown, 2023)
Suleyman co-founded DeepMind and now runs Microsoft AI. His book argues that AI and synthetic biology represent a "wave" of technologies that will be impossible to contain via current institutions. Less directly applicable than others on this list but useful context for anyone deciding "should this work be done locally vs hosted?"
**Read for:** the broader frame on AI's trajectory, which informs hosting / privacy / capability tradeoffs.

**Hands-On Large Language Models** — Alammar & Grootendorst (above) is also primary here for the chapters on quantization and inference optimization.

---

## For [06 Token efficiency](../06-token-efficiency/) — caching, tiering, cost

**Prompt Engineering for Generative AI** — James Phoenix & Mike Taylor (O'Reilly, 2024)
The most practical book on prompt design. Strong on prompt structure (system, instructions, examples, output format) and how that structure interacts with caching. The non-obvious chapter: how prompt order affects model attention and therefore cost.
**Read for:** how to structure prompts so caching actually works.

**Algorithms to Live By** — Brian Christian & Tom Griffiths (Henry Holt, 2016)
Computer-science concepts as life advice — explore/exploit, optimal stopping, scheduling theory, caching. Counter-intuitively useful for thinking about token-budget allocation, model tiering decisions, when to compact context.
**Read for:** the formal frameworks for decision-making under uncertainty, applied via friendly examples.

---

## For [07 Tools and MCP](../07-tools-and-mcp/) — server design, hooks, observability

**Domain-Driven Design** — Eric Evans (Addison-Wesley, 2003)
The classic on aligning software design with the domain it serves. Bounded contexts, ubiquitous language, aggregates. Old, dense, still load-bearing for thinking about what a "tool" should be in MCP terms — a tool is the API surface of a bounded context, not a random function.
**Read for:** the discipline of carving the right boundaries. Tool design is domain modeling.

**Refactoring: Improving the Design of Existing Code** (2nd ed.) — Martin Fowler (Addison-Wesley, 2018)
The catalog of refactorings. Applies directly when you're tuning MCP server APIs after watching agents use them — most of the changes are recognizable refactorings (extract function, rename, split parameter, etc.).
**Read for:** the vocabulary for "what change to make to make the API better."

---

## Adjacent: psychology and decision-making

These don't map to a single section but inform every layer of agentic engineering work.

**Psychology of Money** — Morgan Housel (Harriman House, 2020)
Twenty short essays on why we make terrible decisions about money. Reads as a book about money; is actually a book about how psychology beats reasoning more often than we admit. Directly applicable: most cost decisions in AI engineering are psychological (loss aversion, anchoring, sunk cost) more than technical.
**Read for:** the framework for noticing when you're making decisions for psychological reasons rather than rational ones.

**Thinking, Fast and Slow** — Daniel Kahneman (Farrar, Straus and Giroux, 2011) *(also featured above)*
The cognitive vocabulary.

**Range: Why Generalists Triumph in a Specialized World** — David Epstein (Riverhead, 2019)
The argument that breadth often beats depth, particularly in domains that change rapidly. Direct application: agentic engineering is a "wicked" learning environment (rules change, feedback is delayed and noisy) — Epstein's framework explains why career-long specialists in pre-AI engineering may not be the best builders in the AI era.
**Read for:** the framework for when generalist thinking outperforms specialist thinking.

**The Black Swan** — Nassim Taleb (Random House, 2007)
Taleb's argument that history is shaped by improbable, high-impact events that are obvious only in retrospect. Applies to AI capability jumps, model releases, prompt-injection incidents — "rare" failures that compound to define the field.
**Read for:** the framework for designing systems that survive surprises rather than predicting them.

**Antifragile** — Nassim Taleb (Random House, 2012)
Sequel to Black Swan. Things that gain from disorder — bones, immune systems, well-designed systems. The framework applies directly to building agent infrastructure that gets stronger from incidents rather than weaker.
**Read for:** the design principle of building systems that benefit from stress.

---

## Adjacent: software craft

**Working Effectively with Legacy Code** — Michael Feathers (Prentice Hall, 2004)
Twenty years old, still the canonical reference for refactoring code without tests. Increasingly relevant as agents add code to legacy bases that lack the safety nets agents assume.
**Read for:** the discipline of changing code safely when you can't trust the existing tests.

**Software Engineering at Google** — Wright, Manshreck & Winters, eds. (O'Reilly, 2020)
Google's accumulated practices, written by the people who actually run the systems. Long but worth it for the chapters on testing, code review, dependency management, large-scale changes. The "deprecation" chapter alone justifies the book.
**Read for:** the operating model of large-scale engineering, applicable even at much smaller scale.

**Crafting Engineering Strategy** — Will Larson (2025, Stripe Press)
Larson on how to write engineering strategy documents that survive contact with the team. Directly applies when you're being asked to make platform-level decisions about AI tooling.
**Read for:** strategy as written artifact, decision documentation, navigating ambiguity at scale.

---

## Adjacent: engineering leadership

**An Elegant Puzzle: Systems of Engineering Management** — Will Larson (Stripe Press, 2019)
The first essential book Larson wrote. Patterns and case studies for engineering management. Strong on the "this isn't a people problem, it's a systems problem" framing.
**Read for:** the systems lens on management problems.

**Staff Engineer: Leadership Beyond the Management Track** — Will Larson (Stripe Press, 2021)
The complement to *An Elegant Puzzle*. For engineers who want to grow without becoming managers. Directly applicable for senior people who are now being asked to architect agent-driven workflows for their teams.
**Read for:** the framework for technical leadership without people-management overhead.

**The Manager's Path** — Camille Fournier (O'Reilly, 2017)
The transition from senior engineer to manager to director, told as a path with named stages. Particularly good on the early-manager pitfalls.
**Read for:** the realistic view of what each step of the engineering management ladder actually involves.

**The Making of a Manager** — Julie Zhuo (Portfolio, 2019)
Zhuo (formerly VP of Design at Facebook) writes about the first months and years of being a manager. Different angle than Fournier; pairs well.
**Read for:** the human texture of becoming a manager.

---

## Adjacent: writing and communication

The agentic AI work in this repo depends heavily on writing — specs, CLAUDE.md files, prompt design, decision documentation. These books are the foundation.

**On Writing Well** — William Zinsser (Harper Perennial, 30th anniversary ed. 2006)
The classic on nonfiction writing. Short sentences, clear nouns and verbs, ruthless trimming. The first two chapters alone change how you write technical docs.
**Read for:** the discipline of clear prose. Directly applies to every CLAUDE.md and spec you write.

**The Sense of Style** — Steven Pinker (Viking, 2014)
Pinker's argument for "classic style" — writing as conversation between equals about something both parties can see. Less prescriptive than Zinsser; more theoretical. Pairs well.
**Read for:** the cognitive theory of why some prose lands and other prose doesn't.

---

## Bonus: shell, devops, and the supporting stack

Foundational for the dev environment that hosts agentic work. See [`08-resources/shell-and-terminal-tools.md`](./shell-and-terminal-tools.md) for the operational picks; the books here are the why-it-works reading.

**The Linux Command Line** — William Shotts (No Starch Press, 2nd ed. 2019)
The free PDF version is the most-downloaded technical book on the internet. Comprehensive, well-written, opinionated. The right introduction to bash for someone who's been faking it.
**Read for:** the comprehensive shell foundation, with discipline beyond what most users develop on their own.

**UNIX and Linux System Administration Handbook** — Nemeth, Snyder, Hein, Whaley & Mackin (Addison-Wesley, 5th ed. 2017)
The encyclopedic reference. ~1500 pages. Don't read cover to cover; mine when you need to operate Linux at depth (not just use it).
**Read for:** the deep reference when you're doing something operational the man pages don't quite cover.

**The Bash Guide for Beginners** by Machtelt Garrels — free online; pair with **Advanced Bash-Scripting Guide** by Mendel Cooper. Both are aging but the bash language hasn't moved much. (Note: these are book-length references, even though distributed online.)

---

## A note on the "agentic AI" book glut

There are dozens of books with "Agentic AI" in the title published in the last 18 months. The quality varies wildly. The picks above survived a winnowing process that started with ~50 candidates.

Common failure modes in the rejects:
- Repackaged blog content with no synthesis
- Code examples that don't run or are wildly outdated
- "Agentic" used as branding for what is actually plain LLM API usage
- Excessive enterprise framing with no implementation details
- Books that confidently predict the future of an industry that's three months old
- LLM-generated text that the author barely edited

If a book makes claims that "this will define the next decade," put it down. The honest books admit the field is moving too fast for that.

---

## What this list isn't

- **Not white papers, articles, or blog posts.** Those are valuable but ephemeral; this list is books with shelf life.
- **Not exhaustive.** Many good books didn't make the cut. The bar is "would I tell a friend to read this if they asked."
- **Not balanced across publishers as an end in itself.** The publisher mix happens to be diverse because the field's important books happen to be diverse.
- **Not static.** As books shift in usefulness or new picks earn their place, the list will too. See [CHANGELOG](../CHANGELOG.md) for revisions.

---

*Snapshot: May 2026. Reading-list picks evolve. Re-evaluate annually.*
