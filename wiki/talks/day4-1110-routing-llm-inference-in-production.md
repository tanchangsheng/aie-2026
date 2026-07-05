---
type: talk
tags: [inference, load-balancing, routing, gpu, kv-cache, production-systems, openai]
updated: 2026-07-05
---

# Routing LLM Inference in Production: From Engine Signals to Policy

**Speakers:** [Qianru Lao](../speakers/qianru-lao.md), [Lu Zhang](../speakers/lu-zhang.md)  
**Org:** OpenAI  
**Date:** July 2, 2026 (Day 4 — Session Day 3)  
**Time:** 11:10am–11:30am  
**Track:** Inference  
**Room:** Track 9  
**Raw notes:** [raw/routing-llm-inference-in-production.md](../../raw/routing-llm-inference-in-production.md)

## Official description

Production LLM apps need more than a fast model: they need an inference routing layer that can choose where each request should run as engines, capacity, latency, and geography cost change. This talk shares a generalized Inference Load Balancer (ILB) proxy/controller architecture. A low-latency proxy applies routing weights and request-path signals, while a controller computes source-cluster-to-engine weights from demand, capacity/performance profiles, replica state, and geography cost. It also covers practical debugging patterns: reading engine signals, explaining why a request went to one backend instead of another, handling retries and load shedding, and keeping routing behavior observable.

## Architecture overview

### Inference Load Balancer (ILB)

An ILB sits between frontend CPU clusters and backend GPU engine clusters. Its job for each request: select the best engine, then proxy the request.

**Frontend clusters** (CPU): parse and validate the user request, form it into an inference request, dispatch it through the ILB.

**Engine clusters** (GPU): host the actual model replicas. A single model may have many engines across multiple clusters and regions.

**Routing signals considered:** TBoT (time between output tokens), TTFT (time to first token), load, network overhead, error rate, and KV cache match.

### Early approach: periodic feedback loop

Flow: `Inference requests → Eligibility filtering (capability) → Weighted consistent hashing → Inference engines`

Signals updated periodically via EWMA smoothing + PID-like weight updates.

**Pros:** many signals considered, self-adaptive, routing restrictions balance out naturally, "just works."

**Cons:** hard to reason about or control decisions; uneven load on mixed fleets; slow oscillations in the feedback loop hurt KV cache hit rate.

### Evolved architecture: separated control plane and data plane

**Control plane** (async, global view):
- Ingests engine signals
- Loads network overhead and capacity/TTFT/TBoT regression models
- Runs the Optimizer to compute routing weights per CPU cluster → engine
- Publishes routing weights

**Data plane** (synchronous, per-request, no waiting):
- Locally caches candidate engines + routing weights (refreshed async)
- Engine selector picks an engine and proxies the request
- Engine signals flow back async to the control plane

Key principle: **no request waits on the data plane.**

### Three paths through the system

1. **Inference request path** (sync, per-request): CPU cluster request → engine selection + route → GPU engine serves
2. **Engine signal path** (async feedback): GPU engine signals → aggregate → control/data plane ingest
3. **Routing weight path** (async pull): control plane computes weights → publishes → data plane updates locally

### Why nearest-only routing fails

GPU capacity and user demand are not geographically co-located. Example: Region 2 CPU cluster generates 120 RPS but its local engine caps at 100 → 20 RPS must spill to Region 3 (which has 40 RPS of spare capacity). Nearest-only would simply fail those requests.

### The Optimizer

Inputs: per-CPU-cluster demand, network latency, engine capacity + health, TTFT/TBoT latency profiles.  
Output: routing weight matrix (CPU cluster → engine).

**Goal:** minimize expected end-to-end latency across all traffic, balancing network distance against engine-side latency.

**Hard constraints:** route all demand; stay within effective engine capacity; keep weights non-negative.

### Protection mechanisms

- **Penalties** — down-weight engines that are misbehaving
- **Retry with cap** — retry on failure, but bound total retries
- **Load shedding** — drop requests when the system is overwhelmed

## Key claims

- A periodic, PID-like feedback loop is a reasonable starting point but creates oscillations that hurt KV cache hit rate at scale.
- Separating control plane (global, async optimization) from data plane (local, sync routing) eliminates per-request latency from routing decisions.
- "Nearest" routing is insufficient — the optimizer must consider cross-region spillover because demand and capacity are geographically imbalanced.
- KV cache match is a first-class routing signal alongside latency and load.
- No request should ever wait for a routing decision; all weight computation happens off the hot path.

## Reactions

The control/data plane separation is clean and mirrors what you see in SDN and service mesh design — applying it to LLM inference is the right call. The "PID controller" framing of the early approach is a nice way to describe why it works but why it's hard to reason about. The optimizer formulation (minimize E2E latency subject to capacity and non-negativity constraints) is elegant and tractable.

The talk is notably light on the protection mechanisms slide — Penalties / Retry with cap / Load shedding are named but not explained, likely covered verbally or in a follow-up talk. Worth digging into how penalties are computed and when load shedding triggers.

KV cache hit rate as a routing signal is interesting and worth tracking across talks — it suggests routing and caching are increasingly co-designed at scale.
