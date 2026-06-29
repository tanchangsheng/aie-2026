# Wiki Index

Master catalog of all pages. Updated on every ingest.

---

## Talks

| Page | Speakers | Day/Time | Tags |
|------|----------|----------|------|
| [LLM Inference at Scale](talks/llm-inference-at-scale.md) | Harshul Jain, Tanmay Sah | Day 1 · 12:10–2:15pm · Track 3 | inference, kv-cache, attention, vllm, sglang, quantization, workshop |
| [Context Engineering in 2026: Compaction, Memory & Cost](talks/context-engineering-2026.md) | Louis-François Bouchard, Samridhi Vaid, Omar Solano | Day 1 · 2:20–4:20pm · Track 6 | context-engineering, compaction, rag, caching, eval, agents, workshop |
| [What is an Inference Engine, Anyway?](talks/what-is-an-inference-engine-anyway.md) | Charles Frye | Day 1 · 11:05–12:05pm · Track 8 | inference, vllm, llm-d, kv-cache, infrastructure, workshop |
| [Agents That Own Their Inference](talks/agents-own-inference.md) | Du'an Lightfoot (+ Omer Aslan, co-presenter) | Day 1 · 9:00–11:00am · Track 7 | inference, vllm, kv-cache, quantization, speculative-decoding, moe, kubernetes, self-hosted, workshop |

## Speakers

| Page | Role | Affiliation |
|------|------|-------------|
| [Harshul Jain](speakers/harshul-jain.md) | Sr. Software Engineer — ML/AI | Audible (Amazon) |
| [Tanmay Sah](speakers/tanmay-sah.md) | Sr. Quantitative Modeler | Zions Bancorporation |
| [Louis-François Bouchard](speakers/louis-francois-bouchard.md) | CTO & Co-Founder | Towards AI |
| [Samridhi Vaid](speakers/samridhi-vaid.md) | Senior ML Engineer | Towards AI |
| [Omar Solano](speakers/omar-solano.md) | AI Engineer | Towards AI |
| [Charles Frye](speakers/charles-frye.md) | Member of Technical Staff | Modal |
| [Du'an Lightfoot](speakers/duan-lightfoot.md) | Senior AI Engineer | Akamai Technologies |

## Concepts

| Page | Summary |
|------|---------|
| [KV Cache](concepts/kv-cache.md) | The central memory bottleneck in LLM inference; four levers to manage it; also the key API-layer caching metric for agents |
| [Context Engineering](concepts/context-engineering.md) | Deciding what the model sees every call; compaction + memory + skills under one discipline |
| [Attention Mechanisms](concepts/attention-mechanisms.md) | MHA → GQA → MLA: how attention design determines KV cost and quality |
| [Inference Engines](concepts/inference-engines.md) | vLLM vs SGLang vs TensorRT-LLM — workload-driven selection |
| [Memory Bandwidth & the Roofline](concepts/memory-bandwidth-roofline.md) | Why decode is memory-bound and how to reason about GPU ceilings |
| [Quantization](concepts/quantization.md) | Weight (INT4/INT8) and KV (FP8) quantization — effects on capacity and quality |
| [Speculative Decoding](concepts/speculative-decoding.md) | Draft-and-verify decode; when it helps and when it doesn't |
| [Mixture of Experts (MoE)](concepts/mixture-of-experts.md) | Active vs. total parameters; the VRAM/bandwidth tradeoff |
| [Self-Hosted Inference](concepts/self-hosted-inference.md) | Renting vs. owning: cost, data residency, rate limits, control |

---

_Last updated: 2026-06-29 (4 talks ingested)_
