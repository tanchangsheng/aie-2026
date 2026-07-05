---
type: concept
tags: [inference, memory, optimization, kv-cache, context-engineering, caching]
updated: 2026-07-05 (6 sources)
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

## KV Cache Economics in Model Routing

A fourth layer of KV cache concern emerges from model routing: **the cost of invalidating the API-level cache by switching models mid-session**.

When a coding agent accumulates a long shared prefix (prior turns, system prompt, tool schemas), the provider caches that prefix and charges ~10% of uncached input cost on subsequent hits. If a router switches from Model A to Model B, Model B must re-process the entire context at uncached rates — potentially costing more than staying on the more expensive model.

A **cache-aware router** must track:
- Whether the cache is actually warm (TTL is commonly 5 minutes)
- Context compaction events (which reset the prefix)
- Media attachments and other prefix-invalidating changes

Cache-unaware routing can lose all of its paper savings on a real coding session. This is one of the clearest places where routing for agentic workloads diverges from routing for chat.

See: [Model Routing](model-routing.md)

## Agent-Layer Prefix Caching (Factory)

Tížková adds a production perspective on prefix caching at the agent-platform layer. A transformer re-reads the full conversation history on every turn; prefix caching skips recomputing the unchanged parts (system prompt, tools, skills). Her reported ratio: **the second turn is ~10× cheaper than the first**.

Factory passes these savings directly to end users rather than pocketing them. The caveat: "caching is not a moat" — any team can implement it, including self-hosted open models. The final price a user sees is a pricing decision, not a technical constraint.

This is consistent with the Towards AI finding that keeping-full-history + caching often beats compaction, and with Not Diamond's observation that cache-unaware routing can lose all of its paper savings. All three converge on the same production lesson: **the KV cache hit rate at the API layer is as important as KV cache utilization at the serving layer.**

See: [Software Factory](software-factory.md)

## KV Cache Match as an Engine Routing Signal (OpenAI ILB)

Lao & Zhang introduce an angle not covered in prior talks: **KV cache match** as a first-class signal in *engine routing* — i.e., which GPU replica serves a request, not which model. Their Inference Load Balancer's routing signals are: TBoT, TTFT, load, network overhead, error rate, and KV cache match. Routing a request to an engine that already holds the relevant cached prefix avoids recomputing it, directly reducing TTFT.

This is distinct from the cache-aware *model* routing concern in the Not Diamond talk (don't switch models mid-session because it invalidates the prefix). The OpenAI framing is at a lower level: even within the same model, different replicas have different cache states, and the routing layer should prefer the one that already holds the prefix.

Together, the two talks bracket KV cache into three tiers where it now appears as a production signal: (1) at the API/application layer (Towards AI, Factory — prompt caching hit rate), (2) at the model-routing layer (Not Diamond — don't invalidate by switching models), and (3) at the engine-routing layer (OpenAI — route to the replica that already has the prefix).

See: [Inference Control Plane](inference-control-plane.md) · [Routing LLM Inference in Production](../talks/day4-1110-routing-llm-inference-in-production.md)

## Sources

- [LLM Inference at Scale Workshop](../talks/day1-1210-llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah (GPU/serving layer)
- [Context Engineering in 2026: Compaction, Memory & Cost](../talks/day1-1420-context-engineering-2026.md) — Bouchard, Vaid, Solano (application/API layer)
- [Agents That Own Their Inference Workshop](../talks/day1-0900-agents-own-inference.md) — Du'an Lightfoot, Modules 2 & 3 (per-token formula, live prefix-cache measurement)
- [Intelligent Model Routing: Frontier Performance Without Frontier Bills](../talks/day3-1450-intelligent-model-routing.md) — Tomás Hernando Kofman (cache-aware routing; model-switch invalidation costs)
- [Rise of the Software Factory](../talks/day2-1110-rise-of-the-software-factory.md) — Tereza Tížková (agent-platform prefix caching; 10× second-turn cost reduction; pass-through economics)
- [Operating Distributed Inference Systems at Scale](../talks/day4-1045-operating-distributed-inference-systems-at-scale.md) — Gupta & Ahuja (KV cache hit rate as one of 8 canonical observability signals; KV cache availability as a first-class scheduler input; AVOID→cache as the first question in the optimization decision tree)
- [Routing LLM Inference in Production: From Engine Signals to Policy](../talks/day4-1110-routing-llm-inference-in-production.md) — Lao & Zhang (KV cache match as a first-class engine-routing signal; three-tier KV cache signal stack: API layer, model-routing layer, engine-routing layer)
