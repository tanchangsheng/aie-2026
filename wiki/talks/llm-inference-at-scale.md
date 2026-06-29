---
type: talk
tags: [inference, gpu, kv-cache, attention, vllm, sglang, quantization, memory, workshop]
updated: 2026-06-29
---

# LLM Inference at Scale — With First Principles

**Official title:** 2 hr deep dive on LLM Inference at Scale (Parts 1 & 2)  
**Speakers:** [Harshul Jain](../speakers/harshul-jain.md) · [Tanmay Sah](../speakers/tanmay-sah.md)  
**Day:** Day 1 — Workshop Day  
**Time:** 12:10pm–2:15pm (Parts 1 & 2, Track 3)  
**Room:** Track 3  
**Track:** Workshops Day 1  
**Format:** 60% explanation + live demos, 40% hands-on exercises. Compute sponsored by Coreweave/Marimo.

---

## Official Description

Most engineers using LLMs can call an API. Far fewer can explain why their model is slow, why it's running out of memory, or how the inference engines powering every major LLM API actually work. This workshop walks through the full inference stack — from how a transformer generates a single token to serving billions of tokens a day with vLLM, SGLang, TensorRT-LLM, Ray, and KServe/llm-d.

Repo: [github.com/harshuljain13/llm-inference-at-scale](https://github.com/harshuljain13/llm-inference-at-scale) — 55+ modules, 12 chapters, 8 runnable demo notebooks.

---

## Key Claims

- **Inference cost now dwarfs training cost** — GPT-3 training was $4.6M one-time; serving now exceeds $4.6M/week and growing.
- **One memory equation governs everything**: `GPU memory = weights + KV cache + overhead`. For Mistral-7B on A100-80: weights (14.5 GB) + overhead (~8 GB) leaves ~57.5 GB for KV cache → 107 max concurrent users at 4K context without any optimizations.
- **Decode is memory-bandwidth bound, not compute-bound.** Every decode step streams 14.5 GB from HBM at 2 TB/s → theoretical ceiling of ~138 tok/s. The roofline is ~2 FLOP/byte, vs 156 for compute saturation. You cannot buy your way out — you raise arithmetic intensity via batching.
- **KV cache is the variable that kills you.** At 131 KB/token/user for Mistral-7B (GQA-8), 80 users × 16K context = 168 GB — OOM on any single A100.
- **Four orthogonal levers compound to ~20×**: GQA (4×) + INT4 (4×) + PagedAttention (4×) + Continuous Batching (10×) + Prefix Caching (20× TTFT) + Speculative Decoding — stacked on same hardware.
- **MLA (DeepSeek-V3) is the Pareto winner in attention design**: 56× less KV memory than MHA with near-identical quality.
- **No universal engine winner** — vLLM for general API serving, SGLang for agentic/shared-prefix workloads (5× cache hit rate), TensorRT-LLM for max NVIDIA throughput (2× over vLLM after compile).

---

## The Core Equation

```
KV per token (Mistral-7B) = 2 × 32 layers × 8 KV heads × 128 dim × 2 bytes = 131 KB
max_users = (80 - 14.5 - 8) GB ÷ (0.131 MB × 4096 tokens) = 107
```

---

## The Compounding Stack

| Optimization | Cumulative Multiplier |
|---|---|
| Baseline | 1× |
| + GQA | 4× |
| + INT4 quantization | 7× |
| + PagedAttention | 10× |
| + Continuous Batching | 13× |
| + Prefix Caching | 16× |
| + Speculative Decoding | ~20× |

---

## Engine Benchmark (Qwen2.5-7B · H100 · 20 concurrent users)

**Standard API:** vLLM (29.79 req/s) ≈ SGLang (29.03 req/s) — near parity, vLLM slightly ahead.  
**Agentic shared-prefix (~1K-token prefix):** SGLang (1.07 s TTFT) vs vLLM default (6.30 s) — **5.8× faster**.

---

## Notable Quotes

> "You will not memorize these. You will learn the foundations that let you evaluate whatever ships next."

> "Training is a one-time capital cost. Inference is an operating cost that scales with every user, every token, every session."

> "The most expensive GPU per hour is the cheapest per token."

> "No new hardware. Each lever multiplies the last — that's the whole game."

---

## Reactions

A genuinely first-principles workshop — rare to see the KV cost derived from model config and then proven empirically in the same session. The compounding stack framing (each lever multiplies the last) is the clearest way I've seen the optimization landscape explained. The vLLM vs SGLang benchmark result on agentic workloads (5.8×) is striking — most teams probably default to vLLM even for agent use cases.

The MLA / DeepSeek angle is underappreciated: a 56× KV reduction at near-MHA quality is what makes DeepSeek cheap to serve, not just clever training.

Open question: how does the compounding interact across levers in practice — are they truly multiplicative or do some interfere?

---

## Linked Concepts

- [KV Cache](../concepts/kv-cache.md)
- [Attention Mechanisms](../concepts/attention-mechanisms.md)
- [Inference Engines](../concepts/inference-engines.md)
- [Memory Bandwidth & the Roofline](../concepts/memory-bandwidth-roofline.md)
- [Quantization](../concepts/quantization.md)
