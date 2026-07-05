---
type: concept
tags: [agent-readiness, software-factory, agentic-sdlc, code-quality, enterprise]
updated: 2026-07-05 (2 sources)
---

# Agent Readiness

## Definition

Agent readiness is the measurable degree to which a codebase and engineering environment support autonomous agent work. It is not a sentiment or a capability question about the model — it is a structural question about the environment the agent runs in. Poorly structured environments degrade agent output in predictable, measurable ways.

> "Teams blame the model when the real bottleneck is missing pre-commit hooks, undocumented env vars, or build steps buried in Slack."
> — Tereza Tížková, "Rise of the Software Factory"

## How Different Speakers Framed It

### Tereza Tížková — Factory (Rise of the Software Factory, Day 2)

The core empirical finding: after AI coding adoption, repos with basic structure show 3–4× better metrics than repos without. This data is from GitClear and DX research.

**Repo levels:**
- **Level 1** — no linting config, no AGENTS.md, no pre-commit hooks
- **Level 2+** — even basic structure present

**Post-adoption metric gap:**

| Metric | Level 1 | Level 2+ | Ratio |
|--------|---------|----------|-------|
| Cognitive complexity increase | +96% | +29% | ~3.3× |
| Static-analysis warnings increase | +45% | +13% | ~3.5× |
| Duplicated line density increase | +122% | +34% | ~3.6× |

**Factory's agent-readiness scoring** evaluates repos across eight axes:
1. Style and validation (linting config, formatters, pre-commit hooks)
2. Build system (deterministic, documented build steps)
3. Testing (coverage, test harness accessibility)
4. Documentation (AGENTS.md, inline docs, README completeness)
5. Dev environment (reproducible setup, env var documentation)
6. Observability (structured logs, traces)
7. Security (scoped credentials, secrets management)
8. Task discovery (issues, project boards, structured work items)

These are not vanity metrics — they directly predict how well an agent will perform on the codebase.

**Encoding preferences:** once a codebase is ready, preferences should be encoded once (not re-explained every session). Factory's primitives for this:
- **Plugins** — bundle skills, slash commands, hooks, MCP server connections into one install
- **AutoWiki** — generates always-fresh documentation from the codebase automatically
- **Packaging layer** — ships droids, hooks, MCP servers as versioned packages
- **Context engine** — deferred context + scoped tools + structured user preferences

### Uday Kiran Medisetty & Adam Huda — Uber (Agentic SDLC at Uber, Day 2)

Uber operationalised agent readiness at scale through six shared infrastructure building blocks, each addressing a specific failure mode that blocks agent effectiveness:

| Building Block | Failure Mode It Solves |
|---------------|----------------------|
| [Model Gateway](model-gateway.md) | PII leakage, scattered cost attribution, per-request safety overhead |
| [MCP Gateway](mcp-gateway.md) | Tool-schema context bloat; N×M integration complexity |
| [DevPods](devpods.md) | Slow env setup; agents without isolated workspaces |
| [Agent Skills](agent-skills.md) | Duplicated agent expertise; no quality gate |
| [Context Graph](context-graph.md) | Blind agents doing expensive tool-call fan-out |
| AI Assistant | Zero-setup access for non-engineering users |

Uber's agent-readiness is expressed as concrete infrastructure rather than a scoring rubric, but both approaches share the same underlying theory: agent performance is a function of the environment, not just the model.

## Agreements

Both talks converge on the same core claim: **agent results are an environment problem, not a model problem.** The path to higher autonomous output runs through readiness investment — whether measured with a rubric (Factory) or built as shared infrastructure (Uber).

Both also share the "encode once, reuse everywhere" principle: Uber's [Agent Skills](agent-skills.md) are eval-gated units of expertise shared across agent harnesses; Factory's Plugins serve the same purpose at a per-developer level.

## Open Questions

- Is Factory's 8-axis scoring rubric correlated with Uber's 6-building-block checklist? They approach the same problem from different angles — can a single framework reconcile them?
- The GitClear/DX data covers "AI coding adoption" broadly — how much of the Level 1 degradation is from agents specifically vs. humans using Copilot-style tools carelessly?
- Is there a point of diminishing returns on readiness investment, or does the benefit keep compounding?
- How quickly can a Level 1 repo be brought to Level 2+ — is it a day's work (add a lint config, write an AGENTS.md) or months?

## See Also

- [Software Factory](software-factory.md) — the broader system agent readiness feeds into
- [Agent Skills](agent-skills.md) — Uber's reusable, eval-gated expertise units
- [DevPods](devpods.md) — Uber's ephemeral agent workspaces
- [Context Engineering](context-engineering.md) — what the agent sees; complements what the environment provides

## Sources

- [Rise of the Software Factory](../talks/day2-1110-rise-of-the-software-factory.md) — Tereza Tížková (Factory) — scoring rubric, empirical repo-level data
- [Agentic SDLC at Uber: Building Blocks for Uber's Software Factory](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda (Uber) — six-building-block infrastructure approach
