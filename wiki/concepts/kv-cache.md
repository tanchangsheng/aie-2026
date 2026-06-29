---
type: concept
tags: [inference, memory, optimization, kv-cache, context-engineering, caching]
updated: 2026-06-29 (3 sources)
---

# KV Cache

## Definition

The KV (Key-Value) cache stores the attention keys and values computed for each past token so they don't have to be recomputed on every decode step. Without it, inference is O(n²) in context length; with it, O(n).

Each new token must read back the K and V of every previous token. Computing them once and reusing them is the KV cache.

## Cost

For Mistral-7B with GQA-8:
```
2 (K+V) × 32 layers × 8 KV heads × 128 dim × 2 bytes = 131 KB per token per user
```

| Scenario | Memory |
|----------|--------|
| 1 user · 4K context | 524 MB |
| 1 user · 16K context | 2.1 GB |
| 80 users · 4K context | 42 GB |
| 80 users · 16K context | 168 GB — OOM |

## Why It's the Bottleneck

Weights are fixed (14.5 GB for Mistral-7B). The KV cache is the variable component that determines how many concurrent users fit in GPU memory. The capacity equation:

```
max_users = (GPU_VRAM - weights - overhead) ÷ (KV_per_token × context_length)
           = (80 - 14.5 - 8) GB ÷ (0.131 MB × 4096) = 107 users on A100-80
```

## The Four Levers

| Lever | Technique | Gain |
|-------|-----------|------|
| 01 | PagedAttention | ~4× users — eliminates fragmentation |
| 02 | Continuous Batching | ~10× context reclaim — no idle slots |
| 03 | Prefix Caching | ~20× TTFT — reuse shared context |
| 04 | KV Quantization (FP8) | ~2× users — < 0.1% quality loss |

These are orthogonal and stack multiplicatively.

## Application-Level Caching (Prompt Caching)

Beyond GPU memory, KV caching also applies at the API/application layer as **prompt caching** — providers cache the KV state of a stable prompt prefix so repeated calls don't recompute it.

Key findings from the context engineering workshop:
- Gemini 2.5+: ~90% cost reduction on cached tokens
- DeepSeek: ~50× reduction ($0.14 → $0.0028/M tokens)
- Anthropic: explicit `cache_control` pinning, with TTL + storage rent

**The cache coherence trap:** summarization rewrites the prompt prefix, invalidating the cache. In Towards AI's tutor experiment, the production compaction preset sent ~42% fewer tokens than full_history but paid ~2× more — because full_history billed ~87% of input at the cache discount while production got mostly cache misses.

> "If I had to choose just one metric, I'd argue that the KV-cache hit rate is the single most important metric for a production-stage AI agent." — Manus AI

## Self-Hosted KV Budget: Per-Token Formula (Akamai Workshop)

The Akamai workshop provides the explicit per-token formula and worked arithmetic, which the prior session stated but didn't derive:

```
KV bytes per token = 2 (K + V) × layers × kv_heads × head_dim × bytes_per_element
```

For **Qwen3-4B at BF16** (2 bytes/element): 2 × 28 layers × 8 KV heads × 64 head_dim × 2 = **~144 KiB/token**

GQA ratio for Qwen3-4B: 16 query heads share 8 KV heads → 2:1. Without GQA it would be ~288 KiB/token.

**Concurrency table (RTX 4000 Ada, 20 GB, gpu-memory-utilization=0.7):**

| Context length | KV per request | Requests that fit |
|---|---|---|
| 1024 tokens | ~0.14 GB | ~60 |
| 2048 tokens | ~0.29 GB | ~30 |
| 4096 tokens | ~0.58 GB | ~15 |
| 8192 tokens | ~1.18 GB | ~7 |

**Prefix caching measured result:** warm request (shared prefix already cached) showed 2–5× lower TTFT than cold request. vLLM hashes each block by token IDs; changing even one early token invalidates the cache for everything after it.

## Sources

- [LLM Inference at Scale Workshop](../talks/llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah (GPU/serving layer)
- [Context Engineering in 2026: Compaction, Memory & Cost](../talks/context-engineering-2026.md) — Bouchard, Vaid, Solano (application/API layer)
- [Agents That Own Their Inference Workshop](../talks/agents-own-inference.md) — Du'an Lightfoot, Modules 2 & 3 (per-token formula, live prefix-cache measurement)
