# Model Selection

Picking which local model for which task. The right answer changes faster than this document can keep up with, but the framework is durable.

## The size/capability tradeoff

Larger models are more capable. They're also slower, hotter, and more memory-hungry. The sweet spot depends on your hardware and your task.

Rough size buckets and what they're typically useful for:

**Tiny (1B–3B parameters).** Fast, fits anywhere. Useful for: classification, simple routing, basic summarization, autocomplete-style work. Not useful for: anything requiring reasoning over more than a sentence or two.

**Small (7B–14B).** The default for most developer machines. Useful for: simple coding tasks, Q&A over a small context, basic code review. Acceptable for: more complex tasks if you're patient and your prompt is good.

**Mid (30B–70B).** A real workhorse if your hardware can run it. Useful for: most agentic tasks, multi-step reasoning, code review, planning. Approaches frontier-class on many tasks; falls behind on the hardest.

**Large (100B+).** Approaches frontier capability on many tasks. Useful for: anything you'd otherwise use a frontier API for. Slow on consumer hardware; needs server-class GPUs to be responsive.

## Model families to know about

The major open-weight families update frequently. Rather than name specific models (which will be stale), the families that consistently lead in code:

- **Llama family.** Long-running open-weight series, generally strong at instruction following and coding.
- **Mistral / Mixtral family.** Often strong per-parameter efficiency. MoE variants are useful for large effective capacity at lower active cost.
- **Qwen family.** Often the strongest open coding models at any given moment. Worth tracking.
- **DeepSeek family.** Strong reasoning. The "thinking" variants do extended reasoning on hard problems.
- **Code-specialized variants** (CodeLlama, DeepSeek-Coder, Qwen-Coder, etc.). Often the best choice for pure code work; sometimes weaker on general reasoning than the general model in the same family.

When picking, check current benchmarks for code (HumanEval, SWE-Bench, LiveCodeBench, MBPP). Don't trust general leaderboards for code-specific tasks; the rankings differ.

## Quantization

Models can be run at lower precision than they were trained at. Common quantizations:

- **FP16 / BF16.** Original precision. Highest quality, most memory.
- **8-bit.** Roughly half the memory, near-original quality.
- **4-bit (Q4 in GGUF).** Roughly a quarter of the memory, noticeable but tolerable quality loss for most uses.
- **2-bit / 3-bit.** Aggressive. Quality often drops sharply. Useful only if you need the size budget.

Practical guidance: start at 4-bit (Q4_K_M is a common high-quality variant). If you have memory headroom, try 8-bit and see if quality matters for your tasks. Don't go below 4-bit unless you've measured.

## Context length

Different models have different max context lengths. Don't trust the advertised number without checking quality at long context:

- Many local models advertise 128K but degrade noticeably past 32K
- Frontier models hold quality better at long context, generally
- Long context costs memory; full 128K context can require multiples of model weight size

For most coding tasks, 16K-32K of working context is sufficient if you're targeted about what you put in. See [06-token-efficiency/context-management.md](../06-token-efficiency/context-management.md).

## Specialized models

For some tasks, a smaller specialized model beats a larger general one:

- **Embedding models** for retrieval. Don't use a general LLM to generate embeddings; use a dedicated embedding model.
- **Code-specific models** for pure code tasks. Often beat general models on code generation at the same parameter count.
- **Instruct vs. base.** Use instruct/chat variants for any agent work. Base models predict text continuation; they don't reliably follow instructions.

## Picking for your hardware

**Modest dev machine (16-32 GB RAM, no dedicated GPU or modest GPU).** Models in the 3B-7B range, 4-bit. Good for routing decisions, classification, basic completions. Not strong enough for complex agent work.

**Strong dev machine (64+ GB RAM or 24+ GB VRAM).** Models in the 13B-30B range. Useful for moderate agent work. Can handle 30B at 4-bit comfortably; 70B is borderline.

**High-memory workstation or M-series Mac with 64-128 GB unified memory.** 70B models at 4-bit run usefully. Approach frontier-class on many tasks. Slow but viable.

**Server with multiple GPUs.** Anything. The constraint becomes power, cost, and cooling rather than capability.

## Picking for your task

**Code completion / inline.** Small specialized code model. 7B Code* class. Speed matters more than reasoning depth.

**Inline chat ("explain this function").** Small-to-mid general model. 13B-class. Needs to follow instructions; doesn't need deep reasoning.

**Agent tasks (read code, plan, edit).** Mid-to-large model. 30B-70B. The harder the task, the more this matters.

**Complex reasoning, planning, critique.** Large local model or frontier API. The capability gap shows up most here.

**High-volume routine work (classification, routing, simple summaries).** Small model. The economics flip immediately at volume.

## Don't over-rotate on benchmarks

Benchmark leaderboards move every week. Don't pick a model because it's at the top of HumanEval today. Pick because:
- It runs well on your hardware
- It works on your actual tasks (test it on real work, not benchmarks)
- It's well-supported in your runtime and harness

A model that's #5 on the leaderboard but works smoothly in your setup beats a #1 model that has runtime quirks.

## Update cadence

Local model quality has been improving roughly quarterly. Any specific model recommendation will be obsolete within 6-12 months. Practice:

- Re-evaluate quarterly. Pull a few current models, run them against your real tasks, see if they're better than what you have.
- Don't churn for small improvements. Switching models has setup cost. A 5% improvement isn't worth it.
- Do switch for large improvements. A 30% improvement on tasks you do daily is worth a day of setup.

## What to measure

Generic benchmarks tell you something. Your specific tasks tell you more. Build a small evaluation harness with your actual work:

- 20-50 representative prompts from your real use
- Known-good answers (or at least known-bad answers to avoid)
- A way to compare two models side-by-side on the same prompt

Run any candidate model through this. The whole exercise is an hour. You learn more than from any leaderboard.

See [benchmarking.md](./benchmarking.md) for patterns.
