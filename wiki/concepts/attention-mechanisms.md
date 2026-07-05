---
type: concept
tags: [attention, gqa, mla, mha, flashattention, inference, memory]
updated: 2026-06-29
---

# Attention Mechanisms

## Definition

Attention computes, for each token, a weighted mix of past token values using query-key similarity:

```
out = softmax( Q · Kᵀ / √d ) · V
```

The design of attention — how many heads share keys and values — determines KV cache size, quality, and throughput.

## Mechanism Scorecard

| Mechanism | KV/Token | Quality | Verdict |
|-----------|----------|---------|---------|
| MHA (Multi-Head) | 524 KB · 1× | Reference | Memory-bound |
| GQA-8 (Grouped Query) | 131 KB · 4× | ≈ MHA | ~free 4× win — ships in Mistral, Llama, Qwen |
| MQA (Multi-Query) | ~16 KB · 32× | Dips | Max KV cut, quality cost |
| MLA (Multi-Latent) | ~36 KB · 14× | Near-MHA | Pareto winner — DeepSeek-V3 |
| Sliding Window | constant | Long ctx | Weak retrieval |
| Linear Attention | constant | O(N) | Recall bottleneck |
| Mamba | constant | Selective SSM | Linear time |

## Key Insight: MLA

DeepSeek-V3's Multi-Latent Attention achieves **56× less KV memory** than MHA (9.3 KB vs 524 KB per token) at near-MHA quality. This is why DeepSeek is cheap to serve — it's not training cleverness, it's attention architecture.

## FlashAttention

Bit-exact to standard attention but reorders the computation to stay in SRAM, avoiding repeated HBM reads. Up to 6× faster. No accuracy tradeoff. Default in every major engine (vLLM, SGLang, TensorRT-LLM). Already on — not something to configure.

## The Trade-off Triangle

```
QUALITY
  MHA — all quality, no efficiency
  MLA — Pareto winner
  GQA — pragmatic middle
  MQA — cache wins, quality drops

MEMORY EFFICIENCY ←——→ THROUGHPUT
```

No single corner beats all others.

## Sources

- [LLM Inference at Scale Workshop](../talks/day1-1210-llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
