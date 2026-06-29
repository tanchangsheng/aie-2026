---
type: concept
tags: [quantization, fp16, int8, int4, fp8, memory, weights, kv-cache]
updated: 2026-06-29 (2 sources)
---

# Quantization

## Definition

Quantization reduces the numerical precision of model weights (and optionally KV cache values) to use fewer bits per parameter, shrinking memory footprint and increasing throughput.

## Weight Quantization (Mistral-7B)

| Precision | Model Size | Freed for KV Cache | Quality Impact |
|-----------|-----------|-------------------|----------------|
| FP16 | 14.5 GB | baseline | reference |
| INT8 | 7.2 GB | +7.2 GB freed | minimal |
| INT4 / NF4 | 3.6 GB | +10.9 GB freed | minor loss on reasoning |

Every gigabyte freed from weights directly becomes additional KV cache budget — more room, more concurrent users.

## KV Cache Quantization

Separately from weight quantization, the KV cache itself can be stored in reduced precision:

- **FP16 → FP8**: 2× more users for the same KV budget
- Quality loss < 0.1% — KV values are less sensitive than weights
- vLLM flag: `--kv-cache-dtype fp8`

## Combined Effect

GQA (4×) + INT4 weight quantization (4×) = **~7× more capacity** from the same GPU, before any serving-level optimizations. This is the first 7× of the ~20× compounding stack.

## Key Insight

Quantization and attention architecture (GQA/MLA) address the same problem — KV cache size — from different angles. They stack multiplicatively: INT4 weights free GPU memory for more KV cache; GQA reduces how much KV cache each user needs in the first place.

## Open Questions

- At what context length does INT4 quality degradation become noticeable for complex reasoning tasks?
- Does FP8 KV quantization interact differently with MLA vs GQA architectures?

## The Hypothesis-Driven FP8 Workflow (Akamai Workshop)

The Akamai workshop adds the operational procedure for switching to FP8 in production, not just the theory:

1. Measure BF16 baseline (fixed prompt shape, fixed concurrency levels, record throughput + TTFT p95)
2. Edit `manifests/vllm.yaml` — change only the `--model` argument to the FP8 build
3. `kubectl apply` → `kubectl rollout status` → confirm via `/v1/models`
4. Run the same sweep with identical inputs
5. Compare; run quality checks; keep or reject

**Observed result (rehearsal):** FP8 was a clear throughput win for Qwen3-4B on RTX 4000 Ada.

**Quality check non-negotiable:** before shipping a precision change, run workload evals covering: JSON validity, keyword presence, refusal behaviour, tool call accuracy, factual drift. Tools: promptfoo, DeepEval, Ragas, lm-evaluation-harness.

**Format ladder (complete, Akamai workshop):**

| Method | Bytes/weight | Main upside | Main risk |
|---|---|---|---|
| BF16 / FP16 | 2 | Broad support | Larger weight reads |
| FP8 weights | ~1 | Smaller reads, more cache room | Hardware/engine support |
| INT8 weights | 1 | Conservative compression | Less gain than lower-bit |
| INT4 / W4A16 | 0.5 | Large memory drop | Quality risk |
| AWQ / GPTQ | varies | Better low-bit quality | Calibration set dependency |
| KV-cache quantization | 1 (cache) | More concurrent context | Long-context drift |
| GGUF | varies | Edge / llama.cpp | Different serving stack |

**Disagreement with prior session:** the Harshul/Tanmay workshop framed INT4/NF4 as the primary quantization lever (3.6 GB vs 14.5 GB for Mistral-7B). The Akamai workshop's live path uses FP8, which is more conservative and better supported by current serving stacks. Both are valid; the choice depends on the quality/throughput tradeoff acceptable for the workload.

## Sources

- [LLM Inference at Scale Workshop](../talks/llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
- [Agents That Own Their Inference Workshop](../talks/agents-own-inference.md) — Omer Aslan, Module 5 (FP8 workflow, format ladder, quality checks)
