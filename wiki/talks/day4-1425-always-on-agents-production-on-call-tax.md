---
type: talk
tags: [background-agents, production-context, invisible-toil, on-call, agentic-engineering, resolve-ai, scheduling, event-driven, toil-reduction, operations]
updated: 2026-07-05
---

# Always-on agents run production without the on-call tax

**Speaker:** [Justin Smith](../speakers/justin-smith.md) — Founding Product Engineer, Resolve AI  
**Day:** Day 4 — Session Day 3 · 2:25–2:45pm · Track 8  
**Track:** Agentic Engineering  
**Slides:** [raw/slides/always-on-agents-production-on-call-tax/](../../raw/slides/always-on-agents-production-on-call-tax/)  
**Raw notes:** [raw/always-on-agents-production-on-call-tax.md](../../raw/always-on-agents-production-on-call-tax.md)

---

## Official Description

> Most production teams have the same problem. The work that keeps systems healthy — deployment checks, on-call handoffs, anomaly reviews — never makes it into a sprint. It falls to whoever has bandwidth, gets done inconsistently, and disappears when people are stretched thin. Background agents fix this by running that work on a schedule, using the same production context a senior engineer would, without waiting for someone to initiate it. Justin Smith, Founding Engineer at Resolve AI, walks through the architecture behind always-on agents, the use cases teams are starting with today, and what we have learned from running them in our production environment.

---

## Key Claims

**1. Most operational toil is invisible — it has no ticket, no pager, no sprint slot.**

Smith opens with a category of work he calls "invisible toil": tasks that are always present but never owned. Examples: watch the deploy that just went out, write the morning deploy and incident digest, re-investigate the p99 drift that came back, produce the weekly capacity report, run the recurring health check nobody owns. These have no critical trigger to surface them. "On-call has a page. Incidents have a bridge." Invisible toil has neither.

**2. Every operational task = Execution × Production Context — and Context dominates.**

The central formula of the talk:

> **Task = Execution × Production Context**

- **Execution** = the task in isolation, the actual analysis or change. Small and roughly fixed.
- **Production Context** = what you know about the specific environment the work is being done in, to know how to execute and evaluate the work. Knowledge and tools. Large and growing.

**Production Context >> Execution.** Navigating your environment is the major contributor to toil — not the work itself. This is the core motivation for Resolve AI: build agents that carry and accumulate production context, so engineers don't have to re-navigate it every time.

**3. Background agents solve invisible toil because they carry persistent production context.**

Background agents run indefinitely on a schedule or trigger, maintain durable state, and accumulate knowledge of your environment over time. They don't just execute fixed steps — they reason over the environment using the same context a senior engineer would.

Three operating principles:
- **When**: wake on a predefined schedule, task, or trigger
- **How**: run indefinitely; persistent in state; delegate to sub-agents by complexity
- **What to do**: predefined set of tasks, skills, and integrations — backed by Resolve AI architecture

**4. Three trigger modes cover the full range of operational work.**

| Mode | Mechanism | Example |
|------|-----------|---------|
| On schedule | Cron / interval | Morning deploy digest; on-call handoff at shift change |
| Event streams | Keyword-matched ingestion | Deploy event with "deploy," "release," "rollback" auto-triggers a deploy monitor — no human needed |
| Message-based | Direct Slack DM or channel | A DM wakes the agent immediately; no polling delay |

The agent tracks *why* it woke up each time and acts accordingly.

**5. Three runtime properties make background agents operationally reliable.**

- **Always available**: runs in the cloud; enters standby when idle; wakes when there's work — never cold, never burning idle compute
- **Durable state**: sandboxed filesystem; tasks, memory, and files persist across restarts; nothing in-flight is lost
- **Learning**: builds a knowledge graph of your systems; learns usage and preference patterns over time

**6. Four starter workloads, with templates.**

Resolve AI ships pre-built templates to lower the adoption bar:

| Workload                                 | Trigger          | Template                  |
| ---------------------------------------- | ---------------- | ------------------------- |
| Deployment monitoring                    | Deploy event     | Engineering Deploys Agent |
| Scheduled health & anomaly checks        | Schedule / alert | PostgreSQL Watch          |
| Operational reports & handoffs           | Schedule         | Daily Pulse               |
| First responder to engineering questions | Slack message    | Primary on-caller         |

**7. The platform is composable — you can drive it or be driven by it.**

Two integration modes:
- **Drive from your agents**: Resolve AI exposes a Plugin, MCP server, API, and composable skill — so existing AI systems can call its investigation capabilities directly
- **Bring your skills to Resolve**: Resolve AI agents consume internal skills and tools, investigating with full context of how your systems actually work

---

## Emerging Concepts (1st source — no concept page yet)

These ideas are strongly articulated in this talk but have no second source yet. Concept pages will be created when a second source appears.

- **Production Context**: the environmental knowledge required to execute operational tasks correctly; scales faster than the tasks themselves; the dominant cost in operational work
- **Background Agents**: always-on, persistent agents that run on schedules or triggers without human initiation; durable state; reasoning over environment rather than executing fixed steps
- **Invisible Toil**: operational work with no critical trigger, no owner, no sprint slot; disappears when people are stretched thin; accumulates silently

---

## Connections to Existing Concepts

- **[Human Verdict](../concepts/human-verdict.md)**: Smith's closing takeaway — "you open Resolve AI to verify findings" — is a second real-world instantiation of the human-verdict pattern. The agent does the work and generates evidence; the engineer's role is verification and sign-off. This mirrors Osmani's outer loop (VERIFY → APPROVE → OWN) and uReview's approval gate, applied now to production operations rather than code review.
- **[Guardrails](../concepts/guardrails.md)**: The Resolve AI agent architecture includes "Scoped autonomy" and "Guardrails" as part of the Actions layer — consistent with the principle (from Manuja and Goyal) that agents must have bounded action surfaces with explicit policy checks.
- **[Context Engineering](../concepts/context-engineering.md)**: "Production Context" is a domain-specific flavor of context engineering applied to operational work. The Towards AI workshop found that what the model sees on every call is the primary cost lever at the application layer. This talk extends the same argument to the operational domain: the context *about your environment* is the primary cost lever in operational work.
- **[Software Factory](../concepts/software-factory.md)**: The "AI for prod" framing extends the software factory concept beyond the dev cycle to the operational lifecycle. The invisible toil category — deploys, health checks, handoffs — is the operational equivalent of the dev factory's "always-improving" property.

---

## Reactions / Open Questions

- The "Production Context >> Execution" formula is a clean articulation of something that's been implicit across the conference (Uber's Context Graph; Azzam's devbox design; the context engineering workshop's token cost data). It's worth naming as an explicit cross-conference pattern.
- Resolve AI's learning layer (explicit feedback + implicit signals) overlaps structurally with Uber's Agent Skills eval gating. Open question: does accumulating environment-specific context require a different architecture than Uber's SKILL.md + 120-pt eval approach?
- "You open Resolve AI to verify findings" positions the human as a *verifier* of agent work rather than an initiator. This is the most operationally concrete version of Human Verdict at the conference — a tight interface where the agent brings findings and the human renders a yes/no on them.
- The BYOA / MCP integration story is important for the conference's platform-layer theme. If background-agent infrastructure can be composed via MCP like any other tool, then it potentially fits inside Uber's existing MCP Gateway without re-architecting.
- Coverage note: 10 slides captured, full deck. No obvious gaps.
