---
type: speaker
tags: [pydantic, python, rust, open-source, sandbox]
updated: 2026-07-05
---

# Samuel Colvin

**Role:** Founder & CEO  
**Company:** Pydantic (backed by Sequoia)  
**Twitter:** https://x.com/samuelcolvin  

## Background

Python and Rust developer with 13+ years of software engineering experience. Created Pydantic Validation, an open-source library downloaded over 550M times per month and a core dependency of virtually every GenAI Python library. Has since built a suite of developer tools under the Pydantic umbrella: Pydantic Logfire (developer-first observability), Pydantic AI (agent framework), Pydantic Evals (AI evaluation), Pydantic AI Gateway (model routing), and Pydantic Monty (a Python implementation in Rust, for LLMs to run code without host access).

## Talks at AIE World Fair 2026

- [Your Agent Needs a Sandbox, Not a Desert](../talks/day3-1205-agent-needs-sandbox-not-desert.md) — Day 3, 12:05–12:25pm, Track 1

## Recurring Themes

- Minimal, safe-by-default compute for agents over full Linux VM sandboxes.
- Python + Rust as a combination: using Rust for safety/performance, Python as the surface API for agents.
- Constraints as features: the limitations of a minimal runtime (no filesystem, no network by default) are what make it trustworthy for AI-generated code.
- Scepticism of the "agents always need full sandboxes" conventional wisdom — the argument that sandbox vendors benefit from this assumption.
