---
type: overview
updated: 2026-06-29 (4 sessions)
---

# AIE World Fair 2026 — Conference Overview

_This page is updated after each talk ingest. It reflects the evolving synthesis of themes, debates, and key takeaways from the conference._

---

## Major Themes

### 1. Inference Cost Is Now a Board-Level Problem

The dominant framing from the first session: inference has crossed from engineering concern to financial reality. Training GPT-3 cost $4.6M once; serving it now costs more than that per week. The "token hunger games" — teams competing for compute budgets, C-suites forcing token audits — reflects an industry-wide reckoning with operating costs that scale with every user and every session. Cost per 1M tokens has fallen 1,200× since 2023, but volume grew faster.

### 2. The Memory Equation Is the Foundation

Everything in LLM serving reduces to: `GPU memory = weights + KV cache + overhead`. Weights are fixed; the [KV cache](concepts/kv-cache.md) is the variable that grows with users and context length. All optimization work is, at root, managing this variable.

### 3. Decode Is Memory-Bound — Hardware Alone Won't Save You

The decode bottleneck is memory bandwidth, not compute. For Mistral-7B on an A100 the ceiling is ~138 tok/s set by HBM bandwidth. The [roofline](concepts/memory-bandwidth-roofline.md) sits at ~2 FLOP/byte vs 156 for compute saturation. You raise arithmetic intensity through batching and architecture, not by buying a faster chip.

### 4. Compounding Levers, Not Silver Bullets

~20× throughput improvement from stacking orthogonal techniques: [attention architecture](concepts/attention-mechanisms.md) (GQA/MLA) + [weight quantization](concepts/quantization.md) (INT4) + PagedAttention + continuous batching + prefix caching + speculative decoding. Each multiplies the last.

### 5. No Universal Inference Engine

vLLM vs SGLang vs TensorRT-LLM is workload-matching, not a ranking. Near-parity on standard API serving; SGLang is 5.8× faster on agentic/shared-prefix workloads. See [Inference Engines](concepts/inference-engines.md).

### 6. Context Engineering: What the Model Sees Is as Important as the Model Itself

The context engineering workshop added a second lens to the cost story: inference cost isn't just a GPU/serving problem, it's a context management problem at the application layer. Towards AI's empirical finding — keep-everything + prompt caching is cheaper than aggressive compaction — is the application-layer parallel to the inference layer's "stack orthogonal levers" story. In both cases, the answer is non-obvious and requires measurement.

The key application-layer insight: **KV cache hit rate at the API level is as important as KV cache utilization at the serving level**. Anything that rewrites the stable prompt prefix (like summarization) destroys cache coherence and inverts the expected cost saving. This connects the two Day 1 workshops through [KV Cache](concepts/kv-cache.md) as a shared concept operating at different abstraction layers.

### 7. MLA / DeepSeek as the Attention Architecture to Watch

MLA achieves 56× less KV memory than MHA at near-identical quality — a genuine Pareto win. This is why DeepSeek-V3 is cheap to serve. If MLA becomes standard (as GQA already has), the capacity equation changes dramatically.

---

### 8. The Engine/Server Distinction — and Why It Matters for Scale

Charles Frye drew a clean line that's often blurred: the **inference engine** does compute; the **inference server** handles orchestration (routing, resource management, APIs). vLLM is primarily an engine with a thin server. At production scale a dedicated server layer — like llm-d — handles the concerns the engine shouldn't: prefix-cache-aware routing, tiered KV offloading to CPU/disk, disaggregated prefill/decode, and SLO-aware autoscaling. The takeaway: picking an engine and picking a serving stack are two separate decisions.

The framing also sharpens the revenue/cost argument: "training is a cost centre, inference is a revenue centre" — which is why infrastructure investment has followed this direction. See [Inference Engines](concepts/inference-engines.md).

---

### 9. Owning the Inference Layer Is a Three-Axis Decision

