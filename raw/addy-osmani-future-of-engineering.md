# The Future of Engineering — Addy Osmani (keynote)
AIE World Fair 2026
Speaker: Addy Osmani (@addyosmani)
[NOTE: These are raw notes from slides 1–32 of the talk. One slide missing: IMG_8859 was not uploaded (gap between slides 30 and 31).]

## Slide Photos

All original photos are saved in `slides/addy-osmani-future-of-engineering/`.

| Slide | File | Content |
|---|---|---|
| 1 | [IMG_8829.HEIC](slides/addy-osmani-future-of-engineering/IMG_8829.HEIC) | Central thesis — engineer of the future |
| 2 | [IMG_8830.HEIC](slides/addy-osmani-future-of-engineering/IMG_8830.HEIC) | Definitions: Quality / Verdict / Answerability |
| 3 | [IMG_8831.HEIC](slides/addy-osmani-future-of-engineering/IMG_8831.HEIC) | Boris Cherny tweet — 5 engineering archetypes |
| 4 | [IMG_8832.HEIC](slides/addy-osmani-future-of-engineering/IMG_8832.HEIC) | Future of Careers — roles rebundling around ownership |
| 5 | [IMG_8833.HEIC](slides/addy-osmani-future-of-engineering/IMG_8833.HEIC) | Harness Engineering diagram |
| 6 | [IMG_8834.HEIC](slides/addy-osmani-future-of-engineering/IMG_8834.HEIC) | Loop Engineering diagram |
| 7 | [IMG_8835.HEIC](slides/addy-osmani-future-of-engineering/IMG_8835.HEIC) | Agentic Software Factory diagram |
| 8 | [IMG_8836.HEIC](slides/addy-osmani-future-of-engineering/IMG_8836.HEIC) | AI code share chart — no longer marginal |
| 9 | [IMG_8837.HEIC](slides/addy-osmani-future-of-engineering/IMG_8837.HEIC) | Clean code is cheaper for agents to read |
| 10 | [IMG_8838.HEIC](slides/addy-osmani-future-of-engineering/IMG_8838.HEIC) | Reviewers are already overloaded — trust gap chart |
| 11 | [IMG_8839.HEIC](slides/addy-osmani-future-of-engineering/IMG_8839.HEIC) | Governance gap — generation moved faster than control |
| 12 | [IMG_8840.HEIC](slides/addy-osmani-future-of-engineering/IMG_8840.HEIC) | Alpha and Decay definitions |
| 13 | [IMG_8841.HEIC](slides/addy-osmani-future-of-engineering/IMG_8841.HEIC) | Paul Graham tweet on taste |
| 14 | [IMG_8842.HEIC](slides/addy-osmani-future-of-engineering/IMG_8842.HEIC) | Mitchell Hashimoto tweet — Defining Taste |
| 15 | [IMG_8843.HEIC](slides/addy-osmani-future-of-engineering/IMG_8843.HEIC) | Taste summary — judgment before the metric exists |
| 16 | [IMG_8844.HEIC](slides/addy-osmani-future-of-engineering/IMG_8844.HEIC) | Capability decay test — Speed / Recall / Verification / Taste / Judgment |
| 17 | [IMG_8845.HEIC](slides/addy-osmani-future-of-engineering/IMG_8845.HEIC) | Stop asking what AI can't do |
| 18 | [IMG_8846.HEIC](slides/addy-osmani-future-of-engineering/IMG_8846.HEIC) | Everyone is a developer now |
| 19 | [IMG_8847.HEIC](slides/addy-osmani-future-of-engineering/IMG_8847.HEIC) | Cognitive debt definition + Anthropic data |
| 20 | [IMG_8848.HEIC](slides/addy-osmani-future-of-engineering/IMG_8848.HEIC) | Long-horizon agents — delegation depth chart |
| 21 | [IMG_8849.HEIC](slides/addy-osmani-future-of-engineering/IMG_8849.HEIC) | Cognitive surrender + Wharton data |
| 22 | [IMG_8850.HEIC](slides/addy-osmani-future-of-engineering/IMG_8850.HEIC) | Orchestration tax definition |
| 23 | [IMG_8851.HEIC](slides/addy-osmani-future-of-engineering/IMG_8851.HEIC) | Accountability is what lets the rest scale |
| 24 | [IMG_8852.HEIC](slides/addy-osmani-future-of-engineering/IMG_8852.HEIC) | Half life of an edge / signature — decay chart |
| 25 | [IMG_8853.HEIC](slides/addy-osmani-future-of-engineering/IMG_8853.HEIC) | The agent can choose. Only people inherit consequences. |
| 26 | [IMG_8854.HEIC](slides/addy-osmani-future-of-engineering/IMG_8854.HEIC) | High agency — actively taking ownership |
| 27 | [IMG_8855.HEIC](slides/addy-osmani-future-of-engineering/IMG_8855.HEIC) | The Agency Ladder (7 levels) |
| 28 | [IMG_8856.HEIC](slides/addy-osmani-future-of-engineering/IMG_8856.HEIC) | Agents run the inner loop / Engineers own the outer loop |
| 29 | [IMG_8857.HEIC](slides/addy-osmani-future-of-engineering/IMG_8857.HEIC) | The Loop Boundary Is Evidence — inner/outer loop diagram |
| 30 | [IMG_8858.HEIC](slides/addy-osmani-future-of-engineering/IMG_8858.HEIC) | Explain it or don't ship it |
| *(missing)* | IMG_8859.HEIC — not uploaded | *(unknown content)* |
| 31 | [IMG_8860.HEIC](slides/addy-osmani-future-of-engineering/IMG_8860.HEIC) | New Work Is Real Work — engineering moves up a level |
| 32 | [IMG_8861.HEIC](slides/addy-osmani-future-of-engineering/IMG_8861.HEIC) | Closing — every time we made it easier, we wrote exponentially more |

