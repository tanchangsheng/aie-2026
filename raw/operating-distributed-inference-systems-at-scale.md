# Operating Distributed Inference Systems at Scale

**Speakers:** Nishant Gupta (Staff Software Engineer & Tech Lead, Meta) and Naman Ahuja (Senior Software Engineer, Meta)
**Day:** Day 4 — Session Day 3
**Time:** 10:45am–11:05am
**Room:** Track 9
**Track:** Inference

**Official description:** Inference has rapidly become one of the most important infrastructure problems in modern computing. As AI systems evolve into autonomous agents with persistent memory, tool usage, and multi-step reasoning, traditional inference architectures struggle under growing demands for latency, throughput, cost efficiency, and reliability. Topics include distributed inference architectures for large-scale AI systems, GPU scheduling and elastic compute for inference workloads, multi-tenant inference infrastructure, caching, batching, latency optimization strategies, reliability and fault isolation for inference systems, observability and control loops for AI serving platforms, balancing cost, throughput, and user experience, and why inference is becoming an infrastructure orchestration problem.

**Note:** Slides captured from Act 2 onward — Act 1 / title slide not photographed.

**Slide photos:** `raw/slides/operating-distributed-inference-systems-at-scale/`

| Slide | File |
|---|---|
| Act 2 — The great inference mismatch | [IMG_8884.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8884.HEIC) |
| Act 2 — Hidden decisions behind every prompt | [IMG_8885.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8885.HEIC) |
| Act 2 — The modern inference stack | [IMG_8886.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8886.HEIC) |
| Act 2 — One prompt becomes a distributed workflow | [IMG_8887.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8887.HEIC) |
| Act 3 — Scheduling GPUs is no longer enough | [IMG_8888.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8888.HEIC) |
| Act 3 — Optimization is decision making | [IMG_8889.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8889.HEIC) |
| Act 3 — Reliability means preventing cascades | [IMG_8890.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8890.HEIC) |
| Act 3 — Observability enables intelligence | [IMG_8891.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8891.HEIC) |
| Act 3 — The impossible triangle | [IMG_8892.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8892.HEIC) |
| Act 4 — The inference control plane | [IMG_8893.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8893.HEIC) |
| Act 4 — Lessons from operating AI infrastructure | [IMG_8894.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8894.HEIC) |
| Act 4 — What's next for AI infra | [IMG_8895.HEIC](slides/operating-distributed-inference-systems-at-scale/IMG_8895.HEIC) |

---

## Act 2 · Why Traditional Serving Breaks

### The great inference mismatch

Table comparing traditional serving vs modern inference across six dimensions:

| Dimension | Traditional serving | Modern inference |
|---|---|---|
| Request shape | Stateless, ms-scale | Streaming, seconds to minutes |
| Compute unit | CPU cores | GPU + HBM + KV cache |
| Batching | Per-request | Continuous / in-flight |
| State | Cache-friendly, small | KV cache, model weights, huge |
| Scaling unit | Pods (fast) | GPUs (slow, scarce, costly) |
| Failure blast radius | One request | Entire batch mid-generation |

Bottom callout: *The bottleneck is no longer the model. The bottleneck is orchestration.*

---

### Hidden decisions behind every prompt

12 steps labeled mostly INFRA (with only 2 labeled ML) happen before any token is generated:

01 Authenticate · 02 Choose Model · 03 Select Region · 04 Admission Control · 05 Cache Lookup · 06 GPU Reservation · 07 Batch · 08 **Generate** (ML) · 09 **Safety** (ML) · 10 Logging · 11 Billing · 12 Response

Bottom callout: *Which of these are actually ML? Almost none. They're infrastructure.*

---

### The modern inference stack

Layers from top to bottom, with descriptions:

