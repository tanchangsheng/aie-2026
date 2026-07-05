---
type: concept
tags: [gpu, scheduling, inference, latency, cost, throughput, bin-packing, agent-aware]
updated: 2026-07-05
---

# GPU Scheduling

Deciding which GPU(s) to place an inference request on, and how to batch requests together. As inference workloads have grown more complex, GPU scheduling has evolved from simple bin-packing to multi-dimensional, agent-aware optimization.

## The evolution

**Old scheduler:** Two axes — CPU and memory. Bin-packing problem. Works fine for stateless services.

**Modern inference scheduler:** Seven dimensions must be considered simultaneously:
- Model (which weights are loaded)
- GPU Memory (HBM availability for KV cache)
- Latency SLA (TTFT / TPOT targets per tenant)
- Tenant (quota, priority, isolation)
- Context Size (how much KV cache the request will consume)
- Cost (spot vs. reserved, accelerator tier)
- Reasoning Workflow (where is this request in a multi-step agent chain)

> *Agent-aware scheduling: place work where the whole workflow will finish fastest and cheapest — not where one GPU is free.*
> — Gupta & Ahuja, AIE 2026

## The impossible triangle

Every scheduling decision moves the system inside a three-way tension:

```
        LATENCY
          /\
         /  \
        / ·  \    ← you are always here
       /      \
   COST ———— THROUGHPUT
```

Optimizing for any one vertex degrades at least one other. There is no static optimum — the right position depends on real-time load, tenant priority, and cost budget.

> *Optimization is continuous, not static. Every scheduling decision moves the system somewhere inside this triangle.*

## Optimization as a decision tree

Gupta & Ahuja's framework: before doing work, ask four questions in order:

1. **Can we AVOID it?** → Cache (prompt, prefix, KV, response)
2. **Can we SHARE it?** → Batch (continuous / in-flight batching)
3. **Can we MOVE it?** → Route (region, model tier, accelerator)
4. **Can we DELAY it?** → Queue (priority, admission, defer)

## Cascade risk

Mis-scheduling under load creates positive feedback loops: GPU failure → retry storm → queue growth → timeouts → more retries → system collapse. The scheduler must participate in cascade prevention, not just placement.

## Relationship to other concepts

- [Inference Control Plane](inference-control-plane.md) — scheduling is one of the control plane's core responsibilities
- [KV Cache](kv-cache.md) — KV cache availability is a first-class scheduling input
- [Guardrails](guardrails.md) — admission control (reject/defer under load) is a form of scheduling-adjacent policy
- Workshop coverage: [LLM Inference at Scale](../talks/day1-1210-llm-inference-at-scale.md), [Agents That Own Their Inference](../talks/day1-0900-agents-own-inference.md)

## Open questions

- How do you model the "reasoning workflow position" of an in-progress agent chain as a scheduling input without coupling the scheduler to agent logic?
- At what queue depth does deferring requests become cheaper than degraded-model fallback?
