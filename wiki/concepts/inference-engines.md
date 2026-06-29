---
type: concept
tags: [vllm, sglang, tensorrt, inference, serving, engines]
updated: 2026-06-29 (3 sources)
---

# Inference Engines

## Overview

LLM inference engines handle batching, KV cache management, scheduling, and GPU kernel execution. The major production engines as of 2026 are vLLM, SGLang, and TensorRT-LLM. Choice depends on workload, not on a single "best" engine.

## Engine Comparison

| Engine | Best For | Key Feature | Tradeoff |
|--------|----------|-------------|----------|
| **vLLM** | General production API | PagedAttention + continuous batching | Broadest support, simplest ops |
| **SGLang** | Agents + multi-turn | RadixAttention (tree-based prefix cache) | Slightly lower throughput on standard API |
| **TensorRT-LLM** | Max NVIDIA throughput | AOT compile, CUDA graphs, FP8 | Build step required, NVIDIA-only |
| **NVIDIA Dynamo** | Agentic session routing | KV-aware load balancing | Emerging |
| **HuggingFace** | Prototyping | Simple, no server needed | 24× lower throughput than vLLM |

## Benchmark (Qwen2.5-7B · fp16 · H100 80GB · 20 concurrent · 1,000 ShareGPT reqs)

**Standard API workload (independent requests):**
| Metric | vLLM | SGLang |
|--------|------|--------|
| Requests/sec | 29.79 | 29.03 |
| Tokens/sec | 7,453 | 7,279 |
| Avg TTFT | 0.51 s | 0.52 s |
| P99 latency | 3.61 s | 3.97 s |

**Agentic workload (~1K-token shared prefix, 3 branches):**
| Engine | TTFT |
|--------|------|
| vLLM default | 6.30 s |
| vLLM + prefix cache | 4.80 s |
| SGLang native | **1.07 s** |

SGLang is **5.8× faster** than vLLM default on agentic workloads due to RadixAttention.

At 120B scale (Clarifai benchmark on H100/B200): same pattern holds — TensorRT-LLM wins raw throughput once compiled, SGLang wins on agent/multi-turn, vLLM is most consistent and easiest to operate.

## vLLM Architecture

```
API Server → Scheduler → KV Block Manager (PagedAttention) → GPU Workers (fused kernels)
```
Start command: `vllm serve mistralai/Mistral-7B-v0.1`
24× throughput vs HuggingFace baseline.

## SGLang: RadixAttention

Tree-based prefix cache — stores shared prefix at root, branches reuse it automatically. 5× cache hit rate vs vLLM on agent workloads. Detects shared system prompts without configuration.

## Engine vs Server Distinction

Per Charles Frye: the **inference engine** is the core compute component — it takes input, processes it through the model, and produces output. The **inference server** wraps the engine to handle request routing, resource management, and client APIs. vLLM blurs this boundary (mostly engine, minimal server); production systems often need a separate server layer.

**llm-d** is an inference server designed to sit above engines like vLLM. It adds orchestration for real-world scale across five dimensions:

| Layer | What it does |
|---|---|
| Intelligent Routing | Prefix-cache and load-aware balancing; predicted-latency scheduling |
| KV-Cache Management | Tiered offload to CPU/disk; global KV index for multi-turn |
| Large Model Serving | Prefill/decode disaggregation; wide expert-parallelism (DeepSeek-R1, GPT-OSS) |
| Operational Excellence | Multi-tenant flow control; SLO-aware autoscaling from real-time signals |
| Batch Processing | OpenAI-compatible Batch API; async offline inference |

## Statefulness

Inference engines are **stateful when performance matters** — the KV cache is the key stateful component. Vanilla vLLM is stateless by default, which gives lower performance. Enabling the KV cache (stateful mode) is required for production throughput.

This connects the engine layer directly to the serving-layer KV discussion in [KV Cache](kv-cache.md).

## vLLM Serving-Policy Knobs (Akamai Workshop)

The Akamai workshop adds the operational view — tuning a live vLLM deployment rather than benchmarking across engines:

| Flag | What it controls | When to change |
|---|---|---|
| `--gpu-memory-utilization` | Fraction of VRAM reserved (default 0.7 in workshop) | KV cache is tight |
| `--max-model-len` | Context length the engine plans around | Lower to free cache for more concurrency |
| `--max-num-seqs` | Max requests in the running batch | Throughput flattening while requests queue |
| `--max-num-batched-tokens` | Token budget per scheduler step | Long prompts waiting too long |

**Saturation signals to watch:** `num_requests_running`, `gpu_cache_usage_perc`, `num_requests_waiting`, preemption counts. The saturation knee (throughput flat, TTFT rising) is the operating boundary.

**llama.cpp added as the fourth category:** best for local, edge, and CPU-first inference. Runs heavily quantized models from a single file. All four major engines (vLLM, SGLang, TensorRT-LLM, llama.cpp) expose an OpenAI-compatible server, so client code moves between them.

**Cross-engine tension:** the Harshul/Tanmay workshop found SGLang 5.8× faster than vLLM default on shared-prefix/agentic workloads. The Akamai workshop is entirely vLLM-centric. Open question: should the Module 9 agent be pointed at SGLang for the shared-prefix case?

## Open Questions

- How does the engine choice interact with multi-node / disaggregated prefill-decode architectures?
- Does RadixAttention's advantage hold at very long contexts (>32K)?
- Where does llm-d's routing layer fit relative to SGLang's native RadixAttention — complementary or competing?
- Should production agents on Akamai Cloud default to SGLang rather than vLLM for shared-prefix workloads?

## Sources

- [LLM Inference at Scale Workshop](../talks/llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
- [What is an Inference Engine, Anyway?](../talks/what-is-an-inference-engine-anyway.md) — Charles Frye (Modal)
- [Agents That Own Their Inference Workshop](../talks/agents-own-inference.md) — Du'an Lightfoot / Omer Aslan (operational tuning, saturation, llama.cpp addition)