---

## Slide 1 — Central Thesis

> The engineer of the future will choose what is worth doing, then own the evidence, understanding and verdict for work increasingly automated by agents.

---

## Slide 2 — Core Definitions

**Quality** = the system of checks that produces evidence.

**Verdict** = the human/accountable decision made from that evidence.

**Answerability** = the ability to explain and stand behind the verdict later.

---

## Slide 3 — Five Engineering Archetypes (via Boris Cherny, @bcherny)

Addy showed a tweet from Boris Cherny reflecting on role archetypes he observed at the Claude Code team. His framing: as engineering, product, design, DS etc. melt into a new kind of role, roles in the future may look like:

1. **Prototyper** — comes up with brand new ideas; churns out many ideas, most of which don't ship
2. **Builder** — quickly turns a prototype/idea into production-grade product/infra
3. **Sweeper** — cleans up the UI, simplifies the code and system, unships, optimizes performance
4. **Grower** — takes a product that has been built and iterates on it to improve Product-Market Fit
5. **Maintainer** — owns a mature system to make it secure, reliable, fast, and efficient as it scales

"Many people span across 2 roles, and sometimes 3 roles. I also notice that these roles are not really tied to job function — e.g. across Anthropic, some designers match category 1, some 2, some 3; same for engineers, PM, DS."

---

## Slide 4 — The Future of Careers

> Roles are unbundling from craft and rebundling around **ownership**.

**Prototype. Build. Sweep. Grow. Maintain.**

> The title matters less than the part of the system you can own.

---

## Slide 5 — Harness Engineering

Section header: "The agent is the system around the model"

**agent = model + harness**
harness = prompts, tools, state, constraints, feedback loops

Diagram components:
- **MODEL** — reasons / decides ("one chip on the board")
- **CONTEXT** — rules, memory → feeds into model
- **CONTROL** — plans, routing → feeds into model
- **OBSERVE** — tests, logs ↔ model (bidirectional)
- **ACTION** — tools, MCPs ← model output
- **PERSIST** — files, git ← model output
- **HOOKS** — block, retry ↔ model (bidirectional)
- **RATCHET** — new rule (receives from action path)
- **FAILURE** — agent slipped (outer failure path)

Source: addyosmani.com/blog/agent-harness

---

## Slide 6 — Loop Engineering

Section header: "From prompting agents to designing the system that prompts them"

**Design the loop, not the prompt.**

loop = goal + cadence + isolated work + verification + state

Diagram (cycle):
- **VERDICT** — owns outer loop → AUTOMATE
- **AUTOMATE** — cadence finds work → STATE
- **STATE** — memory lives outside → ACT
- **ACT** — agents in worktrees → LEARNING
- **LEARNING** — tomorrow reads today → VERIFY
- **VERIFY** — maker != checker → ISOLATION
- **ISOLATION** — parallel, no chaos → DECIDE
- **DECIDE** — ship, block, queue → back to VERDICT
- Central concept: **RECURSIVE GOAL** — iterate until done

> Loops change the work. They do not delete the engineer.

Source: addyosmani.com/blog/loop-engineering

---

## Slide 7 — Agentic Software Factory

Section header: "The software factory, with the lights on"

Inputs ("stuff worth doing"): product intent / incidents / user feedback

Flow:
1. **agent inner loop**: guide/context → generate → verify/solve (sandbox, traces, tests)
2. → **evidence**: tests, diff summary, risk notes
3. → **human verdict**: ship / block / redirect
4. → **prod** → **users** → **monitor** (feedback loop back to inputs)

