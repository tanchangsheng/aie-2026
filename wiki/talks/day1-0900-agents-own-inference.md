---
type: talk
tags: [inference, vllm, kv-cache, quantization, speculative-decoding, moe, kubernetes, gpu, self-hosted, workshop]
updated: 2026-06-29
---

# Agents That Own Their Inference: Building Production AI Agents on Dedicated GPUs

## Metadata

| Field | Value |
|---|---|
| Speakers | Du'an Lightfoot (Akamai, Modules 0–4 & 9), Omer Aslan (co-presenter, Modules 5–8) |
| Day / Time | Day 1 — Workshop Day · 9:00am–11:00am |
| Room | Track 7 |
| Track | Sponsor workshop |
| Format | 120-min hands-on Jupyter notebooks on Akamai Cloud GPU (LKE + vLLM + Qwen3-4B) |
| Status | Confirmed |

> Note: Omer Aslan co-presented Modules 5–8 (the performance block) but is not listed in the official conference speaker registry.

## Official Description

Every production agent today is renting its intelligence. You're paying per token, sending your customer's data to someone else's servers, and hoping the provider doesn't rate-limit you during your launch. For most teams, that's fine. But for a growing number of teams in regulated industries, with high-volume products, latency-sensitive workloads, or rising token bills, it's starting to look like a liability.

In this 120-minute hands-on workshop you'll get a dedicated GPU and build an agent that runs on infrastructure you control. You'll stand up vLLM, point your agent at it, and drive concurrent load through the stack until you can see batching, KV cache pressure, and throughput limits in the metrics. Then you'll optimize the deployment to improve throughput while keeping per-request latency in line.

The focus isn't agent frameworks. It's the inference layer underneath them.

## Module Map

| Module | Presenter | Topic |
|---|---|---|
| 0 | Du'An | Connect and verify (environment, kubectl, GPU, first request) |
| 1 | Du'An | Inference stack: renting vs owning, request trace, runtime landscape |
| 2 | Du'An | Units and memory budget: parameters, bytes, VRAM, KV formula, concurrency |
| 3 | Du'An | Prefill, decode, KV cache: build attention by hand, measure TTFT, prefix caching |
| 4 | Du'An | Dense vs MoE: roofline model, MoE active vs total params, reasoning tax |
| 5 | Omer | Quantization: BF16 baseline → FP8 via manifest edit → compare |
| 6 | Omer | Speculative decoding: 0.6B drafter → measure acceptance and throughput |
| 7 | Omer | Engine saturation: concurrency sweep → find the knee → read the metrics |
| 8 | Omer | Tune and evaluate: one hypothesis → one flag → redeploy → keep or reject |
| 9 | Du'An | Agent on Kubernetes: deploy, call it, watch it batch in your vLLM metrics |

## Key Claims

**On the inference layer:**
- The inference engine is a real layer — the same model on a different engine and card changes throughput by orders of magnitude.
- To make one decode token the GPU reads every weight once. Memory bandwidth (not compute) is the ceiling: `tok/s ≤ bandwidth ÷ weight_GB`.
- For RTX 4000 Ada (360 GB/s): BF16 Qwen3-4B (~8 GB) → ~45 tok/s ceiling; FP8 (~4–5 GB) → ~90 tok/s.
- The KV cache memory formula: `2 × layers × kv_heads × head_dim × bytes_per_element` = ~144 KiB/token for Qwen3-4B at BF16.
- GPU memory budget: weights (fixed) + KV pool (variable) + overhead. KV pool ÷ sequence length = concurrent requests that fit.

**On MoE:**
- Qwen3-30B-A3B: 30.5B total parameters, only ~3.3B active per token (8 of 128 experts fire).
- MoE decode reads ~6.7 GB/token vs ~61 GB for a dense 30B → ~9× faster decode, at the cost of storing the full 30.5B in VRAM.
- MoE trades VRAM footprint for a smaller per-token bandwidth read.

**On quantization (FP8):**
- FP8 halves weight bytes → raises the memory-bandwidth decode ceiling and frees VRAM for KV cache.
- Workflow: measure baseline → change one thing (the model in the manifest) → re-measure → keep or reject with quality evidence.
- In rehearsal, FP8 was a clear throughput win for this model and workload.
- Quality check is non-optional before production: use deterministic assertions + LLM-as-judge harnesses.

**On speculative decoding:**
- Draft-and-verify: a 0.6B FP8 drafter proposes up to 4 tokens; the 4B FP8 target verifies in one parallel pass.
- Speedup appears only at low-to-medium concurrency. At high concurrency the extra draft work competes with normal batching.
- A negative result is a valid production decision — measure acceptance and throughput before keeping the draft path.
- Drafter family alignment matters. A 0.6B from the same model family (Qwen) drafts better than a random small model, but acceptance still depends on prompt distribution.

**On engine saturation:**
- Continuous batching: one weight read serves many requests simultaneously. Throughput rises with concurrency until something else becomes the limit.
- The saturation knee: throughput flattens, TTFT rises. Beyond the knee you're paying latency for no throughput gain.
- Metric signals: `num_requests_running`, `gpu_cache_usage_perc`, `num_requests_waiting`, preemption counts.

**On agents:**
- An agent's wall-clock time is almost entirely inference. A CPU tool call rounds to zero versus seconds of model time.
- Six concurrent agents batch on the server and finish in far less than 6× the time of one — continuous batching is visible from the agent side.
- An agent is just an HTTP client with a system prompt. No framework required; swap in any framework and the inference layer stays the same.

## Notable Quotes

> "Every production agent today is renting its intelligence." — workshop premise

> "A negative result is not a failed lab; it is the production decision you would want to make before shipping a slower server." — Module 6, on speculative decoding

> "Inference is essentially 100% of the agent's wall time." — Module 9

## My Reactions

The hypothesis-driven workflow (measure → change one thing → redeploy → keep or reject) is the most disciplined framing I've seen at AIE so far. The prior inference workshop (Harshul & Tanmay) laid out the theory; this one makes you execute it against a real deployment. The gap between knowing the roofline and reading the saturation knee in live metrics is significant.

The speculative decoding section is refreshingly honest — the workshop explicitly prepares you for a negative result and explains why it's still valuable. Most demonstrations cherry-pick the happy path.

Tension with prior sessions: the inference workshop benchmarked vLLM vs SGLang and found SGLang 5.8× faster on agentic workloads. This workshop never mentions SGLang — it's entirely vLLM-centric. A fair point to investigate: should the agent from Module 9 be pointed at SGLang for a shared-prefix workload?

The Omer Aslan performance block (Modules 5–8) felt like a different workshop grafted onto Du'An's conceptual half — same repo, different presenter style. The connection points (FP8 result is the baseline for speculative decoding, which is the baseline for saturation) are well-constructed but the seam shows.

## Links

- [Du'an Lightfoot](../speakers/duan-lightfoot.md)
- Concepts: [KV Cache](../concepts/kv-cache.md) · [Inference Engines](../concepts/inference-engines.md) · [Memory Bandwidth & the Roofline](../concepts/memory-bandwidth-roofline.md) · [Quantization](../concepts/quantization.md) · [Speculative Decoding](../concepts/speculative-decoding.md) · [Mixture of Experts](../concepts/mixture-of-experts.md) · [Self-Hosted Inference](../concepts/self-hosted-inference.md)
- Repo: https://github.com/akamai-developers/akamai-workshop-ai-inference
