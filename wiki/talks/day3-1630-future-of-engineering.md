---
type: talk
tags: [future-of-engineering, harness-engineering, loop-engineering, human-verdict, taste, cognitive-debt, agentic-software-factory, agency-ladder, accountability, keynote]
updated: 2026-07-05
---

# The Future of Engineering (Closing Keynote)

## Metadata

| Field | Value |
|---|---|
| Speaker | Addy Osmani |
| Official title | Closing Keynote |
| Slide deck title | The Future of Engineering |
| Date | Day 3 — Session Day 2 |
| Time | 4:30–4:50pm |
| Room | Main Stage |
| Track | Autoresearch |
| Type | Keynote |
| Source | [raw notes](../../raw/addy-osmani-future-of-engineering.md) — 32 slides (1 missing: gap between slides 30–31) |

---

## Central Thesis

The engineer of the future will choose what is worth doing, then own the evidence, understanding and verdict for work increasingly automated by agents.

Three definitions anchor the whole talk:

- **Quality** = the system of checks that produces evidence
- **Verdict** = the human/accountable decision made from that evidence
- **Answerability** = the ability to explain and stand behind the verdict later

---

## Key Claims

### 1. Roles Are Unbundling from Craft and Rebundling Around Ownership

Drawing on a Boris Cherny (@bcherny) framework observed at the Claude Code team, Osmani argues that engineering roles are separating from job titles and re-forming around five archetypes:

| # | Mode | What they do |
|---|---|---|
| 1 | **Prototyper** | Comes up with brand new ideas; churns out many, most don't ship |
| 2 | **Builder** | Quickly turns a prototype/idea into production-grade product/infra |
| 3 | **Sweeper** | Cleans up UI, simplifies code and system, unships, optimizes performance |
| 4 | **Grower** | Takes a built product and iterates it toward Product-Market Fit |
| 5 | **Maintainer** | Owns a mature system; keeps it secure, reliable, fast, and efficient at scale |

People often span 2–3 archetypes. The archetypes cross job function — some designers are Builders, some engineers are Sweepers. The title matters less than the part of the system you can own.

### 2. Harness Engineering: The Agent Is the System Around the Model

`agent = model + harness` — the harness is prompts, tools, state, constraints, and feedback loops. Engineering the harness is now a core discipline.

Diagram components of the harness:
- **CONTEXT** (rules, memory) — feeds into model
- **CONTROL** (plans, routing) — feeds into model
- **OBSERVE** (tests, logs) — bidirectional with model
- **ACTION** (tools, MCPs) — model output
- **PERSIST** (files, git) — model output
- **HOOKS** (block, retry) — bidirectional with model
- **RATCHET** (new rule) — output of action path, feeds back as context

Source referenced: addyosmani.com/blog/agent-harness

### 3. Loop Engineering: Design the Loop, Not the Prompt

The discipline shift is from writing prompts to designing the systems that prompt agents.

`loop = goal + cadence + isolated work + verification + state`

The loop cycle (outer ring): VERDICT (owns outer loop) → AUTOMATE (cadence finds work) → STATE (memory lives outside) → ACT (agents in worktrees) → LEARNING (tomorrow reads today) → VERIFY (maker ≠ checker) → ISOLATION (parallel, no chaos) → DECIDE (ship, block, queue) → back to VERDICT. At center: RECURSIVE GOAL (iterate until done).

Loops change the work. They do not delete the engineer.

Source referenced: addyosmani.com/blog/loop-engineering

### 4. The Agentic Software Factory — With the Lights On

The full pipeline Osmani presents:

- **Inputs**: product intent / incidents / user feedback ("stuff worth doing")
- → **Agent inner loop**: guide/context → generate → verify/solve (sandbox, traces, tests)
- → **Evidence**: tests, diff summary, risk notes
- → **Human verdict**: ship / block / redirect
- → **Prod** → **Users** → **Monitor** → back to inputs

"Lights off fails here" — marked at the human verdict stage. Fully automated pipelines break down at the accountability checkpoint.

The win is not removing people from the loop. The win is moving human judgment to the highest leverage checkpoint.

### 5. AI Code Share Is No Longer Marginal — Data

| Year | AI-generated or significantly assisted code |
|---|---|
| 2023 | 6% |
| 2024 | 19% |
| Now (2026) | 42% |
| 2026 est. | 55% |
| 2027 est. | 65% |

Source: Sonar State of Code Developer Survey 2026

The factory is no longer experimental. It is entering the commit history.

Clean code matters more than ever: **7–8% fewer tokens, 34% fewer file revisits, same pass rate** when agents work on clean vs. messy codebases.

### 6. The Trust/Verification Gap

From the same Sonar survey:
- 96% do not fully trust AI-generated code
- 48% always verify AI code before committing
- 38% say reviewing AI code takes longer than human code

**2× trust gap**: skepticism is high, but verification is not keeping up. The risk is not that engineers distrust AI. It is that they distrust it and still ship faster than they verify.

### 7. The Governance Gap — Generation Moved Faster Than Control

From GitLab AI accountability research, June 2026:
- 85% — review/validation is now the bottleneck
- 84% — governance after creation is the challenge
- 82% — AI code risks new technical debt
- 80% — adopted AI faster than policy
- 43% — cannot reliably distinguish AI vs human code
- **92%** — report some governance challenge with AI-generated code

