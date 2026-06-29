---
type: concept
tags: [inference, self-hosted, cost, data-residency, renting, owning, vllm, gpu, production]
updated: 2026-06-29
---

# Self-Hosted Inference (Renting vs. Owning)

## Definition

Self-hosted inference means running your own inference server (e.g., vLLM) on hardware you control, rather than sending requests to a hosted API billed per token. The two models differ on cost structure, data path, rate limits, and operational control.

## The Comparison

| Dimension | Renting (hosted API) | Owning (self-hosted) |
|---|---|---|
| **Cost structure** | Per-token, scales with every request | Flat hourly rate (GPU instance), fixed regardless of volume |
| **Cost crossover** | Cheap at low volume | Cheaper above a certain request/day threshold |
| **Data residency** | Customer data leaves your infrastructure | Data never leaves your cluster |
| **Rate limits** | Provider-set; can throttle during launch | Only ceiling is the card you provisioned |
| **Model control** | Provider's model, provider's updates | You choose the model, precision, context length |
| **Operational burden** | Zero | You operate the server, monitor metrics, tune knobs |

## Cost Crossover (Worked Example)

With Qwen3-4B at 200k requests/day (500 input tokens, 300 output tokens):

- **Hosted** (representative prices, mid-2025): ~$2,700–$3,600/month, growing with volume
- **Owned** (RTX 4000 Ada on Akamai Cloud): ~$1,080/month, flat

Crossover depends on request volume, prompt length, and the hosted provider's pricing. The workshop arithmetic demonstrates the calculation; teams should run it with their actual numbers. The crossover volume for a 4B-class model on a ~$1.50/hr GPU is typically in the low tens of thousands of requests per day.

## Data Residency as a Hard Line

For regulated industries (healthcare, finance, legal), data residency is not a cost optimisation — it is a compliance requirement. A self-hosted request never leaves the cluster. A hosted API request does, regardless of data processing agreements. This is the dimension where the economics argument doesn't apply: owning is the only option.

## What You Actually Own

Self-hosted inference means owning a stack:
- The model (choice of weights, quantization format)
- The server (vLLM, SGLang, TensorRT-LLM — you pick and tune)
- The metrics (your Prometheus endpoint, your dashboard)
- The tuning (context length, batch size, KV headroom — your parameters)
- The agent on top (pointing at your endpoint, not a rented one)

The OpenAI-compatible API surface means existing client code barely changes — only `base_url`.

## The Operational Cost

Self-hosting is not free. You take on:
- Infrastructure provisioning and lifecycle (Kubernetes, GPU nodes)
- Model loading, serving configuration, and version management
- Monitoring the `/metrics` endpoint (TTFT, KV utilisation, queueing, preemptions)
- A tuning loop: measure → change one flag → redeploy → keep or reject

The workshop's thesis: this operational cost is the entry price for rate-limit immunity, data sovereignty, and the ability to tune for your specific workload rather than accepting a provider's defaults.

## Connection to Context Engineering Cost Analysis

The context engineering workshop (Towards AI, Day 1) also ran cost comparisons — but at the API/application layer rather than the infrastructure layer. That workshop found DeepSeek at ~$1.9k/month vs Gemini at ~$34k/month for the same workload (18× difference from model choice alone). Self-hosting adds a third point on that curve: the fixed-cost alternative that becomes cheapest at sustained high volume.

The two workshops together frame a **three-level cost stack**: (1) application-layer prompt engineering (caching, compaction), (2) model choice (provider pricing tiers), (3) infrastructure choice (hosted vs self-hosted).

## Open Questions

- At what request volume does self-hosting make sense for a team with no existing Kubernetes expertise?
- How does scale-to-zero (KServe/Knative) change the economics for bursty, low-average-volume workloads?
- Does the vLLM operational overhead scale with number of models served, or is it roughly fixed?

## Sources

- [Agents That Own Their Inference Workshop](../talks/agents-own-inference.md) — Du'an Lightfoot, Modules 1 & 9
- [Context Engineering in 2026](../talks/context-engineering-2026.md) — Bouchard, Vaid, Solano (application-layer cost comparison)
