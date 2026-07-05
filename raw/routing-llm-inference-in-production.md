# Routing LLM Inference in Production: From Engine Signals to Policy

**Speakers:** Qianru Lao (Inference, OpenAI), Lu Zhang (Inference, OpenAI)  
**Org:** OpenAI  
**Slides:** [slides/routing-llm-inference-in-production/](../slides/routing-llm-inference-in-production/)

---

## Slide 1 — Title

"Routing LLM inference in production: From engine signals to policy"

![Slide 1](../slides/routing-llm-inference-in-production/IMG_8896.HEIC)

---

![Slide 2](../slides/routing-llm-inference-in-production/IMG_8897.HEIC)

## Slide 2 — What is Inference Load Balancer

Architecture overview: Incoming request → Frontend clusters → **Inference Load Balancer** (Select an Inference Engine → Proxy the request) → Engine clusters

**Frontend clusters:**
- CPU clusters
- Processes user request
- Forms into inference request
- Sends to backend engines through Inference LB

**Select an Inference Engine (the core routing problem):**
- A model may have multiple engines behind it
- Select the best one — what to consider?
- TBoT, TTFT, load, network overhead, error rate
- But also: KV match

**Engine clusters:**
- GPU clusters
- Each hosts various inference engines

---

![Slide 3](../slides/routing-llm-inference-in-production/IMG_8898.HEIC)

## Slide 3 — Load Balancing in the "early days": A periodic feedback loop (diagram)

Flow:
INFERENCE REQUESTS → ELIGIBILITY FILTERING (e.g., capability) → WEIGHTED CONSISTENT HASHING SELECTION → INFERENCE ENGINES

Feedback loop: PERIODIC SIGNAL UPDATES (EWMA / smoothing, PID-like weight update) feeds back into the weighted consistent hashing selection step.

---

![Slide 4](../slides/routing-llm-inference-in-production/IMG_8899.HEIC)

## Slide 4 — Load Balancing in the "early days": Pros and cons

**Pros:**
- Many signals considered
- Routing restrictions naturally balanced out
- Self-adaptive, like a "PID controller"
- It "just works"

**Cons:**
- Hard to reason about routing decisions or control
- Load not always evenly distributed, especially for a mixed fleet
- Feedback loop creates slow oscillations, impacting KV cache hit rate

---

![Slide 5](../slides/routing-llm-inference-in-production/IMG_8900.HEIC)

## Slide 5 — Which GPU engine should serve each request from a CPU cluster?

Subtitle: Input: CPU cluster + model. Output: one eligible engine in a GPU cluster.

Architecture:
- **Control plane** — Global view for routing optimizations (top level)
- Multiple CPU clusters (1 to N), each with a Local data plane
- All CPU clusters route down to: GPU engines across GPU clusters (engine 1 … engine N)

---

![Slide 6](../slides/routing-llm-inference-in-production/IMG_8901.HEIC)

## Slide 6 — Separated control plane and data plane

Legend: Hot request path (sync), Async feedback, Async routing weights pull

**Control plane:**
Engine signals → Data loader → Optimizer → Publish routing weights
(Data loader also takes in: Network overhead; Capacity + TTFT/TBoT regressions)

**Data plane:**
- Async refresh of candidate engines
- Local routing state: candidate engines + routing weights
- Requests from CPU clusters → Engine selector → GPU fleet → Engine signals (async feedback back to control plane)

Key design principle: **"No request waits on the data plane."**

---

![Slide 7](../slides/routing-llm-inference-in-production/IMG_8902.HEIC)

## Slide 7 — Three paths through the system

Subtitle: One synchronous request path; two asynchronous control paths.

1. **Inference request path** (synchronous, per request):  
   Request from CPU cluster → Engine selection + route → GPU engine serve request

2. **Engine signal path** (async feedback):  
   GPU engine signals → Aggregate signals → Control/data plane ingest

3. **Routing weight path** (async pull):  
   Control plane computed routing weight → Publish routing weights → Data plane weights updated

---

![Slide 8](../slides/routing-llm-inference-in-production/IMG_8903.HEIC)

## Slide 8 — Why nearest-only fails

Subtitle: Demand and GPU capacity are not geographically balanced.

Example with 3 regions:
- Region 1: Engine A (100 RPS cap) ↔ CPU Cluster A (90 RPS) — fine, 10 RPS headroom
- Region 2: Engine B (100 RPS cap) ↔ CPU Cluster B (120 RPS) — at capacity, 20 RPS spillover needed
- Region 3: Engine C (80 RPS cap) ↔ CPU Cluster C (40 RPS) — 40 RPS spare

The 20 RPS spillover from Region 2 routes to Region 3's spare capacity. Nearest-only would fail Region 2 requests; you need cross-region routing awareness.

---

![Slide 9](../slides/routing-llm-inference-in-production/IMG_8904.HEIC)

## Slide 9 — Optimizer: From signals to weights

Subtitle: Compute how each CPU cluster should route traffic across engines.

**Inputs to the Optimizer (traffic allocation):**
- Request from CPU cluster (demand)
- Network latency
- Engine capacity + health
- TTFT / TBoT latency profiles

**Output:** Routing weights

**Optimization goal:**  
Minimize expected end-to-end latency across all routed traffic. Balance network distance with engine-side latency.

**Hard constraints:**
- Route all demand
- Stay within effective engine capacity
- Keep routing weights non-negative

---

![Slide 10](../slides/routing-llm-inference-in-production/IMG_8905.HEIC)

## Slide 10 — Protection mechanisms

Three mechanisms listed:
1. **Penalties**
2. **Retry with cap**
3. **Load shedding**

(No further detail given on the slide — likely covered verbally)
