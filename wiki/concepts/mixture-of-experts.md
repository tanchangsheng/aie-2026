---
type: concept
tags: [moe, inference, architecture, decode, memory, gpu, parameters]
updated: 2026-06-29
---

# Mixture of Experts (MoE)

## Definition

A Mixture-of-Experts model holds many expert sub-networks (typically FFN layers) but routes each token to only a small subset of them — the **active experts**. Total parameters set the VRAM footprint; active parameters per token set the per-token memory bandwidth cost.

## The Key Asymmetry

| Quantity | Role |
|---|---|
| Total parameters | Sets VRAM footprint (all experts must be stored) |
| Active parameters per token | Sets decode bandwidth cost (only active experts are read) |

This asymmetry means an MoE can deliver the quality of a large model while paying the per-token bandwidth cost of a much smaller one — **at the cost of VRAM**.

## Worked Example: Qwen3-30B-A3B

Derived from the model's real `config.json`:

- **Total:** 30.5B parameters (~61 GB at BF16, ~30 GB at FP8)
- **Config:** 128 experts, 8 active per token, across 94 layers; hidden size 3072; expert MLP width 1536
- **Active per token:** ~3.3B ("A3B" in the model name)
- **Decode reads:** ~6.7 GB/token (active experts only)
- **Dense 30B would read:** ~61 GB/token
- **Effective speedup vs dense 30B:** ~9× faster decode on bandwidth-limited hardware

Quality sits between a 3B dense model and a full 30B — driven by the active parameter count, not the total.

## Roofline Placement

On the [memory bandwidth roofline](memory-bandwidth-roofline.md), an MoE model's decode lands higher (better) than a dense model of equal total size, because it reads fewer bytes per token:

- Dense 4B: reads ~8 GB/token (BF16) → ~45 tok/s ceiling on RTX 4000 Ada
- MoE 30B-A3B: reads ~6.7 GB/token → similar ceiling — with far higher effective quality

The practical catch: the 30.5B at FP8 needs ~30 GB VRAM, which overflows a 20 GB card. MoE is a multi-GPU story at this scale.

## Multi-GPU: Expert Parallelism

Dense models scale with tensor and pipeline parallelism. MoE models have an additional lever: **expert parallelism** — different GPUs hold different expert subsets, and the router directs tokens across nodes. This is the MoE-specific reason multi-GPU scaling can be more efficient for MoE than for dense.

## Relationship to Other Concepts

- MoE is the "read fewer bytes per token" architectural answer, complementary to [quantization](quantization.md) (fewer bytes per weight) and [speculative decoding](speculative-decoding.md) (fewer target steps).
- The "A3B" naming convention (active-parameter count) is useful shorthand; compare it to the total to understand the routing ratio.

## Open Questions

- At what VRAM size does an MoE become practical on a single card with FP4/NF4 quantization?
- How does expert parallelism interact with prefill-decode disaggregation?
- Is Qwen3-30B-A3B actually cheaper to serve than Qwen3-4B at the same quality level, once you account for the multi-GPU overhead?

## Sources

- [Agents That Own Their Inference Workshop](../talks/day1-0900-agents-own-inference.md) — Du'an Lightfoot, Module 4
