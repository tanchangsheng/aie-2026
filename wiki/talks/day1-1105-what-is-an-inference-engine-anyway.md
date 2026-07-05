---
type: talk
tags: [inference, vllm, llm-d, kv-cache, infrastructure]
updated: 2026-06-29
---

# What is an Inference Engine, Anyway?

## Metadata

| Field | Value |
|---|---|
| Speaker | [Charles Frye](../speakers/charles-frye.md) |
| Affiliation | Modal (Member of Technical Staff) |
| Type | Workshop |
| Track | Workshops Day 1 |
| Day | Day 1 — Workshop Day |
| Time | 11:05am–12:05pm |
| Room | Track 8 |
| Status | Confirmed |

**Official description:** To run state-of-the-art inference yourself, you must master the inference engine: vLLM, SGLang, TRT-LLM, or your own jawn. The inference engine manages the lifecycle of an inference request, from input to output. In this workshop, we'll examine the architecture of modern high performance inference engines, the key techniques that inference engines need to deliver that performance, and the traces and metrics that inference engines emit.

**Workshop notebooks:** https://github.com/akamai-developers/akamai-workshop-ai-inference

---

## Key Claims

**Inference vs Training**
Companies focus more on LLM inference than training because training is a cost centre but inference is a revenue centre. This reframes infrastructure investment decisions.

**Engine vs Server distinction**
Inference servers wrap inference engines — they handle requests, manage resources, and provide APIs for clients. The inference engine is the core component that performs the actual compute: it takes input, processes it through the model, and produces output. vLLM contains both but is more engine than server.

**llm-d as orchestration layer**
llm-d is an inference server that sits above engines like vLLM. It handles orchestration and optimizations for high-scale, real-world traffic across five themes:
- *Intelligent Routing*: prefix-cache and load-aware balancing, with experimental predicted-latency scheduling
- *Advanced KV-Cache Management*: tiered offloading to CPU or disk, global indexing of KV cache state for multi-turn requests
- *Serving Large Models*: prefill/decode disaggregation and wide expert-parallelism over fast accelerator interconnects (e.g. DeepSeek-R1, GPT-OSS)
- *Operational Excellence*: flow control for multi-tenant serving, SLO-aware autoscaling from real-time inference signals
- *Batch Processing*: OpenAI-compatible Batch APIs and async processing for offline inference

**Statefulness and KV cache**
The inference engine is stateful when performance matters — the KV cache is the key stateful component. Vanilla vLLM is stateless by default, which yields lower performance.

---

## Notable Quotes

> "Training is a cost centre but inference is a revenue centre."

---

## Reactions

The engine/server distinction is a useful mental model that's often collapsed in casual usage. The llm-d breakdown (routing → KV management → large model serving → ops → batch) maps well onto the layers where production systems actually fail. The statefulness point is underappreciated — most tutorials skip it.

---

## See Also

- [Charles Frye](../speakers/charles-frye.md)