This is the argument for the HumanVerdict: provenance, intent, and ownership need a first-class interface.

### 8. Alpha and Decay — The Framework for Career Durability

- **Alpha**: what makes you meaningfully better than what current models can do
- **Decay**: how fast the models catch up

Capabilities decay at different rates across model releases:
- **Speed**: gone — 600 lines while you find coffee
- **Recall**: gone — it's a context window now
- **Verification**: automating — "we were the weak link"
- **Taste**: alpha — resets every release, will take longer to decay
- **Judgment**: a slope rather than a wall

The half life of an edge is a release. The half life of a signature is a career.

Durability chart: accountability stays flat (doesn't decay); taste decays slowly; verification and speed decay faster.

### 9. Taste as the Primary Human Differentiator

Paul Graham (Feb 14, 2026): "In the AI age, taste will become even more important. When anyone can make anything, the big differentiator is what you choose to make."

Mitchell Hashimoto's definition of taste (Jun 26): "Taste is the ability to consistently make high-quality qualitative judgments where no objective metric exists. It's the creation of something that feels right intuitively, with no real justifiable way to measure that. But when you do it, people feel it... The funny thing about taste is that it's hard to create, but its result is very easy to copy. Once someone makes a tasteful decision, others can imitate it almost immediately."

Osmani's synthesis: Taste is the judgment before the metric exists. Hard to create. Easy to copy. Valuable because it decides what everyone copies next.

### 10. Three Anti-Patterns Engineers Must Avoid

**Cognitive debt**: the erosion of your understanding and memory around how to solve problems. 17% lower comprehension when learning through AI-generated code (50% vs 67%). Source: Anthropic internal study, 52 engineers.

**Cognitive surrender**: when you blindly accept what AI gives you and stop critical thinking. 73% accepted the wrong answer when AI was wrong, and felt more sure. Source: Wharton, 1,372 people.

**Orchestration tax**: the diminishing returns and cognitive drain experienced when managing parallel AI agents at once.

### 11. The Agency Ladder

Stop asking what AI can't do. Ask what only a human can be answerable for.

Everyone is a developer now. That doesn't make them an engineer. The question is: what should engineers avoid to stay effective and accountable?

The Agency Ladder — how much of the problem do you own:

| Level | Mode | Phrasing |
|---|---|---|
| 7 *(apex)* | **DISCERN** | "Found it — not worth fixing. Moving on." *(knowing what NOT to do)* |
| 6 | **RESOLVE** | "Found it. Fixed it. Looping you in." |
| 5 | **RECOMMEND** | "...and here's the one I'd pick." *(live here from day one)* |
| 4 | **PROPOSE** | "...and a few ways to fix it." |
| 3 | **DIAGNOSE** | "Here's the problem, and the cause." |
| 2 | **EXECUTE** | "Hand me the fix — I'll ship it." |
| 1 *(base)* | **FLAG** | "There's a problem." *(then walks away)* |

High agency = the mindset of actively taking ownership of your outcomes. The agent can choose. Only people inherit consequences.

### 12. Inner Loop / Outer Loop Split — The Loop Boundary Is Evidence

Agents run the inner loop. Engineers own the outer loop.

**Agent inner loop** (capability: do work, return evidence):
- INVESTIGATE → IMPLEMENT → TEST → REPORT

**Boundary**: EVIDENCE (diff · tests · logs · why)

**Engineer outer loop** (agency: decide what earns production trust):
- DECIDE (is this worth doing?) → VERIFY (is the evidence enough?) → APPROVE (ship, block, or redirect) → OWN (carry the consequence)

Constraints and learning from the outer loop feed the next agent run.

The inner loop is capability. The outer loop is accountability.

Operationalized: Explain it or don't ship it. You cannot answer for what you cannot understand.

### 13. New Work Is Real Work — Engineering Moves Up a Level

Section header: "Automation Moves the Floor"

What gets automated (the new floor): typing · boilerplate · first drafts · routine fixes · repeated checks → fewer keystrokes, more surface area. NOT LESS ENGINEERING.

New bottlenecks: coordination · verification · maintenance · product judgment · incident learning

Engineering moves up a level to own: **Loop Design** (what runs, when, and why) + **Evidence Design** (what proves it worked) + **Brownfield Care** (keep real systems healthy)

The job is not to protect the old loop. It is to own the new one.

### 14. The Historical Pattern

Every time we made it easier to write software, we ended up writing exponentially more of it.

The future belongs to engineers who make agent work legible, verifiable, and worth shipping.

---

## Reactions

*[Slide-only notes — no personal commentary captured. To be annotated after review.]*

---

## Links

- [Addy Osmani](../speakers/addy-osmani.md)
- [Harness Engineering](../concepts/harness-engineering.md)
- [Loop Engineering](../concepts/loop-engineering.md)
- [Human Verdict](../concepts/human-verdict.md)
- [Taste as Alpha](../concepts/taste-as-alpha.md)
- [Cognitive Debt](../concepts/cognitive-debt.md)
- [Software Factory](../concepts/software-factory.md)
