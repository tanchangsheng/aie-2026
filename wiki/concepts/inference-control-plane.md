---
type: concept
tags: [inference, control-plane, orchestration, scheduling, routing, observability, multi-tenancy, reliability]
updated: 2026-07-05 (3 sources)
---

# Inference Control Plane

A unified system layer that manages all operational decisions for an inference fleet — routing, scheduling, autoscaling, cost optimization, observability, policy, reliability, admission control, and multi-tenancy — and directs them at the underlying resources: GPU fleet, models, and applications.

> *Models become resources. The control plane decides how they are used.*
> — Gupta & Ahuja, AIE 2026

## How different speakers framed it

**Gupta & Ahuja (Meta)** presented it as an explicit architectural target: the next step after "better serving," analogous to the Kubernetes control plane for containers. Responsibilities include: routing, scheduling, autoscaling, cost optimization, observability, policy, reliability, admission control, and multi-tenancy. The plane sits between applications and the GPU fleet, and between applications and the model registry.

**Lao & Zhang (OpenAI)** presented the most detailed production instantiation: the **Inference Load Balancer (ILB)**, which separates a control plane (global view, async, runs an Optimizer) from a data plane (local, synchronous, zero-wait per request). The control plane ingests engine signals, network overhead, and TTFT/TBoT regression models, then publishes routing weight matrices (per CPU cluster → per GPU engine). The data plane caches those weights locally and uses them for every request with no blocking on the control path. Routing signals include TBoT, TTFT, load, network overhead, error rate, and — notably — KV cache match. The Optimizer frames the problem as a constrained minimization: minimize expected end-to-end latency subject to (a) all demand routed, (b) engine capacity not exceeded, (c) weights non-negative. See [Routing LLM Inference in Production](../talks/day4-1110-routing-llm-inference-in-production.md).

**Adam Azzam (Modal)** framed a related concept as the **control plane / data plane split** for agent environments: the control plane handles orchestration, policy, and lifecycle; the data plane handles execution. He argued that conflating the two is how agents end up with too much ambient authority. See [Agent Environment Architecture](agent-environment-architecture.md).

## Key idea: models as managed resources

The framing shift from Gupta & Ahuja: models are not endpoints you call — they are **resources** managed by a control plane, the same way VMs are managed by a hypervisor or pods by a scheduler. The control plane decides placement, quota, fallback, and retirement.

## Responsibilities

- **Routing** — model/region/capacity selection based on SLA + cost
- **Scheduling** — batching, GPU placement, workflow-aware decisions
- **Autoscaling** — elastic scale-out/-in based on queue depth, TTFT, and throughput targets
- **Cost Optimization** — bin-packing, spot/preemptible usage, model tier selection
- **Observability** — continuous telemetry → analysis → decision loop
- **Policy** — tenant quotas, safety guardrails, admission rules
- **Reliability** — cascade prevention, circuit breaking, fallback chains
- **Admission Control** — prioritize, defer, or reject under load
- **Multi-Tenancy** — per-tenant queue depth, cost attribution, isolation

## Relationship to other concepts

- [GPU Scheduling](gpu-scheduling.md) — one of the core responsibilities the control plane owns
- [Model Gateway](model-gateway.md) — the gateway is the entry point; the control plane is what acts on the signal from the gateway
- [Model Routing](model-routing.md) — model selection (which model) is distinct from engine routing (which replica); both are control-plane responsibilities
- [Agent Environment Architecture](agent-environment-architecture.md) — Azzam's control/data plane split is a related pattern at the agent harness level
- [Guardrails](guardrails.md) — safety policy enforcement is one of the control plane's responsibilities

## Routing layers — an important distinction

The OpenAI ILB talk clarifies that there are two distinct routing problems that production systems must solve, often conflated:

1. **Engine routing** (ILB): which replica/engine of a given model serves this request? Optimized for latency, capacity, KV cache match, and geography. This is the ILB's job.
2. **Model routing**: which model handles this request at all? Optimized for quality/cost tradeoffs. See [Model Routing](model-routing.md).

A complete inference control plane needs both layers. Mixing them leads to incorrect abstractions — e.g., a "smart router" in a gateway that conflates model selection with replica selection.

## Why nearest-only engine routing fails

The OpenAI talk provides a clean illustration: GPU capacity and user demand are not geographically co-located. A CPU cluster generating 120 RPS pointed at a local engine capped at 100 RPS will drop 20 RPS if the system only routes to the nearest engine. Cross-region spillover to spare capacity (e.g., another region running at 40/80 RPS) is required — which means the control plane must have a global view of capacity across all regions, not just local state.

## Open questions

- How should the control plane represent reasoning workflow state (e.g., a multi-hop agent that is mid-chain) for scheduling decisions?
- Where does the boundary sit between the agent runtime and the inference control plane?
- The OpenAI ILB's protection mechanisms (penalties, retry-with-cap, load shedding) were named but not detailed. How are penalties computed — is it a time-decay function after an error event, or continuous from signal data?
- Does the Optimizer run as a hot loop (continuous recompute) or on a fixed cadence? How stale can routing weights be before they cause meaningful quality loss?
