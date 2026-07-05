# Always-on agents run production without the on-call tax

Speaker: Justin Smith
Role: Founding Product Engineer
Company: Resolve AI
Date: 2026-07-05
Day: Day 4 — Session Day 3
Time: 2:25pm–2:45pm
Room: Track 8
Track: Agentic Engineering
Type: session

Slides: slides/always-on-agents-production-on-call-tax/
- IMG_8909.HEIC — AI for prod (platform overview)
- IMG_8912.HEIC — Agents to run and fix your software (full architecture breakdown)
- IMG_8913.HEIC — Most production work is always there and not always a critical event
- IMG_8914.HEIC — Every operational task requires two things (Execution × Production Context)
- IMG_8915.HEIC — Every background agent operates on three principles
- IMG_8916.HEIC — When does the agent work? (schedule / event streams / message-based)
- IMG_8917.HEIC — How does the agent work? (always available / durable state / learning)
- IMG_8919.HEIC — Four workloads to easily hand off to background agents
- IMG_8920.HEIC — BYOA, or call Resolve AI from yours
- IMG_8921.HEIC — Three things to take away

Official description: Most production teams have the same problem. The work that keeps systems healthy — deployment checks, on-call handoffs, anomaly reviews — never makes it into a sprint. It falls to whoever has bandwidth, gets done inconsistently, and disappears when people are stretched thin. Background agents fix this by running that work on a schedule, using the same production context a senior engineer would, without waiting for someone to initiate it. Justin Smith, Founding Engineer at Resolve AI, walks through the architecture behind always-on agents, the use cases teams are starting with today, and what we have learned from running them in our production environment.

---

## Slide: AI for prod

Resolve.ai pitch: AI agents that handle on-call, incidents, and daily operational tasks.

Three claims:
- AI agents that handle on-call, incidents, and daily operational tasks
- Trusted to get to root cause in the most demanding production environments
- Fits into your agent ecosystem, or build your agents on Resolve AI

Platform stack (three layers):
- **AI Agents**: On-call agents, Incidents agents, Background agents, Your agents
- **Agent Architecture**: Models · Context · Reasoning · Actions · Learning ←→ Evals
- **Enterprise Grade Platform**: Integrations · Cost efficiency · Agent Security · Usage & audit · Deployment

---

## Slide: Agents to run and fix your software

Full breakdown of platform (zoomed-in version of previous slide):

**AI Agents:**
- On-call agent: Autonomous triage, Root cause backed by evidence
- Incidents agent: Shared investigation thread, Multiplayer investigation, Drives incident channel
- Background agents: Schedule on event or trigger, Multi-turn chat
- Your agents: MCP/API/Skill

**AI Agent Architecture:**
- Models: Post-trained models, Frontier model orchestration
- Context: Knowledge graph, Tool fluency, Context engineering
- Reasoning: Causal reasoning, Multi-agent coordination
- Actions: Guardrails, Scoped autonomy
- Learning: Explicit feedback, Implicit signals
- Evals: Absorbs new model and harness releases, Domain specialized eval framework, Calibrated to mirror engineer's investigation

**Enterprise Grade Platform:**
- Integrations: 60+ connectors, Custom tool integrations
- Cost efficiency: Optimized token usage, Credit based pricing
- Agent Security: Tenant isolation, Data protection
- Usage & audit: Usage reports, Trace agent reasoning
- Deployments: Cloud, Resolve AI Managed VPC w/BYOK*

---

## Slide: Most production work is always there and not always a critical event

Core framing: "On-call has a page. Incidents have a bridge."

**INVISIBLE TOIL**: This work has no critical trigger that makes anyone account for it, so it stays invisible and adds up.

Examples of invisible toil tasks:
- Watch the deploy that just went out
- Write the morning deploy and incident digest
- Re-investigate the p99 drift that came back
- Produce the weekly capacity report
- Run the recurring health check nobody owns

---

## Slide: Every operational task requires two things

Formula: **Task = Execution × Production Context**

**Execution**: The task in isolation, the actual analysis or change. Small and roughly fixed.

**Production Context**: What you know about the specific environment the work is being done in to know how to execute and evaluate the work being done. Knowledge and tools. Large and growing.

Key insight: **Production Context >> Execution.** Navigating your environment is a major contributor to toil.

---

## Slide: Every background agent operates on three principles

1. **When does it work**: Agents wake up on a predefined schedule, task, or a trigger.

2. **How does it work**: Run indefinitely. Persistent in state and delegate work based on the complexity.

3. **How does it know what to do**: Predefined set of tasks, skills, and integrations – backed by Resolve AI architecture.

---

## Slide: When does the agent work?

Subtitle: Runs continuously and involves you when a decision is required

Three trigger modes:

**On schedule** — Cron and interval.
Calendar-aligned work like a morning status report or an on-call handoff at shift change. Intervals for periodic maintenance.

**Event streams** — Keyword-matched ingestion.
Deploy events with "deploy," "release," or "rollback" auto-trigger the deploy monitor. No human needed.

**Message-based** — Direct human contact.
A Slack DM or channel message wakes the agent immediately. The message is the signal, no polling delay.

Bottom callout: "The agent tracks why it woke up each time, and acts accordingly."

---

## Slide: How does the agent work?

Three properties:

**Always Available — Runs indefinitely**
Runs in the Cloud. Enters standby when idle and wakes when there's work, so it is always available without always running hot.

**Durable State — Sandboxed filesystem**
Tasks, memory, and files persist across restarts automatically, so nothing in flight is lost.

**Learning — Knowledge and Memories**
Knowledge loops to understand your systems. Learns usage and preference patterns over time.

---

## Slide: Four workloads to easily hand off to background agents

1. **Deployment monitoring** [DEPLOY EVENT]
   Watch every rollout, run post-deploy health checks, catch regressions before they page.
   Template: Engineering Deploys Agent

2. **Scheduled health and anomaly checks** [SCHEDULE / ALERT]
   Periodic checks on the services and datastores you watch, including recurring drift.
   Template: PostgreSQL Watch

3. **Operational reports and handoffs** [SCHEDULE]
   Morning deploy and incident digest, on-call handoff at shift change, weekly status.
   Template: Daily Pulse

4. **First responder to engineering questions** [SLACK MESSAGE]
   Resolve AI agents becomes the first responder to any questions your engineering team gets.
   Template: Primary on-caller

---

## Slide: BYOA, or call Resolve AI from yours

**Drive from your agents**
Resolve AI runs as a Plugin, MCP server, API, and composable skill, so the AI systems your team already uses call its investigation capabilities directly.

**Bring your skills to Resolve**
Resolve AI agents consume your internal skills and tools, investigating with the full context of how your systems actually work.

---

## Slide: Three things to take away

01 — The biggest cost of operational work is navigating the environment complexity, not the task.

02 — Background agents run on a schedule or a trigger, and reason over your environment instead of executing fixed steps.

03 — You open Resolve AI to verify findings, and stand up new work in a sentence.
