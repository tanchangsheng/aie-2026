---
type: concept
tags: [gpu, memory, bandwidth, roofline, decode, prefill, hardware]
updated: 2026-06-29 (2 sources)
---

# Memory Bandwidth & the Roofline

## The Core Problem

LLM decode is **memory-bandwidth bound, not compute-bound**. Every decode step streams the full model weights from HBM to the chip, regardless of how many tokens are being generated.

For Mistral-7B on A100-80:
```
14.5 GB ÷ 2 TB/s = 7.25 ms/token → ~138 tok/s ceiling
```

This is the theoretical maximum before any other overhead.

## GPU Memory Hierarchy

| Level | Size | Bandwidth |
|-------|------|-----------|
| SRAM (on-chip, per SM) | 20 MB | 19 TB/s |
| HBM (GPU main memory) | 80 GB | 2 TB/s |

The model (14.5 GB) is **700× larger than all SRAM combined** — it cannot fit on-chip. Every decode step, the full weights stream HBM → L2 → SRAM → compute → discard → repeat.

## Prefill vs. Decode

| Phase | Bound | FLOP/byte | Characteristics |
|-------|-------|-----------|-----------------|
| Prefill | Compute | ~100–400 | Parallel, matrix × matrix, sets TTFT (~50 ms) |
| Decode | Memory | ~1–2 | Sequential, matrix × vector, ITL ~10 ms/token |

The roofline for compute saturation is ~156 FLOP/byte. Decode sits at ~2 — deeply in the memory-bound regime.

## Why This Matters for Optimization

- You cannot solve decode latency with a faster GPU alone — you need to raise arithmetic intensity
- The primary lever is **batching** (more users per forward pass = more compute per byte moved)
- Every optimization in the stack either reduces the bytes moved (quantization, GQA/MLA) or amortizes the move across more work (batching, continuous batching)

## The Decode Ceiling by GPU

| GPU | HBM Bandwidth | Mistral-7B Ceiling |
|-----|--------------|-------------------|
| A100 | 2 TB/s | ~138 tok/s |
| H100 | 3.35 TB/s | ~230 tok/s |
| H200 | 4.8 TB/s | ~330 tok/s |

## RTX 4000 Ada (Akamai Workshop)

The Akamai workshop adds concrete numbers for a consumer/prosumer GPU rather than the datacenter cards above:

```
RTX 4000 Ada: 20 GB GDDR6 at 360 GB/s, ~107 TFLOP/s dense FP16
Ridge point: 107 TFLOP/s ÷ 0.36 TB/s ≈ 297 FLOP/byte
```

Key observation: the ridge point clusters near **300 FLOP/byte** across cards (A100, H100, RTX Ada, RTX PRO 6000 Blackwell ~313). Compute and bandwidth scale together, so **decode is always memory-bound regardless of the GPU**.

**Decode ceilings on RTX 4000 Ada:**

| Model | Precision | Weight size | Decode ceiling |
|---|---|---|---|
| Qwen3-4B | BF16 | ~8 GB | ~45 tok/s |
| Qwen3-4B | FP8 | ~4–5 GB | ~72–90 tok/s |
| Qwen3-0.6B | BF16 | ~1.2 GB | ~300 tok/s |

The MoE comparison: Qwen3-30B-A3B reads only its active experts (~6.7 GB/token at BF16), giving a similar decode ceiling to a 4B dense model despite 30.5B total parameters — at the cost of needing ~30 GB VRAM for the full expert set.

## Sources

- [LLM Inference at Scale Workshop](../talks/day1-1210-llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
- [Agents That Own Their Inference Workshop](../talks/day1-0900-agents-own-inference.md) — Du'an Lightfoot, Modules 2 & 4 (RTX Ada numbers, MoE roofline comparison)