- **Application** — Product surface: chat, agents, batch pipelines
- **Agent Runtime** — Planning, tool use, memory, orchestration of multi-step reasoning
- **Inference Gateway** — Auth, quotas, safety, protocol translation, tenant isolation
- **Routing Layer** — Model / region / capacity selection based on SLA + cost
- **Caching Layer** — Prompt, prefix, KV, and response caches
- **Admission Control** — Prioritize, defer, or reject under load
- **Scheduler** — Batching, GPU placement, workflow-aware decisions
- **Serving Runtime** — vLLM, TensorRT-LLM, SGLang — continuous batching, paged attention
- **GPU Fleet** — Heterogeneous accelerators, HBM, interconnect topology

---

### One prompt becomes a distributed workflow

Left: sequential flow — Prompt → Gateway → Router → Cache → Scheduler → Serving Runtime → Response

Right: failure modes at each hop:
- **Retries** — Idempotency, backoff, budget
- **Timeouts** — Per hop and end-to-end
- **Fallbacks** — Smaller / colder / degraded model
- **Failures** — Batch peer poisoning, mid-stream drops
- **Observability** — Trace across every hop

Bottom callout: *Inference behaves like a distributed transaction.*

---

## Act 3 · Operating Inference at Scale

### Scheduling GPUs is no longer enough

Old scheduler: 2 axes — CPU, Memory. "Two axes. Bin-packing."

Modern scheduler: 7 dimensions — Model, GPU Memory, Latency SLA, Tenant, Context Size, Cost, Reasoning Workflow

Bottom callout: *Agent-aware scheduling: place work where the whole workflow will finish fastest and cheapest — not where one GPU is free.*

---

### Optimization is decision making

Four quadrants framing all optimization decisions:

- **Can we AVOID work?** → Cache (prompt · prefix · KV · response)
- **Can we SHARE work?** → Batch (continuous / in-flight batching)
- **Can we MOVE work?** → Route (region · model tier · accelerator)
- **Can we DELAY work?** → Queue (priority · admission · defer)

Bottom callout: *Every optimization is really a question about the work itself.*

---

### Reliability means preventing cascades

Cascade chain diagram: GPU failure → Retry storm → Queue growth → Timeouts → More retries → **System collapse**

Caption: *Feedback loop — small failure, big blast*

---

### Observability enables intelligence

Continuous control loop: Telemetry → Analysis → Decision → Scheduling → Measurement → (back to Telemetry)

Signals that matter:
- TTFT · time-to-first-token
- TPOT · time-per-output-token
- Batch fill rate
- KV cache hit rate
- GPU HBM utilization
- Queue depth per tenant
- Success-per-dollar
- End-to-end trace latency

---

### The impossible triangle

Triangle diagram with vertices: **Latency** (top), **Cost** (bottom-left), **Throughput** (bottom-right). A red dot in the center labeled "you are here."

Bottom callout: *Optimization is continuous, not static. Every scheduling decision moves the system somewhere inside this triangle.*

---

## Act 4 · The Future

### The inference control plane

Central component: **Inference Control Plane**

Responsibilities (left side): Routing, Scheduling, Autoscaling, Cost Optimization, Observability, Policy, Reliability, Admission Control, Multi-Tenancy

Targets (right side): GPU Fleet, Models, Applications

Bottom callout: *Models become resources. The control plane decides how they are used.*

---

### Lessons from operating AI infrastructure

1. Infrastructure bottlenecks appear before model bottlenecks
2. Elasticity beats overprovisioning
3. Scheduling decisions dominate efficiency
4. Visibility precedes optimization
5. Control loops outperform manual operations

---

### What's next for AI infra

Three-phase progression:
- **Past** → Better Models
- **Present** → Better Serving
- **Future** → Better Orchestration

Closing statement on slide: *Future will be about making infrastructure smarter*

---

## Notes

- Slides had a consistent dark/gradient design with act labels top-left and a colored bottom callout bar on most slides
- "Runlayer" branding visible on audience lanyards; possibly the speaker's company
- Act 1 slides were not photographed — unknown what was covered (likely framing / motivation)
- No speaker bio slide captured
