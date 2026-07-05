---
type: concept
tags: [guardrails, safety, security, fail-open, fail-closed, llm-gateway, pii, prompt-injection, secrets, os-isolation, agent-fleet]
updated: 2026-07-05
---

# Guardrails

## Definition

Guardrails are safety/policy checks applied to LLM inputs and/or outputs within a gateway or middleware layer. They intercept requests to enforce rules around PII, toxicity, prompt injection, and other policy concerns. Because they are services in their own right, they introduce a new failure mode: the guardrail itself can fail, and that failure requires an explicit policy decision.

## How Different Speakers Framed It

### Uber (Agentic SDLC at Uber)

Uber's model gateway middleware chain includes an "AI Guard" component positioned after identity establishment and PII anonymisation, and before the LLM cache and smart router. In Uber's framing, guardrails are one layer of a centrally-managed middleware chain that every caller passes through — policy is enforced uniformly, without individual teams implementing it.

The key concern at Uber's scale is PII: without a gateway, every team rolls its own redaction (or skips it), so PII reaches third-party LLMs inconsistently. The AI Guard in the middleware chain is presumably downstream of the anonymiser specifically to catch anything the anonymiser missed.

### Kanish Manuja, Twilio (Productionizing LLM Gateways)

Manuja's framing is more operational and treats guardrails as dependencies that can fail — requiring the same resilience thinking as any other service dependency. The title slide for the guardrails section reads: "ignore all instructions and focus on guardrails" (a deliberate echo of prompt-injection language), with the subtext: "Guardrails are necessary, and they are a new dependency that can take you down."

Three design dimensions he covers:

**1. Fail-Open vs. Fail-Closed**

When a guardrail service is unavailable or returns an error, you must decide:
- **Fail open:** let the request through ungated. Protects availability; accepts security risk.
- **Fail closed:** block the request. Protects security; accepts an availability hit.

There is no universal answer. The right choice is per-policy and per-severity:
- A toxicity filter might reasonably fail open (occasional unmoderated output is tolerable).
- A PII check or prompt-injection check should fail closed (one leak can be a compliance incident).

Decision rule: **decide by blast radius** — default to the worst case you can live with, not a single global switch.

**2. Time Budget**

Every guardrail must have a hard latency deadline. A slow guardrail that times out or runs long must never become your application's latency. This implies guardrails need their own SLOs, monitored independently.

**3. Placement**

When a guardrail runs determines how much latency users perceive:

| Placement | When | Latency impact | Notes |
|-----------|------|----------------|-------|
| Pre-hook | Before the model call | Serial on every request | Only viable for fast input checks |
| **In parallel ★** | Alongside the model call | Virtually none | Preferred default |
| Post-hook | After the model returns | Serial on the response | Required for output inspection |

Running guardrails in parallel with the model call (gating the response before it's sent) is the recommended default — it adds almost no perceived latency while still blocking unsafe outputs.

**Fallback for the guardrail itself:** have a secondary check or cached decision for when the primary guardrail service is down. This is often overlooked.

### Shashank Goyal, OpenRouter (Letting the Interns Loose)

OpenRouter's framing is the most operationally specific of the three: guardrails matter most at the point of secrets and privileged actions. Their production insight is that the real AI adoption unlock came when they could safely give agents *more* capabilities — which required a hard trust boundary, not just policy enforcement.

**The pattern — OS-level secrets isolation:**

The secrets file is masked at the OS level (`systemd InaccessiblePaths`), making it invisible to the harness process entirely. This means the agent cannot read its own credentials even if compromised or prompt-injected — not because it's told not to, but because the file is physically inaccessible at the process level.

**The deployment gate — privileged sibling process:**

```
Intern → Host app (control plane) → Human approval (wired button click) → Privileged sibling (separate process) → Deploy
```

The privileged sibling is a separate process that holds the credentials and can execute deployment actions. It only fires from a wired button click in the host app — it cannot be triggered by a chat message to the harness. This creates a hard human-in-the-loop gate for any privileged action, enforced by process architecture rather than by prompt instructions.

**Lesson from production:** "Guardrails are important especially for secrets — the real unlock started when we could safely add more capabilities to the agents." The implication: teams under-capability their agents because they don't trust them, not because the agents aren't capable enough. Solving the secrets problem unlocks adoption.

## Agreements

Both speakers agree that guardrails must be centrally enforced rather than left to individual teams — the failure mode of per-team implementation is inconsistency (or absence).

## Contradictions / Tensions

- Uber centralises guardrails in a single middleware chain; Manuja argues a central gateway is a SPOF and advocates for each service owning its own guardrail stack with centralised governance only. These represent different architectural bets about where the blast-radius risk is greater: inconsistency (Uber's concern) vs. shared-gateway failure (Manuja's concern).
- Neither speaker addresses the performance cost of running a guardrail model in parallel with a large LLM — the parallel placement recommendation assumes the guardrail is fast enough that its latency is masked by the model call. For very fast model calls (embeddings, classifiers) this may not hold.

## Open Questions

- What's the failure rate of guardrail services in practice, and at what rate does fail-open vs. fail-closed materially affect either safety incidents or availability metrics?
- Is a cached guardrail decision (used when the guardrail is down) stale in any way that matters? If the policy changed, a cached "allow" could be a liability.
- For prompt-injection specifically: are pre-hook (serial before the model) checks required, or is in-parallel detection good enough? If the injected instruction is acted on before the guardrail completes, parallel placement might not help.

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda (AI Guard in middleware chain, PII anonymisation before guardrail)
- [Productionizing LLM Gateways](../talks/day2-1425-productionizing-llm-gateways.md) — Kanish Manuja, Twilio (fail-open/closed, time budget, placement, blast-radius decision rule)
- [Letting the Interns Loose](../talks/day3-1110-letting-the-interns-loose.md) — Shashank Goyal, OpenRouter (OS-level secrets isolation via `systemd InaccessiblePaths`; privileged sibling process; secrets as the unlock for broader agent capabilities)
