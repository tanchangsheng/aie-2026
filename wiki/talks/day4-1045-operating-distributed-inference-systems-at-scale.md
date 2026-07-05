---
type: talk
tags: [inference, distributed-systems, gpu-scheduling, kv-cache, admission-control, observability, reliability, multi-tenant, serving-runtime, vllm, sglang, tensorrt-llm, inference-control-plane, meta]
updated: 2026-07-05
---

# Operating Distributed Inference Systems at Scale

**Speakers:** [Nishant Gupta](../speakers/nishant-gupta.md) · [Naman Ahuja](../speakers/naman-ahuja.md) (Meta)
**Day/Time:** Day 4 — Session Day 3 · 10:45–11:05am
**Room:** Track 9
**Track:** Inference

> **Official description:** Inference has rapidly become one of the most important infrastructure problems in modern computing. As AI systems evolve into autonomous agents with persistent memory, tool usage, and multi-step reasoning, traditional inference architectures struggle under growing demands for latency, throughput, cost efficiency, and reliability. Topics include distributed inference architectures, GPU scheduling and elastic compute, multi-tenant inference infrastructure, caching, batching, latency optimization, reliability and fault isolation, observability and control loops, balancing cost/throughput/UX, and why inference is becoming an infrastructure orchestration problem.

---

## Structure

The talk was organized in 4 acts (Act 1 slides not captured):

- **Act 2** — Why Traditional Serving Breaks
- **Act 3** — Operating Inference at Scale
- **Act 4** — The Future

---

## Act 2 · Why Traditional Serving Breaks

### The great inference mismatch

Traditional serving was built for a different world. Six dimensions where inference breaks the old assumptions:

| Dimension | Traditional serving | Modern inference |
|---|---|---|
| Request shape | Stateless, ms-scale | Streaming, seconds to minutes |
| Compute unit | CPU cores | GPU + HBM + KV cache |
| Batching | Per-request | Continuous / in-flight |
| State | Cache-friendly, small | KV cache, model weights, huge |
| Scaling unit | Pods (fast) | GPUs (slow, scarce, costly) |
| Failure blast radius | One request | Entire batch mid-generation |

> **The bottleneck is no longer the model. The bottleneck is orchestration.**

### Hidden decisions behind every prompt

12 sequential steps occur before a token is generated — only 2 of them (Generate, Safety) are actually ML. The rest are infrastructure:

01 Authenticate · 02 Choose Model · 03 Select Region · 04 Admission Control · 05 Cache Lookup · 06 GPU Reservation · 07 Batch · **08 Generate (ML)** · **09 Safety (ML)** · 10 Logging · 11 Billing · 12 Response

> *Which of these are actually ML? Almost none. They're infrastructure.*

### The modern inference stack

A layered model from product surface down to hardware:

| Layer | Role |
|---|---|
| Application | Product surface: chat, agents, batch pipelines |
| Agent Runtime | Planning, tool use, memory, orchestration of multi-step reasoning |
| Inference Gateway | Auth, quotas, safety, protocol translation, tenant isolation |
| Routing Layer | Model / region / capacity selection based on SLA + cost |
| Caching Layer | Prompt, prefix, KV, and response caches |
| Admission Control | Prioritize, defer, or reject under load |
| Scheduler | Batching, GPU placement, workflow-aware decisions |
| Serving Runtime | vLLM, TensorRT-LLM, SGLang — continuous batching, paged attention |
| GPU Fleet | Heterogeneous accelerators, HBM, interconnect topology |

### One prompt becomes a distributed workflow

A prompt flows through: Gateway → Router → Cache → Scheduler → Serving Runtime → Response. At every hop there are cross-cutting failure-handling concerns:

- **Retries** — Idempotency, backoff, budget
- **Timeouts** — Per hop and end-to-end
- **Fallbacks** — Smaller / colder / degraded model
- **Failures** — Batch peer poisoning, mid-stream drops
- **Observability** — Trace across every hop

> *Inference behaves like a distributed transaction.*

---

## Act 3 · Operating Inference at Scale

### Scheduling GPUs is no longer enough

Old schedulers managed two axes (CPU, memory) with bin-packing. Modern inference schedulers must handle seven dimensions simultaneously: Model, GPU Memory, Latency SLA, Tenant, Context Size, Cost, Reasoning Workflow.

> *Agent-aware scheduling: place work where the whole workflow will finish fastest and cheapest — not where one GPU is free.*

See [GPU Scheduling](../concepts/gpu-scheduling.md).

### Optimization is decision making

Every optimization decision reduces to one of four questions about the work:

- **Can we AVOID it?** → Cache (prompt · prefix · KV · response)
- **Can we SHARE it?** → Batch (continuous / in-flight batching)
- **Can we MOVE it?** → Route (region · model tier · accelerator)
- **Can we DELAY it?** → Queue (priority · admission · defer)

> *Every optimization is really a question about the work itself.*

### Reliability means preventing cascades

The cascade failure pattern: GPU failure → Retry storm → Queue growth → Timeouts → More retries → **System collapse**. A small failure becomes a big blast through the feedback loop.

### Observability enables intelligence

Inference systems need a continuous control loop: Telemetry → Analysis → Decision → Scheduling → Measurement → (back to Telemetry).

Key signals:
- **TTFT** — time-to-first-token
- **TPOT** — time-per-output-token
- Batch fill rate
- [KV cache](../concepts/kv-cache.md) hit rate
- GPU HBM utilization
- Queue depth per tenant
- Success-per-dollar
- End-to-end trace latency

> *Visibility precedes optimization.*

### The impossible triangle

Every scheduling decision moves the system somewhere inside a triangle with three competing vertices: **Latency**, **Cost**, **Throughput**. You are always at a point inside this triangle, never at a corner.

> *Optimization is continuous, not static. Every scheduling decision moves the system somewhere inside this triangle.*

---

## Act 4 · The Future

### The inference control plane

The future architecture is a unified **Inference Control Plane** that manages: Routing, Scheduling, Autoscaling, Cost Optimization, Observability, Policy, Reliability, Admission Control, Multi-Tenancy — and directs them at three targets: GPU Fleet, Models, Applications.

> *Models become resources. The control plane decides how they are used.*

See [Inference Control Plane](../concepts/inference-control-plane.md).

### Lessons from operating AI infrastructure

1. Infrastructure bottlenecks appear before model bottlenecks
2. Elasticity beats overprovisioning
3. Scheduling decisions dominate efficiency
4. Visibility precedes optimization
5. Control loops outperform manual operations

### What's next for AI infra

Three-phase progression: **Better Models** (past) → **Better Serving** (present) → **Better Orchestration** (future).

> *Future will be about making infrastructure smarter.*

---

## Slide photos

`raw/slides/operating-distributed-inference-systems-at-scale/` (IMG_8884–IMG_8895, HEIC). Act 1 slides not captured.