"lights off fails here" — marked at the human verdict stage (i.e., fully automated pipelines break down at the accountability step)

> The win is not removing people from the loop.
> The win is moving human judgment to the highest leverage checkpoint.

---

## Slide 8 — The Output Curve: AI Code Share Is No Longer Marginal

Chart — % of code that is AI-generated or significantly assisted:
- 2023: 6%
- 2024: 19%
- now: 42%
- 2026 est.: 55%
- 2027 est.: 65%

Callout: **42%** of committed code is already AI-generated or significantly assisted

> The factory is no longer experimental. It is entering the commit history.

Source: Sonar State of Code Developer Survey report 2026

---

## Slide 9 — Clean Code Is Cheaper for Agents to Read

> Clean code is cheaper for agents to read.

**7–8% fewer tokens. 34% fewer file revisits. Same pass rate.**

---

## Slide 10 — Trust Without Capacity: Reviewers Are Already Overloaded

- **96%** do not fully trust AI-generated code
- **48%** always verify AI code before committing
- **38%** say reviewing AI code takes longer than human code

Callout: **2x trust gap** — skepticism is high, but verification is not keeping up

> The risk is not that engineers distrust AI. It is that they distrust it and still ship faster than they verify.

Source: Sonar State of Code Developer Survey report 2026

---

## Slide 11 — The Governance Gap: Generation Moved Faster Than Control

- **85%** — review/validation is now the bottleneck
- **84%** — governance after creation is the challenge
- **82%** — AI code risks new technical debt
- **80%** — adopted AI faster than policy
- **43%** — cannot reliably distinguish AI vs human code

Callout: **92%** report some governance challenge with AI-generated code

> This is the argument for the HumanVerdict: provenance, intent, and ownership need a first-class interface.

Source: GitLab AI accountability research, June 2026

---

## Slide 12 — Alpha and Decay

> Alpha is the gap.

**Alpha**: what makes you meaningfully better than what current models can do.

**Decay**: how fast the models catch up.

---

## Slide 13 — Taste as Differentiator (via Paul Graham, @paulg)

Addy showed a tweet from Paul Graham (Feb 14, 2026 · 2.1M views):

> Prediction: In the AI age, taste will become even more important. When anyone can make anything, the big differentiator is what you choose to make.

paulgraham.com/taste.html

---

## Slide 14 — Defining Taste (via Mitchell Hashimoto, @mitchellh, Jun 26)

Addy showed a tweet thread from Mitchell Hashimoto:

"'Taste' is the ability to consistently make high-quality qualitative judgments where no objective metric exists. It's the creation of something that feels right intuitively, with no real justifiable way to measure that. But when you do it, people feel it.

A person with 'good taste' is someone who can do this repeatedly, consistently. The funny thing about taste is that it's hard to create, but its result is very easy to copy. Once someone makes a tasteful decision, others can imitate it almost immediately.

This is usually an argument against the existence of taste: 'look how easy I [copy it]'" [slide cut off at bottom]

---

## Slide 15 — Taste Summary

> Taste is the judgment before the metric exists.

**Hard to create. Easy to copy.**

**Valuable because it decides what everyone copies next.**

---

## Slide 16 — The Capability Decay Test

Section header: "The Test: Is it a capability? Then it decays"

| Capability | Status |
|---|---|
| **Speed** | gone — 600 lines while you find coffee |
| **Recall** | gone — it's a context window now |
| **Verification** | automating — and we were the weak link |

"Eventually? Maybe?"

| | |
|---|---|
| **Taste** | alpha. resets every release. Will take longer to decay. |
| **Judgment** | a slope rather than a wall |

---

## Slide 17 — The Right Question

Section header: "One Question: What Can the Agent Do?"

> Stop asking what AI can't do.
>
> Ask what only a human can be answerable for.

---

## Slide 18 — The Engineer vs. The Developer

> Everyone is a developer now. That doesn't make them an engineer.

**What should engineers avoid to stay effective and accountable?**

---

## Slide 19 — Cognitive Debt

**Cognitive debt**: the erosion of your understanding and memory around how to solve problems.

**17%** lower comprehension when you learn through AI-generated code (50% vs 67%).
Source: Anthropic internal study, 52 engineers.

---

## Slide 20 — Long-Horizon Agents: Delegation Depth Is Now Real

Section header: "Long-Horizon Agents"

Task completion rates:
- **>30 min** human-equivalent task: **80.6%**
- **>1 hour** human-equivalent task: **70.2%**
- **>8 hours** human-equivalent task: **25.6%**

