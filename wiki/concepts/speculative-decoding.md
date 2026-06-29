---
type: concept
tags: [inference, decode, speculative-decoding, vllm, latency, throughput, optimization]
updated: 2026-06-29
---

# Speculative Decoding

## Definition

Speculative decoding uses a cheap **proposer** (drafter) to generate candidate tokens, then lets the expensive **target** model verify them in one parallel pass. Accepted tokens advance the sequence by more than one per target-model step, raising effective throughput without changing the target model's output distribution.

## How It Works

1. The drafter generates N candidate tokens sequentially (cheap).
2. The target model verifies all N in one parallel forward pass.
3. The server accepts the longest prefix the target agrees with; rejects the rest.
4. The target's output distribution is preserved — the drafter cannot unilaterally answer.

**The key metric:** accepted tokens per target step. If the drafter proposes 4 and the target accepts 3.5 on average, the server moves forward 3.5 tokens for the cost of one target step plus one (cheap) draft pass.

Speedup ≈ `normal_step_time / (verify_step_time / accepted_per_step)`

## Drafter Method Landscape

| Method | Proposer | Best fit | Tradeoff |
|---|---|---|---|
| Draft model | Smaller autoregressive model | Easy mental model; family-aligned drafter | Extra model memory; acceptance depends on alignment |
| N-gram / suffix lookup | Repeated token sequences from history | Repetitive prompts, code, templates | No extra model; modest gain |
| Native MTP | Extra model checkpoints | Models trained with multi-token prediction | Requires model support |
| Medusa | Extra decoding heads on the target | When Medusa heads are available | Needs trained heads + tree verification |
| EAGLE / EAGLE-3 | Predicts future hidden features | Strong general-purpose; needs matching weights | Needs compatible EAGLE weights |
| DFlash | Block diffusion drafter | Emerging parallel drafting path | Newer stack |

## When It Helps vs. Hurts

**Helps:**
- Low-to-medium concurrency (GPU not already saturated)
- Predictable output distributions: code completion, templates, repeated formats
- Drafter trained for or from the same model family as the target

**Hurts or does nothing:**
- High concurrency — extra draft work competes with normal batching
- Low acceptance rate — drafter family/training mismatches the target
- Speculation depth too high — later positions are rejected more often, wasted work
- Single-GPU contention — drafter shares VRAM and compute with target

## Published Benchmark Reference

- vLLM speculative decoding benchmark: up to **1.5×** speedup (draft model, ShareGPT); up to **2.8×** (n-gram, summarisation) — but reports slowdowns at higher QPS.
- EAGLE-3 on vLLM: up to **2.5×** improvement when speculators are trained for the specific target.
- Medusa: **2.2–3.6×** from extra decoding heads.

## vLLM Configuration

```yaml
- '--speculative-config={"method":"draft_model","model":"RedHatAI/Qwen3-0.6B-FP8-dynamic","num_speculative_tokens":4,"draft_tensor_parallel_size":1}'
```

`num_speculative_tokens` is the primary tuning knob. A good workshop starting point is 4. Values to try: 2, 4, 6. Stop when throughput stops improving or TTFT rises.

## Relationship to Other Concepts

- Speculative decoding addresses the **sequential** bottleneck of decode (each step requires one target pass). It's orthogonal to quantization (which reduces bytes per step) and batching (which amortizes the step across users).
- The draft model consumes VRAM that could be KV cache — a direct tradeoff with [KV Cache](kv-cache.md) capacity.
- A negative result is a valid production decision; measure acceptance and throughput before keeping the draft path.

## Open Questions

- Under what prompt distributions does a 0.6B drafter consistently achieve >70% acceptance with a 4B target?
- How does speculative decoding interact with prefix caching — does the draft model benefit from the same prefix cache?
- Does EAGLE-3 outperform draft-model speculation for Qwen3 specifically?

## Sources

- [Agents That Own Their Inference Workshop](../talks/agents-own-inference.md) — Du'an Lightfoot / Omer Aslan, Module 6
