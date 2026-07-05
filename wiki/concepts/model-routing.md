---
type: concept
tags: [model-routing, inference-cost, kv-cache, coding-agents, cost-optimisation, agents, software-factory]
updated: 2026-07-05 (2 sources)
---

# Model Routing

## Definition

Model routing is the practice of dynamically selecting the best-suited model (and reasoning effort level) for each individual request, rather than using a single model for every request. A router sits between the application/harness and the model gateway, making a per-request or per-turn selection from a pool of candidate models.

> "A gateway gives you access to models. A router determines which one to use." — Not Diamond blog

## Why It Matters

The fundamental economics of routing: most workloads contain a long tail of simple requests that don't need the strongest model in the pool. Routing simple steps to cheaper models while reserving expensive models for steps that genuinely need them closes the gap between actual spend and an optimal policy.

The problem is acute for **coding agents** specifically:
- A single Claude Code session can consume >1M tokens/minute for hours
- More capable frontier models enable longer-running sessions, compounding cost
- Cost surfaces multiply: model choice, reasoning effort, KV cache economics, context pressure, pricing changes, and session length all contribute independently
- The landscape changes weekly as new models ship

## Routing vs. Gateway

These are distinct layers that are often confused:

| | Gateway | Router |
|---|---|---|
| **Function** | Access layer — unified API, auth, billing consolidation, API normalisation | Decision layer — which model handles this request |
| **Decision** | None — model choice is hardcoded by the caller | Dynamic per-request selection from a pool |
| **Value** | Consolidation (N integrations → 1) | Optimisation (spend ≈ what each request actually needs) |
| **Examples** | Uber's Model Gateway, LiteLLM, OpenRouter | Not Diamond, custom RL policies |

Production stacks need both. A gateway without a router hard-codes a single model per endpoint. A router without a gateway still needs an access layer.

See also: [Model Gateway](model-gateway.md)

## Routing Approaches

From least to most sophisticated:

### 1. Heuristic routing
Hard-coded rules on surface features (keyword match, prompt length, regex). Fast to set up; brittle; doesn't scale; misses semantic nuance.

### 2. Semantic routing
Incoming prompt is embedded and cosine-matched against example phrases indexed at setup time. More robust than keyword matching; limited by quality of example set; struggles with multi-turn context where the salient intent isn't in the latest message.

