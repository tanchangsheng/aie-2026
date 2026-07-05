---
type: concept
tags: [agentic-sdlc, agent-skills, skills, portability, quality-gate, eval, skill-marketplace, agent-fleet]
updated: 2026-07-05
---

# Agent Skills

## Definition

Agent Skills are reusable, portable units of expertise (e.g., `SKILL.md` files defining a name, description, and allowed tools) that can be installed into any agent harness (Claude Code, Codex, Cursor, OpenCode) rather than being reimplemented per-team or locked to one tool's config format. Uber runs 2.5K+ skills with 20K+ executions/day.

## What It Solves

- **Every team reinvents the same skill** — identical skills proliferate across repos with no shared quality bar, no reuse, and no discovery mechanism.
- **Skill config isn't portable** — Claude Code, Codex, Cursor, and OpenCode each historically needed separate configuration for functionally the same skill.
- **No feedback loop on quality** — with hundreds of authors and no automated eval, production failures never become test cases that prevent regressions.

## Skill Definition Format

```yaml
# plugins/data-analyst/SKILL.md
---
name: data-analyst
description: Analyze data with SQL queries
allowed-tools: Bash Read Edit
---
```

Installed once via `aifx plugin add data-analyst`, and works across Claude Code, Codex, Cursor, and OpenCode without per-harness reconfiguration — this is the direct fix for the portability problem.

## Who Builds Skills

| Builder | Scale |
|---|---|
| Core team | 450+ skills · 17 categories (data, oncall, testing, dev, ...) |
| Domain teams | 2,000+ skills · 94 orgs (delivery, payments, android, ...) |

The split shows skills aren't a centrally-curated catalog alone — domain teams contribute roughly 4–5× more skills than the core team, which is why a shared quality gate matters.

## Quality Gate Pipeline

```
Monorepo → 14 lint checks → 120-point eval → model gate → Skills Catalog (2.5K+)
```

This is the direct answer to "no feedback loop on quality": every skill, regardless of author, passes the same automated checks before it's discoverable in the catalog.

## Feedback Loop

- **Signals:** OTel traces, review comments (production usage data flows back in)
- **Actions:** reruns of the 120-point eval, automated `SKILL.md` patches

This closes the loop Uber explicitly named as missing: production failures now have a path back into the skill's own definition, rather than living only as one-off incident reports.

## Distribution

`aifx plugin add` (manual) | `aifx.yaml` defaults (per-repo) | persona defaults (per-role) — three distribution mechanisms targeting different adoption patterns (opt-in, repo-standard, role-standard).

## Relationship to Other Concepts

- "Code Mode," the cheapest caller pattern in the [MCP Gateway](mcp-gateway.md), is explicitly described as skills calling MCP CLIs and parsing output automatically — skills are part of how the MCP Gateway gets used cost-effectively.
- Skills run inside [DevPods](devpods.md) and rely on whatever the [Context Graph](context-graph.md) and Model Gateway expose to them.
- [AI Code Review](ai-code-review.md)'s "Custom Agents" are tagged "Skill + Knowledge Base," which may reuse this same skills infrastructure — unconfirmed, see that page's Open Questions.

## OpenRouter's Implementation — Skill Marketplace

OpenRouter takes a different architectural approach than Uber: rather than a monorepo quality gate, skills are distributed through a **marketplace UI** that scopes installs to individual interns. Fleet-wide stats from Shashank Goyal's talk (Jun 2026): **368 skills across 73 interns**, with skills like `agent-browser`, `clickhouse`, `fusion`, `hubspot`, `humanizer`, `internal-comms`, `notion` available for install.

Skills at OpenRouter are versioned as code in each intern's Git repo (`skills.lock` is a visible file in the `buddy` repo shown on slide). The "egg" base template propagates shared skills fleet-wide — an intern forks the egg and then evolves additional skills independently. This gives fleet-wide consistency on core skills while allowing per-intern specialization.

A notable contrast with Uber's approach:

| Dimension | Uber | OpenRouter |
|-----------|------|------------|
| Distribution | `aifx plugin add` / repo defaults / persona defaults | Marketplace UI, scoped to individual intern |
| Quality gate | 14 lint checks + 120-point eval + model gate | Not described in slides (may be in uncaptured slides) |
| Scale | 2,500+ skills, 20K+ executions/day | 368 skills across 73 interns |
| Scope | Enterprise (99% of engineers using AI) | Internal fleet (100% of employees) |
| Versioning | Monorepo CI pipeline | `skills.lock` in per-intern Git repo |

The key lesson from OpenRouter's production experience: **test skills on a single intern, then teach it to others as it improves** — a staged rollout pattern that differs from Uber's gate-before-catalog approach.

## Open Questions

- What does the 120-point eval actually check — correctness, safety, style, all three?
- How does the "model gate" step differ from the lint checks and eval — is it an LLM-as-judge pass?
- At 94 orgs contributing, how is namespace collision or duplicate-skill detection handled?
- Does uReview's "Custom Agents" (skill + knowledge base) run on this same Agent Skills system, or a separate one scoped to code review?

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda
- [Letting the Interns Loose](../talks/day3-1110-letting-the-interns-loose.md) — Shashank Goyal, OpenRouter (skill marketplace, 368 skills/73 interns, staged rollout lesson)
