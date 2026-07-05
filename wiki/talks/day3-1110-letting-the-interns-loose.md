---
type: talk
tags: [agent-fleet, intern-model, config-as-code, slack-native, skill-marketplace, secrets, vm-isolation, openrouter, ai-adoption, self-evolving-agents]
updated: 2026-07-05
---

# Letting the Interns Loose — How We Accelerated AI Adoption

| Field | Value |
|-------|-------|
| Speaker | [Shashank Goyal](../speakers/shashank-goyal.md) |
| Affiliation | OpenRouter |
| Date | 2026-07-05 |
| Time | 11:10–11:30am |
| Track | Sandbox & Platform Engineering |
| Room | Track 1 |
| Day | Day 3 — Session Day 2 |
| Slides coverage | ~14 of ~33 slides (slide ~8–9, 12–15, 18, 20–21, 24, 27, 32–33 captured; title slide and several mid-deck slides missing) |

---

## Summary

Shashank Goyal walks through how OpenRouter built and ran a fleet of 73 AI "interns" — long-running, Slack-native, Git-backed agents — to achieve 100% employee AI adoption (21 sessions/employee/week) and 31,512 intern sessions in a single month, 84% of which ran fully on autopilot. The talk is a technical walkthrough of the architecture that made it possible and six lessons distilled from running it in production.

---

## Key Claims

- **The "intern" framing beats "generic agent" framing.** A narrowly-scoped agent that does one job well is more reliable than a brilliant generalist that wanders. Constraints are the feature, not a limitation.
- **Three properties make an intern delegatable:** it's long-running (not a one-off session), it lives in Slack (zero new tooling for employees), and it improves itself automatically when the team learns something new.
- **Config as code is the enabling abstraction.** Prompts, tools, schedule, and permissions live as versioned files in a Git repo. Behavior changes via PR; rollbacks happen via revert. No state buried in a database.
- **One VM per intern is the right isolation unit.** It gives durable state (working directory, caches, logs that outlive any single task), hard blast-radius isolation (a misbehaving intern can't reach another's secrets), and makes every intern's cost/logs legible without an orchestration layer.
- **Secrets are enforced at the OS level, not the LLM level.** An intern can never read its own credentials. The path to production requires a human-wired button click routed through a separate privileged sibling process — no chat message to the harness can trigger it.
- **A skill marketplace distributes capabilities across the fleet.** Skills are installable per-intern; the marketplace UI allows scoped installs. Fleet total: 368 skills across 73 interns.
- **The "egg" pattern allows fleet-wide propagation.** Every intern forks from a shared base template. Improvements to the egg propagate; each intern then evolves its own skills and memory independently.
- **Model preference is personal to each intern** — start on a smart/capable model, then eval down to cheaper ones once the task definition is stable.
- **Self-evolving agents are necessary** for agents to become integral rather than optional.

---

## Architecture

### The Reframe — Intern vs Generic Agent [slide ~8]

> **The trick:** constraints make agents reliable. A narrow intern beats a brilliant generalist that wanders off.

Two properties distinguish an intern from a generic agent:
- **A specialist** — master of one, not jack of all trades
- **Well defined scope** — does the thing you asked, the way you asked; doesn't freelance

### The Fix — Three Abstractions [slide 9]

| # | Property | Detail |
|---|----------|--------|
| 01 | Long-running | Not a one-off session — stays alive and on the job |
| 02 | Lives in Slack | Talk to it where you already work; nothing to run locally |
| 03 | Improves itself | When the team learns something new, it upgrades — automatically |

### Config — Every Intern Is a Git Repo [slide 12]

Identity, prompts, tools, and schedule all live as files in the intern's own repository. The harness reads from Git; behavior changes with a commit.

- **Config as code** — name, schedule, tools, and permissions are checked-in files, not hidden state buried in a database
- **Skills and workflows are code** — versioning them is the real reason every intern is a repo
- **One source of truth** — update an intern by merging a PR; every change has a diff; roll back the same way

The slide showed a live GitHub repo named `buddy` (Private): 18 branches, 16 tags, ~300 commits, one contributor (`louisgy L`). Repo files include `.claude`, `.github/workflows`, `CLAUDE.md`, `skills.lock`, `slack-manifest.yml`, `biome.json`. Latest release: v1.6.3.

### Runtime — One VM per Intern [slide 13]

| Property | Detail |
|----------|--------|
| Durable local state | Working directory, clone, caches, and logs outlive any single task |
| Hard isolation | A blast radius of one; a misbehaving intern can't touch another's machine or secrets |
| Portable | Use it from anywhere; no laptop required |
| Simple to reason about | One intern = one box = one bill = one set of logs; no orchestration layer to debug |

### Security — Interns Can't Read Their Own Secrets [slide 14]

An intern ships to prod and never sees a credential. The host app owns the secrets; the harness stays blind.