### 3. Complexity classifier
Trained classifier estimates prompt difficulty; routes hard prompts up, easy prompts down. Faster and cheaper than LLM-based routing; difficulty is a coarse signal (doesn't capture domain specialisation); requires retraining as model capabilities evolve.

### 4. LLM-based routing
A small LLM classifies the prompt and routes it. Flexible and nuanced, but adds a full inference call to every request — often self-defeating unless the classifier is heavily finetuned for scale.

### 5. Cascade routing
Send to cheapest model first; escalate to a stronger model if a verification check fails. Conservative on cost; requires a verification mechanism cheaper than the cheap model itself; sequential calls add latency — unsuitable for latency-sensitive applications.

### 6. Predictive / learned routing
A model trained on benchmarks and production traffic predicts how each candidate model will perform on this specific prompt, then picks the best for the objective (quality, cost, latency, or a blend). Most powerful approach; requires training signal and a way to measure success. This is Not Diamond's core approach.

### 7. Session-based / RL routing (agent-native)
Treats routing as a reinforcement learning problem: the routing policy selects from an action space of (model × reasoning effort) pairs, with the KV cache state, session state, and feedback signals as environment. Appropriate for long-horizon, sparse-reward agentic workloads where per-turn context accumulates. See [Agent-Native Routing](#agent-native-routing) below.

In practice, production routers are compositional — e.g. a cascade layered on top of a predictive classifier.

## Agent-Native Routing

For coding agents, routing must handle concerns that don't exist in single-turn chat:

**1. KV-cache economics**  
Switching models mid-session invalidates the cached prompt prefix. A cache read costs ~10% of the uncached input price; a bad switch can cost more than staying on the expensive model. A cache-unaware router can lose all of its paper savings on a real session. The router must track:
- Cache warmth (is the cache actually live?)
- TTL expiry (commonly 5 minutes for most providers)
- Context compaction events (which reset the prefix)
- Media attachments and other prefix-invalidating changes

**2. Long horizons and variable complexity**  
A single coding session moves through planning, code generation, debugging, and summarisation steps — each with different model requirements. A stateless per-request router applies a single policy across these; an RL-based session router can adapt.

**3. Sparse rewards**  
Success signals in agentic workloads are delayed (did the task complete correctly?) rather than immediate. The RL loop:

```
[Routing policy] → [Action: model + reasoning effort] → [Environment: session state, KV cache state, feedback]
                 ←──── Reward ──────────────────────────────────────────────────────────────────────────────
                 ←──── State ───────────────────────────────────────────────────────────────────────────────
```

**4. Multi-level routing**  
Optimal routing in agentic systems operates at multiple levels: session level (anchor model for the whole session), sub-agent level (each spawned worker has its own context), task level (planning vs. code-gen vs. summarisation), and step level (within a task, individual steps vary).

## Results

On Terminal-Bench / Claude Code (Anthropic models only):

| Model | Cost/attempt | Accuracy |
|---|---|---|
| Claude Haiku 4.5 Low | ~$0.50 | ~30% |
| Claude Sonnet 4.6 High | ~$1.50 | ~50% |
| Claude Opus 4.8 high | ~$2.00 | ~60% |
| Claude Opus 4.8 xhigh | ~$2.20 | ~75% |
| **Not Diamond (routed)** | **~$1.00** | **~75%** |

Claimed savings across production workloads: 20–95% on inference costs. F500 engineering teams: 30%+ at frontier quality.

## Architecture Pattern (Not Diamond)

Privacy-preserving: routing decisions are based on **derived, schematized metadata**, not raw prompt content. Raw prompts stay local on the developer's machine; only metadata crosses to the cloud optimisation server.

```
Dev local: [Coding agent harness] ◄──► [ND agent: metadata extraction, local cache management]
                                                │
                                        Client gateway: [Model 1] [Model 2]
                                                │
Not Diamond cloud: [ND Optimization Server: selects optimal model] → [OSS model / ND inference]
                   [Cost dashboard: account, usage, billing]
```

Compliance: ISO 27001, AICPA SOC 2.

## Open Questions

- How does the RL routing policy handle exploration vs. exploitation in production — does a live system ever intentionally route to a suboptimal model to gather signal?
- What exactly is in "derived, schematized metadata"? Does it include task type, prompt length, tool schemas, or something more?
- At what session length / pool composition does cascade routing become cache-uneconomical?
- How quickly can the learned policy adapt to a newly released model with limited benchmark coverage?
- The Terminal-Bench result used Anthropic models only — what accuracy/cost curve emerges when OSS models (DeepSeek, Qwen, etc.) are added to the pool?

## Factory Router (Software Factory Implementation)

Tížková describes Factory's classifier-based router as a second concrete implementation. When an engineer describes a task, the classifier scores: message content, recent tool calls, repo size, language mix, and difficulty — assigns each model a quality-probability score and picks the cheapest predicted to clear a configurable quality threshold. Organizations set different default models per role (sales, marketing, engineers).

Key differences from Not Diamond's approach:
- Quality threshold is set explicitly per deployment (rather than optimizing for a cost/quality Pareto frontier)
- The router can upgrade mid-conversation when a task turns out harder than expected (Not Diamond also supports dynamic escalation, but Tížková emphasizes this as a trust mechanism)
- ~25% benchmark savings (conservative estimate); similar order of magnitude to Not Diamond's 20–30% floor

The Factory Router also factors into the **caching economics**: routing a task to a new model mid-session breaks the cached prompt prefix, making the switch cost more than staying on the expensive model. This is the same cache-aware routing concern Not Diamond raises, suggesting it's an industry-wide constraint rather than either company's unique finding.

See also: [Software Factory](software-factory.md)

## Sources

- [Intelligent Model Routing: Frontier Performance Without Frontier Bills](../talks/day3-1450-intelligent-model-routing.md) — Tomás Hernando Kofman (Not Diamond)
- [Rise of the Software Factory](../talks/day2-1110-rise-of-the-software-factory.md) — Tereza Tížková (Factory)
