# Benchmarking

How to evaluate whether a model is good enough for your actual work. Public benchmarks are useful starting points; they don't tell you what matters for your tasks.

## Why your own benchmark matters

Public benchmarks (HumanEval, MMLU, SWE-Bench, etc.) measure generic capabilities. They tell you that model A is generally smarter than model B. They don't tell you:

- Whether model A is better at your codebase's idioms
- Whether model A handles your project's typical task sizes
- Whether model A's failures correlate with your team's blind spots
- Whether model A is worth the speed/cost difference for your use

The gap between "scored 85% on HumanEval" and "useful for shipping code" is large. Bridging it requires measuring against your work.

## What a useful eval looks like

A small, opinionated test suite that captures the kinds of work you actually do.

**Size.** 20-100 prompts. More is overkill for ad-hoc evaluation; fewer doesn't cover enough ground.

**Source.** Real prompts from real work, lightly anonymized. Made-up benchmark prompts are biased toward what you can easily think to test, not toward what actually shows up.

**Coverage.** Mix of:
- Easy tasks (catch regressions; ensure model can do basics)
- Medium tasks (the bulk of your work)
- Hard tasks (where capability differences matter)
- Edge cases (your codebase's specific quirks)

**Outputs.** Either:
- **Reference answer.** "The correct answer is X; how close did the model get?"
- **Quality criteria.** "A good answer has properties P1, P2, P3."
- **A/B preference.** "Run two models, pick which output is better."

A/B is often the most practical. Reference answers are time-consuming to write; quality criteria are subjective. Comparative evaluation against a known model is fast and reveals meaningful differences.

## Running the eval

The simplest workflow:

1. **Capture.** Save 20+ prompts from real work to `evals/prompts/`. Include the context that was needed (relevant files, conventions).
2. **Run.** Script that runs each prompt against each candidate model, saves outputs to `evals/runs/<model>/<prompt-id>.txt`.
3. **Compare.** Read the outputs side by side. Mark which is better, equal, or which has specific issues.
4. **Aggregate.** "Model A is better on 14/20, worse on 3, equal on 3."

A simple shell script for the run step:
```bash
for prompt in evals/prompts/*.md; do
  id=$(basename "$prompt" .md)
  for model in llama-3.3-70b qwen-2.5-coder-32b deepseek-v3; do
    mkdir -p "evals/runs/$model"
    if [ ! -f "evals/runs/$model/$id.txt" ]; then
      run-model "$model" --input "$prompt" > "evals/runs/$model/$id.txt"
    fi
  done
done
```

## What to measure

Beyond "which output is better," useful measurements:

- **Time per prompt.** Fast model that's slightly worse can win on throughput-heavy tasks.
- **Memory used.** Constraint for your hardware.
- **Failure rate.** How often does the model get stuck, refuse, or output garbage?
- **Tool-use quality.** If your real work involves agent loops with tools, eval that — not just chat completions.
- **Instruction-following.** Does it actually do what the prompt asked, or drift?
- **Convention adherence.** Does the output match your codebase's style?

Don't try to capture everything in one number. A scorecard with 4-6 axes gives you more decision-making information than a single weighted average.

## Eval that matches your real workflow

The biggest pitfall: evaluating in single-shot when you actually use the model in agent loops.

A model that produces nice single-shot completions might do badly in a tool-using agent loop because:
- It's bad at following agent harness conventions
- It hallucinates tool calls
- It doesn't recover well from tool errors
- Its style of output confuses the harness's parsers

If your real use is agent loops, evaluate in agent loops. Set up a small scenario ("here's a repo, here's a task spec, run the agent, did it produce a useful diff?") and measure that. This is more work than chat-style eval but more representative.

## Eval over time

Models update. Re-run your eval suite when:
- You're considering switching models
- A new model in a family you use is released
- Your work shifts (new project, new domain) — your eval suite should evolve with it

Keep historical results. "Model X scored 16/20 on our eval six months ago, the new version scores 18/20" is much more meaningful than the absolute scores.

## Avoiding eval rot

Evals decay if you don't curate them.

- **Add prompts as you encounter new task types.** Your eval suite should grow as your work evolves.
- **Retire prompts that all models trivially solve.** They're not differentiating signal anymore.
- **Update reference answers** when conventions change.
- **Don't include prompts you can't write a good answer to.** If you can't tell what good looks like, you can't evaluate.

## Bias in your own eval

Your eval is biased toward what you've thought to test. That's both feature and bug:
- Feature: it reflects your real work.
- Bug: it misses surprises. The thing you didn't think to test is the thing the model fails on in production.

Mitigations:
- Use the model on real work between formal eval runs. Note surprises. Add them to the suite.
- Periodically have someone else look at your eval and ask "what's missing?"
- Cross-reference public benchmarks for capabilities your eval doesn't cover. If a model crashes on long-context tasks but your eval is all short prompts, the public benchmark caught what you missed.

## When public benchmarks are useful

- **Initial filter.** Quickly cut a long list of candidates to a short list. Don't bother evaluating models that are obvious nonstarters.
- **Capability ceilings.** "This benchmark is hard; if a model can't do it, it can't do anything harder." Useful for filtering out underpowered models for hard tasks.
- **Trend tracking.** Over time, "is the local-model leaderboard catching up to frontier?" tells you when local viability is changing.

When public benchmarks are misleading:
- They often test in ways that don't match real use (single-shot, no tool use, English only, well-defined problems)
- They're a target, so models get over-fitted to them
- They don't capture taste, style, or codebase-specific quality

Trust them for "is this model in the right ballpark?" Don't trust them for "is this model right for me?"
