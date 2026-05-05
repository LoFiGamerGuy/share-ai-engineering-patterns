# Local LLMs Overview

Running models on your own hardware instead of paying per-token to a provider. The capability gap with frontier models is real but shrinking. The question isn't "are local models good enough" in general — it's "are local models good enough for *this specific task*."

## Why local

**Privacy.** Code, data, and prompts never leave your environment. For sensitive work — proprietary code, regulated data, confidential research — this is the dominant reason.

**Cost predictability.** A frontier API can cost $0 or $500 in a given month depending on use. A local model has fixed hardware cost and zero per-token cost. For high-volume use, the math flips quickly.

**Independence.** No external dependency. The model works offline, on a plane, behind a firewall, when the provider is down.

**Latency.** First token can be lower than API calls (no network round-trip). For tasks where you're calling the model many times in a tight loop, this adds up.

**Customization.** You can fine-tune. You can swap models. You can run multiple models locally and route between them. None of this is possible with closed APIs.

## Why not local

**Capability ceiling.** The best frontier models are still meaningfully better than the best local models on hard tasks. The gap shrinks every quarter but is real today.

**Hardware investment.** A useful local setup is a GPU box ($2-10k typical) or a top-end Mac with unified memory ($4-7k). For an individual that's a real expense; for a team it's small.

**Operational overhead.** Running models requires maintenance, updates, occasional debugging. APIs are someone else's problem.

**Speed for hard tasks.** Frontier models often think faster on hard problems because their hardware is better. A local 70B model can be slower than a frontier API call for nontrivial reasoning.

**Long context.** Some local models support 128K or more, but performance often degrades faster as context grows than frontier models.

## When local is the right choice

- **You handle sensitive code or data.** Anything where leaving your environment is a problem.
- **You run very high volumes.** Burning $500/month on agent loops? Local quickly becomes cheaper.
- **You have specific tasks where local is good enough.** Code completion, classification, routing decisions, summarization, basic Q&A. Frontier-level reasoning isn't needed for these.
- **You want to experiment with fine-tuning.** Provider APIs limit how much you can customize.

## When frontier is the right choice

- **The task is hard.** Multi-step reasoning, novel architecture decisions, code review of intricate diffs. Frontier wins.
- **You don't want operational responsibility.** API key + payment method = working AI. No infrastructure.
- **Use is sporadic.** Hard to justify hardware that sits idle 23 hours a day.
- **Ecosystem integration.** Frontier providers have deeper tool ecosystems, more harness support, more pre-built MCP servers.

## The hybrid pattern

Most teams operating at scale don't pick. They route:

- **Cheap local model** for high-volume routine work (autocomplete, classification, simple summaries)
- **Mid-tier API** for normal coding tasks
- **Frontier API** for hard tasks (planning, critique, novel problems)

This is **model tiering** ([06-token-efficiency/model-tiering.md](../06-token-efficiency/model-tiering.md)). Local models are one tier in the stack, not a replacement for the stack.

## What's in this section

- [model-selection.md](./model-selection.md) — picking which local model for which task. Sizes, families, and what they're actually good for.
- [benchmarking.md](./benchmarking.md) — how to evaluate whether a model is good enough for your actual work, not generic benchmarks.

## What's not in this section

- Hardware purchase advice. Moves too fast for durable guidance.
- Specific model names. Same. By the time you read this, the leaderboard has moved.
- Fine-tuning. A whole topic on its own. The patterns here assume off-the-shelf models.

## A practical starting point

If you've never run a local model and want to:
1. Pick a runtime (Ollama, llama.cpp, vLLM, LM Studio — any will do)
2. Pull a current mid-size open-weight model (something in the 7B-30B range)
3. Wire it into your editor or agent harness as one of your model options
4. Use it for a week on real tasks; see what it's good and bad at

This is a few hours of setup. The reflection at the end of the week — "I used local for X tasks, used frontier for Y tasks, and felt the difference at these moments" — is the actual value. Generic benchmarks don't tell you what's relevant for your work.
