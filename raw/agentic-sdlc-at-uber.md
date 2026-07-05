# Agentic SDLC at Uber: Building Blocks for Uber's Software Factory

**Source:** Chang's photos of the speaker slides, two batches:
- Batch 1 (7 photos, inline): slides 2, 3–4, 5, 7, 8, 9
- Batch 2 (9 photos, uploaded as files, higher resolution): re-captured slides 2–5, 7–9 (superseding batch 1's transcription where legibility improved) plus 2 newly-captured slides — 14 (Validation) and 15 (Maintenance)
- Originals archived at `raw/slides/agentic-sdlc-at-uber/` (both the untouched `.HEIC` files and `slide-NN-*.png` conversions used for wiki embedding)

Still not captured: slide 1 (title), slide 6, slide 10 (AI Assistant), slide 11 (Idea), slides 12–13 (Build), and slide 16 (closing).

---

## Slide 2 — Agenda: "What we'll cover"

**What You Need** (BLOCKS):
- Model Gateway — Any model, governed
- MCP Gateway — Tools for agents
- DevPods — Agent workspaces
- Agent Skills — Reusable expertise
- Context Graph — Org knowledge
- AI Assistant — Zero-setup access

**What You Can Build** (SDLC):
- Idea — Slack jam to specs & design variants
- Build — Specs to PR
- Validation — PR self-improved before a human sees it
- Maintenance — Continuously self-improving codebase

---

## Slides 3–4 — Model Gateway

**What we needed to solve:**
- **PII hits third-party LLMs** — every app team rolls their own redaction, or skips it entirely.
- **Safety checks add seconds** — running a full moderation model on every request kills latency.
- **Cost attribution at scale** — spend is scattered across providers, teams, and projects without central metering.

**Architecture diagram:**

```
CALLERS
  Internal Use Cases (Agent Harnesses, Managed Agents, etc.)
  User-Facing AI Products (Customer Support, Rider App, Eats App, etc.)
         │
         ▼
┌─────────────────────────────────────────────┐
│ MODEL GATEWAY                                │
│ MIDDLEWARE                                   │
│  Identity (SPIFFE/SPIRE) → Data Anonymizer   │
│  → AI Guard → LLM Cache → Smart Router       │
│ MANAGEMENT                                   │
│  Project Catalog | Per-Caller/Per-User       │
│  Attribution | Spend Tiers                   │
│ OBSERVABILITY                                │
│  Audit Log | Session Traces | Cost           │
│  Reconciliation                              │
└─────────────────────────────────────────────┘
         │
         ▼
LLM PROVIDERS: OpenAI · Anthropic · Google · OSS Models
```

---

## Slide 5 — MCP Gateway

**Agent harnesses (callers):** Claude Code, Codex, OpenCode, Cursor, Langchain

**Caller patterns (cost ranked, cheapest last):**
- **Direct MCP** ($$$$) — N tool schemas loaded into context
- **Omni MCP** ($$$) — 1 MCP to discover & call any server, nothing in context
- **aifx mcp call** ($$) — CLI for any MCP, JSON out, zero context cost
- **Code Mode** ($) — skills that call MCP CLIs & parse outputs automatically

**Architecture diagram:**

```
┌─────────────────────────────────────────────┐
│ MCP GATEWAY                                  │
│ REQUEST FLOW                                 │
│  Auth → Discovery → Schema → Route → Execute │
│ MANAGEMENT                                   │
│  Auth & Identity | Observability | Rate      │
│  Limits                                      │
│ INFRASTRUCTURE                               │
│  Proxy MCP | Registry | Playground           │
└─────────────────────────────────────────────┘
         │
         ▼
MCP SERVERS
  1P: Code, Phabricator, QueryRunner, Flipt...
  3P: Glean, Slack, Google Workspace, GitHub...
```

---

## Slide 6 — DevPods

**What we needed to solve:**
- **Agents can't run on laptops** — autonomous agents need isolated, ephemeral environments.
- **Env setup kills agent speed** — cloning a monorepo, installing deps, and indexing code takes minutes.
- **Language silos fragment execution** — separate Go, Java, Web, iOS envs mean agents can't work across repos in one session.

**Example usage:**
```
ssh your-devpod-name
aifx agent run claude ...

DevpodCreate({
  name: 'minion-49c6d1ac',
  node_type: 'devpod-agentic',
  flavor: 'go-large',
  region: 'us-or',
  state: STATE_RUNNING,
})
```

**Stats:** 14K+ running DevPods at peak · ~secs startup time

**Architecture:**
```
USERS: Engineers | Interactive Agents | Autonomous Agents
         │
DevPod Platform
  MOBILE SIMULATION: iOS Simulator | Android Emulator | Visual Validation
  AI DEVPOD: AI-Optimized Image | One-Click Setup | Non-Engineer Access
  MEGA DEVPOD: Go, Java, Web, Data, iOS, Android → Unified Environment
  INSTANT START: Balloon Pods | Warm Pool | Snapshot Store | Pre-Indexed Code
  CORE RUNTIME: K8s Pod | DinD Sidecar | SSD Volume | Agent Runtime
         │
GKE REGIONS: Oregon (devpod-us-or) | Virginia (devpod-us-va) |
             Netherlands (devpod-nld) | São Paulo (devpod-bra) | Mumbai (devpod-ind)
```

---

## Slide 7 — Agent Skills

**What we needed to solve:**
- **Every team reinvents the same skill** — identical skills proliferate across repos, no shared quality bar, no reuse, no discovery.
- **Skill config isn't portable** — Claude Code, Codex, Cursor, OpenCode each need separate config for the same skill.
- **No feedback loop on quality** — hundreds of authors, no automated eval; production failures don't become test cases.

**Example:**
```yaml
# plugins/data-analyst/SKILL.md
---
name: data-analyst
description: Analyze data with SQL queries
allowed-tools: Bash Read Edit
---
```
```
# Install across all harnesses — see common config
$ aifx plugin add data-analyst
Added plugin data-analyst@dev-exp-agent-marketplace
Works in Claude Code, Codex, Cursor, OpenCode
```

**Stats:** 2.5K+ skills · 20K+ executions/day

**Who builds:**
- Core team: 450+ skills · 17 categories (data, oncall, testing, dev, ...)
- Domain teams: 2,000+ skills · 94 orgs (delivery, payments, android, ...)

**Quality gate pipeline:** Monorepo → 14 lint checks → 120-pt eval → Model gate → Skills Catalog (2.5K+)

**Distribution:** `aifx plugin add` | `aifx.yaml` defaults | persona defaults → Claude Code, Cursor, OpenCode

**Feedback loop:**
- Signals: OTel traces, review comments
- Actions: 120-pt eval, SKILL.md patches

---

## Slide 9 — Context Graph

**What we needed to solve:**
- **Agents guess without relationships** — agents can't traverse connections they can't see.
- **Context is scattered across systems** — 30+ sources including incidents, ownership, code changes, feature flags.
- **Fan-out burns tokens and time** — assembling connected context takes many tool calls & increases unpredictability.

**Demo comparison (same question, both get the correct answer):**

> "What % of Mobility trips in India use Cash?"

| | With Graph | Without Graph |
|---|---|---|
| Time | 44s | 627s (14x faster with graph) |
| Cost | $0.38 | $2.75 (7x cheaper with graph) |
| Tool calls | 8 | 94 |

> Graph finds the table instantly.
> Without it: 93 Bash calls exploring...

**Correction (from higher-resolution re-upload, 2026-07-05):** the cost figure is **$0.38**, not $0.36 as transcribed from the first, lower-resolution batch of photos. The second annotation line — illegible in batch 1 — is now confirmed as "Without it: 93 Bash calls exploring...". Note this is 93, one less than the 94 in the comparison table above; both figures are as they literally appear on the slide, the source material itself has this minor discrepancy (likely the annotation was written referring to just the exploration phase, or is an off-by-one from the table). Not smoothed over or reconciled here.

**Stats:** 40M+ nodes & edges · 150+ node/edge types

**Use cases:** Planning | Ownership | Oncall & RCA | Data Analysis | Security

**Graph node types (sampled from diagram):** Engineer, AI Tool, Team, Chat Channel, Screen, Mobile App, Design Doc, Repository, Metric, Design Review, Feature, Code Change, Service (center node), API Endpoint, Pipeline, Documentation, Database, Feature Flag, Alert, Work Item, Incident, On-Call, Dataset, Epic, Experiment, Event Stream

**Data sources:** Asset Registry | Code Review | Issue Tracker | Project Planning | Incident Mgmt | Feature Flags | CI/CD | +12 more

---

## Slide 14 — Validation: "Every PR self-improves before a human sees it"

Tab bar shown: Idea | Build | **Validation** | Maintenance

**Framing:** "Most validation used to happen after the PR. Reviewers caught what machines should have caught. We shifted that left."

**Self-improvement loops diagram:**
```
inner
  Code validation → AI code review · scoped
  Frontend visual validation ↔ Backend service integration
outer
  Self-Healing CI → Agentic code review · reasoning + skills
                                              → READY FOR HUMAN REVIEW ATTENTION
```

**Key learnings:**
- Shifting work into the inner loop reduced CI failures. By the time CI runs, the agent has already caught most issues.
- The agent does not flag issues. It fixes them. Each iteration leaves the PR in better shape.
- Trust comes from evidence. Every check passed, every screenshot captured, every review result attached to the PR before a human opens it.

**Powered by:** Model Gateway | MCP Gateway | Agent Skills | Context Graph | Simulators & Emulators

**Right-side mockup** (labeled "ILLUSTRATIVE — TOOLING UI RECONSTRUCTED FOR DEMO"):
```
○ Self-Healing CI · Classifying... · Conflict type: stale base [AUTO]
✓ Auto-rebased · CI re-running...
  All checks passing · 12s · No action required                    [AUTO]

OUTER LOOP · AGENTIC REVIEW                                        uReview
✦ uReview · multi-agent review (runs as a Skill)
✓ 2 inline comments posted
✓ 0 critical · 2 suggestions
✓ No fix agents spawned

✓ VALIDATION COMPLETE · PROOF ATTACHED
✓ Code Validation      22 fixed · 2 flagged
✓ Visual Validation    iOS ✓ · Android ✓ (matches Figma spec)
✓ Backend Staging      zone-router healthy on staging
✓ Self-Healing CI      merge conflict auto-rebased
✓ Agentic Review       2 comments · 0 critical

Screenshots + validation report attached to diff        Ready for human review →
```

Slide number: appears to be 14/16 (partially visible in photo; inferred from tab order — Validation precedes Maintenance which is confirmed 15/16 below).

---

## Slide 15 — Maintenance: "The codebase that maintains itself"

Tab bar shown: Idea | Build | Validation | **Maintenance** (confirmed page 15/16)

**Framing:** "Maintenance agents run on a schedule, making it easier to run five variants instead of two. Cleanup happens automatically."

**Managed self-improvement loops diagram:**
```
weekly    skill runs → PR → CI → review → land
biweekly  review comments → skill improves
monthly   incidents + bugs → new skills created
```

**Key learnings:**
- Maintenance loops should be managed. Control how many PRs run at once, when compute fires, and how much strain the team absorbs. Off-peak by default.
- Automatic maintenance enables more experiments. The cleanup happens. Engineering effort stays the same.
- Review comments improve the skill. Every accepted PR, every rejection, every comment becomes training data. The loop compounds.

**Powered by:** Model Gateway | MCP Gateway | Agent Skills | Context Graph

**Right-side mockup** ("ILLUSTRATIVE — TOOLING UI RECONSTRUCTED FOR DEMO"), a skill marketplace panel:
```
ENROLL A SKILL                                          skill marketplace
go-code · /venue/ + /config/ · ios-code · /Features/Pickup/

feature-flag-cleanup    [iOS · Go]                          [+ Enroll]
  Remove dead variant code after an experiment concludes.

dead-code-cleanup       [Go · Java]                         [+ Enroll]
  Delete unreferenced functions, types, and packages.

build-graph-pruning     [Android · Go]                      [+ Enroll]
  remove-unused-deps on mobile · inline-expensive-imports on Go.
  Keeps build graphs lean and compile times fast.

severe-bug-scanner-fix  [Go · Java]                         [+ Enroll]
  Detects and fixes high-severity bug patterns — nil dereferences,
  data races, unchecked errors — before they reach production.

perf-analysis           [Go · Java]                         [+ Enroll]
  Profile hot paths and surface targeted latency and allocation fixes.

Runs off-peak · opens PRs for review · enroll more than one
```
