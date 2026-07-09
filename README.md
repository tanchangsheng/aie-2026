# AIE World Fair 2026 — Notes & Wiki

Personal notes from [AIE World Fair 2026](https://www.ai.engineer/worldsfair/2026), July 2026. 17 talks ingested across four days covering LLM inference, agentic systems, software factories, and production AI operations.

> Best viewed in [Obsidian](https://obsidian.md) — internal links, graph view, and backlinks work as intended.

## Structure

```
raw/        ← original notes from talks (read-only)
wiki/       ← structured wiki built from the raw notes
  index.md       ← master catalog of all pages
  overview.md    ← high-level synthesis across the conference
  talks/         ← one page per session
  speakers/      ← one page per speaker
  concepts/      ← cross-talk concept pages
event-schedules/ ← session catalog and my schedule
workshops/       ← workshop materials and exercises
slides/          ← saved slide decks
```

## Major Themes

**[The engineer's role shifts to the outer loop](wiki/concepts/human-verdict.md)** — the inner loop (investigate, implement, test, report) is the agent's domain; the outer loop (decide, verify, approve, own) is the engineer's. Accountability and taste — judgment before the metric exists — are the decay-resistant differentiators. Three failure modes don't show up in productivity metrics: cognitive debt, cognitive surrender, and orchestration tax.

**[Agentic engineering needs platform infrastructure](wiki/concepts/software-factory.md)** — at org scale, agentic dev requires shared building blocks (model gateway, MCP gateway, devpods, skills catalog, context graph), not ad hoc tool adoption per team. Uber's six building blocks drove 70% AI-attributed PRs and 15% fully autonomous. The Software Factory frame (spec→build→validate→deploy→learn) unifies what the conference is building toward.

**[The environment is the hard problem, not the agent loop](wiki/concepts/agent-environment-architecture.md)** — three independent production teams (Uber, OpenRouter, Ramp/Modal) converged on the same answer: pre-bake environments, mount warm, isolate secrets from the agent process. The threat model for agent devboxes is fundamentally different from CI: the agent generates and executes its own code. 1,000 sandbox tasks revealed 18 distinct failure modes, most of them silent.

**[Context overhead dominates execution cost at every layer](wiki/concepts/context-engineering.md)** — the same pattern appears at four abstraction levels: KV cache hit rate at the API level, tool-call fan-out (8 vs. 94 calls for the same answer), agent workspace boot time, and production operational context. Routing decisions at every layer (model routing, engine routing, tool access) should minimise recomputation of already-computed context.

**[LLM Inference is an infrastructure orchestration problem](wiki/concepts/inference-control-plane.md)** — of the 12 steps when a prompt arrives, only 2 are ML; the rest is distributed systems. Cost/1M tokens fell 1,200× since 2023, but volume grew faster, making inference a board-level financial concern. Everything reduces to GPU memory management (`weights + KV cache + overhead`); throughput improvements come from stacking orthogonal levers, not a single fix.

## Navigation

- [Wiki Index](wiki/index.md) — full catalog of talks, speakers, and concepts
- [Overview](wiki/overview.md) — running synthesis of conference themes
- [Reading List for Colleagues](wiki/reading-list-colleagues.md) — curated entry points by theme
- [Change Log](wiki/log.md) — record of all wiki updates
