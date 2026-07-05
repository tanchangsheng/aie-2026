---
type: concept
tags: [agentic-sdlc, model-gateway, governance, pii, cost-attribution, llm-infrastructure, availability, latency, guardrails, fallback, circuit-breaker]
updated: 2026-07-02
---

# Model Gateway

## Definition

A Model Gateway is a single governed entry point between an organization's internal callers (agent harnesses, managed agents, user-facing AI products) and external/internal LLM providers. It centralizes identity, PII redaction, safety checks, caching, routing, cost attribution, and audit logging so individual app teams don't each reimplement them.

## What It Solves

- **PII leakage** — without a gateway, every app team rolls its own redaction (or skips it), so PII reaches third-party LLMs inconsistently.
- **Latency from safety checks** — running a full moderation model per request is expensive; a gateway can centralize and optimize this once.
- **Scattered cost attribution** — spend across providers, teams, and projects is otherwise impossible to meter centrally.

## Architecture (Uber's Implementation)

```
Callers → Middleware → Management → Observability → Providers
```

- **Middleware chain:** identity (SPIFFE/SPIRE) → data anonymizer → AI Guard (safety) → LLM cache → smart router
- **Management layer:** project catalog, per-caller/per-user attribution, spend tiers
- **Observability layer:** audit log, session traces, cost reconciliation
- **Providers fronted:** OpenAI, Anthropic, Google, and OSS models — callers don't need provider-specific integration

The middleware ordering is notable: identity is established before anonymization, and anonymization happens before the request ever reaches a safety/guard model or an external provider.

## Why a Gateway Instead of Per-Team Integration

The gateway pattern trades a small amount of indirection for organization-wide consistency: one place to update redaction logic, one place to see total spend, one place to enforce identity-based access — instead of N teams each making (and likely missing) these decisions independently.

## Gateway vs. Router: Clarified

A Day 3 talk by Not Diamond directly addresses the open question below about Uber's "smart router." The distinction matters:

- **Gateway** = access layer (auth, PII redaction, safety, API normalisation, billing). The gateway doesn't decide which model to use.
- **Router** = decision layer (which model handles this specific request, based on content and context). The router sits *inside* the middleware chain — it is a component of the gateway, not a replacement for it.

Uber's architecture has both: the gateway handles identity, anonymization, safety, and cost attribution; the "smart router" inside the middleware chain is (presumably) a model routing policy. See [Model Routing](model-routing.md) for what sophisticated routing looks like.

## The Four-Axis Tradeoff Space (Twilio's Framing)

A Day 2 talk by Kanish Manuja (Twilio) adds a production-engineering lens to the gateway concept that's absent from Uber's platform-architecture framing. His central claim: a gateway is where four forces fight — **availability, latency, guardrails, and cost** — and every safeguard buys you one axis while charging another. You cannot max all four simultaneously.

This four-axis model is more precise than a simple "gateway good" framing: it forces teams to decide *which axis to trade away* before they build anything.

### Availability: Eject, Don't Retry

Standard microservice patterns (retries, circuit breakers) fail for LLMs in three specific ways:
1. Retrying the same provider during its outage burns latency budget rather than recovering.
2. A tripped breaker fails the request even when a secondary was available the whole time.
3. LLM calls are slow and expensive; blind retries multiply cost and tail latency.

The better pattern: a **per-request fallback with an ejecting breaker**. When the breaker opens, the primary is removed from the routing path entirely — requests go straight to secondary, no wasted attempt. State can be fleet-wide (consistent, needs shared infra) or local (simpler, but each instance must observe failures independently).

Three production complications the clean diagram doesn't show:
- **Fallback is not transparent:** providers differ in tool schemas, token limits, and stop reasons. Without a normalisation layer, the response shape shifts under the application on every fallback.
- **Streaming removes levers:** once the first token is sent, you can no longer fall back, retry, or rewrite. Only stream when latency demands it.
- **Fallback capacity is a blind spot:** teams track primary throughput but not secondary, creating an undersized fallback.

### Latency: Track Per Route, Not Gateway-Wide

A gateway running mixed workloads (embeddings at 0.3s p99, reasoning at 38s p99) produces a blended p99 that is useless for alerting — the blended number masks that chat is in outage territory if it hits the same number a reasoning model considers normal. Fix: track p99 per model and per route; set timeouts per model class.

Reasoning and "router" models make latency genuinely unpredictable: the same prompt may take 2s or 60s, and when a provider's own router picks the underlying model, the observed latency is an unmeasurable mixture. Two mitigations: pin a static reasoning level per route, and hedge the tail by firing a backup request after a short delay.

### Cost: Isolate Tenants, Bound Queues

Shared rate limits let one noisy tenant starve others. Isolate usage with separate API keys per team or workload — the simplest path to cost attribution and blast-radius control. Under resource pressure, shed load early with **bounded queues**: an unbounded queue combined with a retry storm is a disaster; a bounded queue that sheds past its limit returns fast 429s and keeps the system stable.

### Architecture Conclusion: Decentralised Resilience, Centralised Governance

Centralising fallback logic, guardrails, and routing in a single process concentrates blast radius: one bad deploy or shared breaker state can take every workload down simultaneously. The recommended split:
- **Decentralise the gateway:** each service owns its own fallback, timeouts, and load shedding.
- **Centralise governance only:** cost attribution and guardrail-failure events flow to a shared control plane without routing traffic through a single hop.

This is a materially different topology from Uber's architecture above, which centralises the middleware chain. The Twilio framing is more appropriate for multi-team organisations where a shared gateway becomes a SPOF; Uber's framing optimises for consistent policy enforcement across a very large engineering org. Both are valid trade-offs.

## Open Questions

- What's the latency overhead of the middleware chain (anonymizer + AI Guard + cache lookup) versus calling a provider directly?
- ~~How is the "smart router" choosing between providers/models?~~ Answered by Day 3 Not Diamond talk: sophisticated routing uses a learned RL policy; simpler implementations use heuristic or complexity-classifier approaches. Uber's specific implementation remains unconfirmed.
- Does centralized caching (LLM Cache in the middleware) cause staleness issues for time-sensitive queries?
- Is Uber's smart router cache-aware? Given their scale, ignoring KV cache economics at the router layer would be expensive.

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda
- [Intelligent Model Routing: Frontier Performance Without Frontier Bills](../talks/day3-1450-intelligent-model-routing.md) — Tomás Hernando Kofman (gateway vs. router distinction)
- [Productionizing LLM Gateways](../talks/day2-1425-productionizing-llm-gateways.md) — Kanish Manuja, Twilio (four-axis tradeoffs, availability/latency/cost patterns, decentralised architecture)
