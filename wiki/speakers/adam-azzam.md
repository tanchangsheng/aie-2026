---
type: speaker
tags: [modal, prefect, fastmcp, infrastructure, devboxes, mcp, orchestration]
updated: 2026-07-05
---

# Adam Azzam

| Field | Value |
|-------|-------|
| Role | Member of Product Staff |
| Affiliation | Modal |
| Previous | VP of Product at Prefect; maintainer of Prefect and FastMCP |
| Background | PhD in mathematics |
| Twitter | [@aaazzam](https://x.com/aaazzam) |
| LinkedIn | [adam-azzam](https://linkedin.com/in/adam-azzam) |
| Website | [adamazzam.com](https://adamazzam.com) |

---

## Background

Adam Azzam holds a PhD in mathematics. Before joining Modal, he was VP of Product at Prefect (a workflow orchestration platform) and a core maintainer of both Prefect and FastMCP. His background spans orchestration, developer tooling, and — at Modal — the infrastructure layer that agent workloads run on.

---

## Talks at AIE 2026

- [Don't Build Agents, Build Environments](../talks/day3-1045-dont-build-agents-build-environments.md) — Day 3, 10:45–11:05am, Track 1 (Sandbox & Platform Engineering)

---

## Recurring Themes

- **Environments over agents** — the agent loop is solved; the hard part is the reproducible, fast-booting, correctly-scoped environment the agent runs inside
- **Trust boundaries** — agent devboxes have a fundamentally different threat model than CI: the agent owns the control flow, not the operator
- **Orchestration** — Control Plane / Data Plane separation as the architectural answer to agent environment management
- **Practical infrastructure patterns** — favors specific, production-tested approaches (e.g., Ramp's Modal-based warm-image pipeline) over abstract frameworks
