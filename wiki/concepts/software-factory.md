---
type: concept
tags: [software-factory, autonomous-agents, agentic-sdlc, agent-readiness, validation, enterprise, human-verdict]
updated: 2026-07-05 (3 sources)
---

# Software Factory

## Definition

A software factory is the full autonomous cycle of developing software: spec, build, validate, deploy, and learn — with the learning feeding back into the next cycle. It is not a coding agent, and not even a swarm of coding agents. The better mental model is a factory floor: parallel lines handling different tasks at different autonomy levels, with policy and priorities set at the top and the floor reporting back up.

> The factory does not come for the interesting work; it comes for the annoying work — the alignment, the status updates, the meetings that exist only to move context between people.
> — Tereza Tížková, "Rise of the Software Factory"

## Why It Wasn't Possible Before 2025

Each precondition had to arrive in sequence before the factory became real:

| Year | Layer |
|------|-------|
| 2021 | Copilot — autocomplete |
| 2022 | ChatGPT — conversation |
| 2023 | GPT-4 + larger context — real code generation; early loops (AutoGPT, BabyAGI) existed but failed (hallucination, context limits, no isolated envs) |
| 2024 | Better reasoning — multi-step tasks |
| 2025 | Persistent environments + long-running missions — genuine autonomy |

Task-length doubles roughly every 7 months, but only at 50% reliability. The real frontier is now **reliability over long horizons**, not raw capability.

## How Different Speakers Framed It

### Tereza Tížková — Factory (Rise of the Software Factory, Day 2)

Three properties every software factory must have:

**1. Agnostic** — not locked to a single model or platform. The same agent runs in the cloud, locally, in CI, and in the IDE. Model routing and prefix caching are the two implementation levers: routing picks the cheapest model that clears a quality threshold; caching makes repeated context ~10× cheaper after the first turn.

**2. Autonomous** — a three-role architecture of orchestrator + workers + validators, bound by a validation contract defined before any code is written. Workers run in sequence (not as a parallel swarm) so each starts with fresh context. Two classes of validator: Scrutiny (reads code, runs tests/lint/typecheck/review, but never wrote the code) and User-Testing (black box — drives the running app through computer-use, never reads source).

Key stat: in one real mission — 16.5 hours, 185 agent runs — validation consumed ~40% of total process time. That's a feature, not a bug.

**3. Always-improving** — agent results are an environment problem, not a model problem. Repos with basic structure (AGENTS.md, linting config, pre-commit hooks) show 3–4× better metrics after AI adoption than repos without. Preferences should be encoded once (via plugins, AutoWiki, packaged droids) rather than re-explained every session.

**Electrification analogy:** factories that bolted a motor onto their steam-era layout saw almost no gains. The jump came from redesigning the factory around electricity. Bolt agents onto today's process and you still have the old factory.

### Uday Kiran Medisetty & Adam Huda — Uber (Agentic SDLC at Uber, Day 2)

Uber operationalised the software factory concept at scale: 99% of engineers use AI monthly, 70% of PRs are AI-attributed, 15% of PRs are done entirely by autonomous agents. Their framing: you cannot reach those numbers by individual teams adopting tools ad hoc. Six shared building blocks are required: [Model Gateway](model-gateway.md), [MCP Gateway](mcp-gateway.md), [DevPods](devpods.md), [Agent Skills](agent-skills.md), [Context Graph](context-graph.md), and an AI Assistant.

The Uber factory is assembled into four SDLC stages: Idea (Slack → specs + design variants), Build (specs → PR), Validation (PR self-improved before human review), Maintenance (continuously self-improving codebase).

### Addy Osmani — The Future of Engineering (Closing Keynote, Day 3)

Osmani's "Agentic Software Factory" is the most explicit statement of the accountability architecture that the prior two talks left implicit.

The pipeline: product intent / incidents / user feedback ("stuff worth doing") → **agent inner loop** (guide/context → generate → verify/solve) → **evidence** (tests, diff summary, risk notes) → **human verdict** (ship / block / redirect) → prod → users → monitor → back to inputs.

The key structural contribution is naming the **human verdict** as a first-class component — not a step in the agent loop, but the boundary between agent-generated work and production. "Lights off fails here" — marked precisely at the human verdict stage. Fully autonomous pipelines fail because provenance, intent, and ownership cannot be delegated to the generating agent.

The reframe: "The win is not removing people from the loop. The win is moving human judgment to the highest leverage checkpoint."

This maps to the loop engineering concept: the **engineer outer loop** (DECIDE → VERIFY → APPROVE → OWN) is the factory's governance layer. Agents run the inner loop (INVESTIGATE → IMPLEMENT → TEST → REPORT); engineers own the outer loop.

Supporting data from this talk:
- 42% of committed code is already AI-generated or significantly assisted (Sonar 2026)
- 92% of teams report some governance challenge with AI-generated code (GitLab 2026)
- 85% say review/validation is now the bottleneck
- 43% cannot reliably distinguish AI vs. human code

Osmani also names three new failure modes the factory creates at the individual level: [Cognitive Debt](cognitive-debt.md), [Cognitive Surrender](cognitive-debt.md), and [Orchestration Tax](cognitive-debt.md).

## Agreements

Both talks agree on the foundational shape: a software factory is a designed organizational infrastructure, not a tool adoption. The failure mode they both warn against is bolting AI onto existing process rather than redesigning around it. Both also emphasize that autonomous agent reliability requires explicit structure — whether that's Factory's validation contract + sequenced workers or Uber's six shared building blocks.

## Tensions / Open Questions

- **Build vs. buy:** Tížková says a factory is something you build and own, not a consultancy you hire. Uber built theirs from scratch. But Factory.ai is itself a vendor offering the infrastructure. There's a question of how much of the factory you outsource vs. own.
- **Validation bottleneck:** if validation takes 40% of mission time (Factory data) and Uber reports agents handling 15% of PRs fully autonomously, what does the validation architecture look like at Uber's scale? The talks are complementary but don't overlap directly.
- **Long-horizon reliability:** Tížková claims agents will eventually run for a year without interruption, but notes current reliability is ~50% at the task-doubling frontier. What does the 50%→99% path look like in practice?
- **What "engineering" means after the shift:** Tížková's "humans decide what to build, agents handle how" is the optimistic frame. Whether "what to build" remains a stable human role — or is the next layer to automate — is unanswered.

## See Also

- [Agent Readiness](agent-readiness.md) — measuring whether a codebase supports autonomous work
- [Model Routing](model-routing.md) — the Factory Router as one implementation of agnostic routing
- [KV Cache](kv-cache.md) — prefix caching at the agent API layer
- [DevPods](devpods.md) — Uber's ephemeral agent workspaces
- [Agent Skills](agent-skills.md) — Uber's portable, eval-gated skill units
- [MCP Gateway](mcp-gateway.md) — Uber's tool access layer

## Sources

- [Rise of the Software Factory](../talks/day2-1110-rise-of-the-software-factory.md) — Tereza Tížková (Factory)
- [Agentic SDLC at Uber: Building Blocks for Uber's Software Factory](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda (Uber)
- [The Future of Engineering — Closing Keynote](../talks/day3-1630-future-of-engineering.md) — Addy Osmani