Callout: **>60h** agent turns per day for p99 internal daily active users by June 2026

> When tasks become hours long and parallel, the scarce resource is no longer generation. It is answerable delegation.

Source: OpenAI Economic Research, June 25, 2026

---

---

## Slide 21 — Cognitive Surrender

**Cognitive surrender**: when you blindly accept what AI gives you and stop critical thinking.

**73%** accepted the wrong answer when the AI was wrong, and felt more sure.
Source: Wharton, 1,372 people.

---

## Slide 22 — Orchestration Tax

**Orchestration tax**: the diminishing returns and cognitive drain experienced when managing parallel AI agents at once.

---

## Slide 23 — Accountability Scales the Factory

Section header: "Accountability Scales the Factory"

> Accountability is not what remains after agents get good.
>
> It is what lets the rest scale.

---

## Slide 24 — What Decays. What Doesn't.

Section header: "What Decays. What Doesn't."

> The half life of an edge is a release.
> The half life of a signature is a career.

Chart (durability over model releases):
- **accountability** — flat, stays high (does not decay)
- **taste** — decays slowly
- **verification** — decays at medium rate
- **speed** — decays fastest

---

## Slide 25 — Consequences

> The agent can choose.
> Only people inherit consequences.

---

## Slide 26 — High Agency

> High agency is the mindset of actively taking ownership of your outcomes.

---

## Slide 27 — The Agency Ladder

**The Agency Ladder — How much of the problem do you own?**

Pyramid from least to most agency:

| Level | Mode | Example phrasing |
|---|---|---|
| 7 (apex) | **DISCERN** | "Found it — not worth fixing. Moving on." *(the rare apex — knowing what NOT to do)* |
| 6 | **RESOLVE** | "Found it. Fixed it. Looping you in." *(earn your way up to here)* |
| 5 | **RECOMMEND** | "...and here's the one I'd pick." *(live here from day one)* |
| 4 | **PROPOSE** | "...and a few ways to fix it." |
| 3 | **DIAGNOSE** | "Here's the problem, and the cause." |
| 2 | **EXECUTE** | "Hand me the fix — I'll ship it." |
| 1 (base) | **FLAG** | "There's a problem." *(then walks away, leaving it for someone else)* |

---

## Slide 28 — What "High Agency" Means Now

Section header: "What 'High Agency' Means Now"

> Agents run the inner loop.
> Engineers own the outer loop.

---

## Slide 29 — The Loop Boundary Is Evidence

Section header: "What 'High Agency' Means Now"

**The Loop Boundary Is Evidence**
Agents run capability loops. Engineers own agency loops.

**01 — Agent Inner Loop** (capability: do work, return evidence)
- INVESTIGATE: inspect, search, plan
- IMPLEMENT: change, refactor, generate
- TEST: run checks, compare results
- REPORT: summarize what changed

→ BOUNDARY: **EVIDENCE** (diff · tests · logs · why)

**02 — Engineer Outer Loop** (agency: decide what earns production trust)
- DECIDE: is this worth doing?
- VERIFY: is the evidence enough?
- APPROVE: ship, block, or redirect
- OWN: carry the consequence

↩ constraints and learning feed the next run

> The inner loop is capability. The outer loop is accountability.
> Investigate · implement · test · report crosses into decide · verify · approve · own.

---

## Slide 30 — The Rule Operationalized

Section header: "The Rule Operationalized"

> Explain it or don't ship it.

> You cannot answer for what you cannot understand.

---

[NOTE: IMG_8859 / one slide missing here — gap in upload sequence]

---

## Slide 31 — New Work Is Real Work

Section header: "Automation Moves the Floor"

**NEW WORK IS REAL WORK**
The job is not to protect the old loop. It is to own the new one.

Diagram (bottom to top):
- **Automated task layer**: typing · boilerplate · first drafts · routine fixes · repeated checks → fewer keystrokes, **more surface area**
- Label: **NOT LESS ENGINEERING**
- New bottlenecks: coordination · verification · maintenance · product judgment · incident learning
- **ENGINEERING MOVES UP A LEVEL** — own the loops, evidence, systems, and consequences
- Three new pillars:
  - **LOOP DESIGN**: what runs, when, and why
  - **EVIDENCE DESIGN**: what proves it worked
  - **BROWNFIELD CARE**: keep real systems healthy

---

## Slide 32 — Closing Statement

> Every time we made it easier to write software, we ended up writing exponentially more of it.

> The future belongs to engineers who make agent work legible, verifiable, and worth shipping.

---

[END OF BATCH 2 — slides 21–32. One slide missing (IMG_8859). Further slides may follow.]
