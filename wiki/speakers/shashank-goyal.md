---
type: speaker
tags: [openrouter, infrastructure, agent-fleet, platform-engineering]
updated: 2026-07-05
---

# Shashank Goyal

| Field | Value |
|-------|-------|
| Role | Head of Provider Ecosystem & Founding Engineer |
| Company | OpenRouter |
| LinkedIn | https://www.linkedin.com/in/shashankgoyal1/ |
| Twitter | https://x.com/shashankgoyal95 |

## Background

Founding Engineer and Head of Provider Ecosystem at OpenRouter, where he helps build the infrastructure powering one of the world's largest LLM marketplaces — enabling developers and enterprises to access, route, and scale across hundreds of AI models and providers through a single API.

Prior to OpenRouter: backend engineer at OpenSea; 4+ years as a software engineer at Google. Experience spans large-scale infrastructure, developer platforms, and AI systems, with a focus on reliability, performance, and developer experience.

## Talks at AIE World Fair 2026

- [Letting the Interns Loose — How We Accelerated AI Adoption](../talks/day3-1110-letting-the-interns-loose.md) — Day 3, Track 1, 11:10–11:30am

## Recurring Themes

- **Constraint as reliability lever** — the "intern" framing: a narrow specialist reliably outperforms a capable generalist that can wander
- **Config as code for agents** — using Git repos as the canonical representation of an agent's identity, prompts, tools, and schedule
- **OS-level security boundaries** — `systemd InaccessiblePaths` for secrets isolation; human-wired approval gates rather than LLM-level trust
- **Fleet-scale thinking from day one** — 73 agents, 368 skills, 31k sessions/month; the design choices (one VM per intern, egg template, skill marketplace) were made for a fleet, not a single bot
- **Self-evolving agents** — agents that update their own skills automatically are the prerequisite for agents becoming integral rather than optional