The Akamai workshop frames self-hosting as a decision across cost, data residency, and control — not just cost. The cost crossover (hosted per-token vs. flat GPU hourly) is workable arithmetic; data residency is a compliance line that arithmetic can't override; control (model choice, precision, batching policy) is the tuning leverage that the rest of the workshop demonstrates. See [Self-Hosted Inference](concepts/self-hosted-inference.md).

This connects to the context engineering session's API-level cost comparison (DeepSeek vs. Gemini, 18×) to form a three-level cost stack: (1) prompt engineering and caching, (2) model/provider choice, (3) infrastructure (hosted vs. self-hosted).

### 10. Speculative Decoding Is Workload-Dependent, Not a Free Speedup

The Akamai workshop is the first to demonstrate speculative decoding live and report a negative result. The draft-and-verify mechanism (0.6B drafter + 4B target) can hurt throughput at high concurrency when the extra draft work competes with normal batching. Published results show up to 2.8× speedup in the right conditions; this workshop shows it can also slow things down. **A negative result is a valid production decision.** See [Speculative Decoding](concepts/speculative-decoding.md).

### 11. The Hypothesis-Driven Tuning Loop as a Practice

Modules 5–8 introduce an explicit method: measure → change one thing → redeploy → keep or reject with data. This is the production discipline that was implicit in prior sessions but never named. It applies to quantization, speculative decoding, and serving-policy flags alike. The corollary: changing many things at once hides the cause.

---

## Key Debates

- **vLLM vs SGLang for agents**: most teams default to vLLM even for agentic workloads — the benchmark suggests this is wrong. The Akamai workshop is entirely vLLM-centric; their agent from Module 9 likely benefits from SGLang's RadixAttention on shared-prefix workloads.
- **Quantization vs attention architecture**: both address KV cache size from different angles. Which gives more headroom at scale?
- **Compact or keep?**: The context engineering workshop's finding (keep-everything wins at 11–13 turns) is at odds with the conventional wisdom that context should be aggressively managed. The question is at what session length / scale the calculus flips.
- **Local vs cloud for agents**: At ~10k turns/day, the gap between DeepSeek ($1.9k/mo) and Gemini ($34k/mo) is already an 18× decision. The Akamai workshop adds the self-hosted GPU as the third option. At what volume does a dedicated $1.50/hr card beat both?
- **FP8 vs INT4**: The Harshul/Tanmay session emphasised INT4; the Akamai workshop uses FP8. Both reduce weight bytes; FP8 is more conservative on quality risk and better supported by current serving stacks.

## Emerging Consensus

- GQA is now table stakes — it's shipped in every major model.
- FlashAttention is default-on everywhere; there's nothing to configure.
- Prefix caching is underutilized in agent frameworks.
- Measurement-first culture is a consistent thread: all four workshops emphasised running real harnesses rather than accepting conventional wisdom.
- An agent's wall-clock time is almost entirely inference. The inference layer is the agent latency budget.

## Open Questions

- Do compounding optimizations truly multiply in practice, or do some interfere?
- Does RadixAttention's advantage hold at very long contexts (>32K)?
- At what task complexity does INT4 quality degradation become meaningful?
- How does disaggregated prefill/decode fit in the stack at what scale does it become worth the complexity?
- At what session length does compaction become cost-effective over keep-everything + caching?
- Does "lost in the middle" context rot emerge for reasoning-heavy agent tasks at >800k token contexts?
- Under what prompt distributions does a 0.6B drafter consistently achieve >70% acceptance with a 4B target?
- When does scale-to-zero (KServe/Knative) change the self-hosted economics for bursty workloads?

---

## Sessions Ingested

1. [LLM Inference at Scale Workshop](talks/llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
2. [Context Engineering in 2026: Compaction, Memory & Cost](talks/context-engineering-2026.md) — Louis-François Bouchard, Samridhi Vaid, Omar Solano
3. [What is an Inference Engine, Anyway?](talks/what-is-an-inference-engine-anyway.md) — Charles Frye (Modal)
4. [Agents That Own Their Inference](talks/agents-own-inference.md) — Du'an Lightfoot / Omer Aslan (Akamai)