Deployment flow:
```
Intern → Host app (control plane) → Human approval (wired button click) → Privileged sibling (separate process) → Deploy
```

**Why it can't bypass:** the secrets file is masked at the OS level (`systemd InaccessiblePaths`), so the harness process can't open it. The privileged sibling only fires from a wired button click — a chat message to the harness can never trigger it.

### Skill Marketplace [slide 15]

A UI-driven marketplace lets skills be installed scoped to a specific intern. Skills visible in the screenshot: `agent-browser`, `clickhouse`, `fusion`, `google-workspace`, `hubspot`, `humanizer`, `install-skill`, `internal-comms`, `notion`.

### The Egg — Base Template [slide 18]

Every intern starts from a shared base template ("the egg") that anyone forks into their own agent.

| Step | Name | Detail |
|------|------|--------|
| 01 | Shared DNA | One base template every intern is born from |
| 02 | Fork your own | Grab an egg, point it at your job; new intern instantiated |
| 03 | Evolve independently | Grows its own skills and memory for what's needed |

---

## Results [slide 20]

OpenRouter fleet metrics — last 4 weeks as of talk date (Jun 3–24, 2026):

| Metric | Value |
|--------|-------|
| Employee adoption | 100% ran a session |
| Avg sessions/employee/week | 21 |
| Total tokens (fleet) | 265B |
| Unique models run | 35 |
| Interns created | 73 |
| Skills across fleet | 368 |
| Input tokens from cache | 89% |
| Tool-call requests | 74% |

**Sessions per week (human + scheduled, fleet-wide):**

| Week of | Sessions |
|---------|----------|
| Jun 3 | 3,904 |
| Jun 10 | 8,691 |
| Jun 17 | 7,484 |
| Jun 24 | 8,925 |

**31,512 intern sessions in 30 days — 84% ran on autopilot — no human in the loop**
*(Jun 3 is a lower bound; newer bots were still ramping)*

---

## Meet the Fleet — Example Interns

### buddy
- **Role:** Helps launch models on OpenRouter — updates configs, runs validation tests, manages internal state
- **Top skills:** `stage-endpoint`, `stage-private-model`, `find-endpoint`, `model-description`

### Shakespeare
- **Role:** Edits blog posts — keeps a consistent voice, fact-checks claims, raises the bar on writing
- **Top skills:** `draft-content`, `announcement-blog`, `content-idea`, `writing-loop`

### eavesdropper
- **Role:** Sits in meetings, transcribes, writes clean notes (decisions, action items, owners) into shared memory
- **Top skills:** `orgchart`, `changelog`, `contents-commit`, `stale-nudge`

---

## Production Lessons

### Lessons 1/2 — "Each one is scar tissue from running this in production." [slide 32]

- **Model preference is personal to each intern** — start smart and then eval to cheaper models
- **Lower friction to drive adoption** — meet people where they work
- **Guardrails are important especially for secrets** — the real unlock started when we could safely add more capabilities to the agents

### Lessons 2/2 — "Three more, for when the fleet starts to grow." [slide 33]

- **Self-evolving agents are necessary** for agents to become integral
- **Build common abstractions** so that managing agents is simple and safe
- **Test skills on a single intern** and then teach it to others as it becomes better

---

## My Reactions

- The "intern" framing is the most memorable mental model for AI agents I've encountered at this conference. "Constraints make agents reliable" is a principle that cuts against the typical "more capable = better" instinct.
- The Git-as-config architecture is elegant and solves real problems (auditability, rollback, collaboration) that most teams are ignoring by using database-stored prompt state.
- 89% cache hit rate on input tokens is remarkable — directly validates the context-engineering workshop's "keep-everything + caching wins" thesis.
- The privileged sibling / systemd InaccessiblePaths secrets pattern is the most concrete secrets-isolation design I've seen described at this conference; Kanish Manuja and Uber both mentioned guardrails but not this depth of OS-level enforcement.
- The "egg" pattern for propagating improvements across a fleet is essentially a platform-layer answer to "how do you keep 73 interns from diverging into entropy." Elegant.
- 84% autopilot rate (31k sessions, no human in loop) is the strongest deployment claim at the conference so far.

---

## Links

- [Shashank Goyal](../speakers/shashank-goyal.md)
- Related concepts: [Agent Skills](../concepts/agent-skills.md) · [Guardrails](../concepts/guardrails.md) · [Model Routing](../concepts/model-routing.md) · [Software Factory](../concepts/software-factory.md)
- Related talks: [Agentic SDLC at Uber](day2-1140-agentic-sdlc-at-uber.md) (skills, fleet infrastructure) · [Productionizing LLM Gateways](day2-1425-productionizing-llm-gateways.md) (guardrails, secrets)
- Source: [raw/letting-the-interns-loose.md](../../raw/letting-the-interns-loose.md)
