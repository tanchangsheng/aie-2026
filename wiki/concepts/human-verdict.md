---
type: concept
tags: [human-verdict, accountability, answerability, quality, governance, agentic-sdlc]
updated: 2026-07-05 (2 sources)
---

# Human Verdict

## Definition

The accountable human decision at the boundary between agent-generated evidence and production. The point where a person — not an agent — decides to ship, block, or redirect.

Three terms constitute the full framework:

- **Quality** = the system of checks that produces evidence
- **Verdict** = the human/accountable decision made from that evidence
- **Answerability** = the ability to explain and stand behind the verdict later

## Why It Matters — The "Lights Off" Problem

In an Agentic Software Factory, the tempting goal is to remove humans from the loop entirely ("lights off"). Osmani argues this fails specifically at the verdict node. The pipeline — product intent → agent inner loop → evidence → human verdict → prod — works end-to-end only because a human stands at the evidence/prod boundary.

"Lights off fails here" is marked at the human verdict stage. The reason: provenance, intent, and ownership are not derivable by inspection of agent output alone. Someone must be accountable for the decision to ship. That accountability cannot be delegated to the agent that generated the work.

## The Governance Data

The urgency of the human verdict concept is backed by a governance crisis:

- 92% of teams report some governance challenge with AI-generated code (GitLab AI accountability research, June 2026)
- 85% say review/validation is now the bottleneck
- 43% cannot reliably distinguish AI vs human code
- 96% do not fully trust AI-generated code, but only 48% always verify it before committing

The gap: skepticism is high but verification is not keeping up. This is the "trust without capacity" problem — engineers distrust AI code and still ship faster than they can verify.

## The Inner/Outer Loop Split

Osmani operationalizes the verdict concept as the outer loop:

**Agent inner loop** (capability): INVESTIGATE → IMPLEMENT → TEST → REPORT → *evidence*

**Engineer outer loop** (accountability): DECIDE → VERIFY → APPROVE → OWN

The boundary between loops is evidence: a diff, test results, logs, and a "why." The outer loop's OWN step is the verdict made concrete — "carry the consequence."

"Explain it or don't ship it. You cannot answer for what you cannot understand."

## The Agency Ladder and Verdict

The Agency Ladder maps the same concept at the individual level. The highest levels of agency (RESOLVE, DISCERN) are defined by owning the consequence of a decision — not by executing work. The lowest level (FLAG) identifies a problem but doesn't carry the verdict. Osmani says engineers should live at level 5 (RECOMMEND) from day one — always providing a verdict, not just evidence.

## Win Condition

The win is not removing people from the loop. The win is moving human judgment to the highest leverage checkpoint.

This is a reframe of the automation question: the goal isn't fewer humans, it's humans operating at higher abstraction — choosing what is worth doing, setting goals for loops, rendering verdicts on agent-generated evidence.

## Implementations

**AI Code Review (uReview, Uber):** Agents generate and grade code review comments; a human engineer renders the final verdict on whether to approve or reject a PR. uReview's inner/outer loop split maps directly onto this framework. See [AI Code Review](ai-code-review.md).

**Production Operations (Resolve AI):** Justin Smith's closing takeaway at AIE is the clearest operational instance of the pattern: "You open Resolve AI to verify findings, and stand up new work in a sentence." The agent monitors production continuously — deploy checks, health scans, incident digests — and surfaces findings. The engineer's role is to open the tool, verify, and decide. This extends the human-verdict pattern from the dev cycle (Osmani, uReview) into the operational lifecycle, and gives it the most minimal possible interface: the human enters only at the verdict step, not the investigation step.

## Open Questions

- What does "explain it" look like as an interface requirement? What evidence package must accompany an agent-generated PR for a human to give a meaningful verdict?
- At what agent reliability level (task success rate) does the human verdict become a bottleneck that limits factory throughput more than it protects quality?
- Can verdict rendering itself be partially assisted by AI — e.g., an agent that summarizes evidence and flags risks, without making the final call?

## See Also

- [Loop Engineering](loop-engineering.md) — the VERDICT node in the outer loop
- [Software Factory](software-factory.md) — the factory pipeline that culminates in human verdict
- [AI Code Review](ai-code-review.md) — a concrete implementation at Uber
- [Harness Engineering](harness-engineering.md) — the inner-loop system that generates evidence
- [The Future of Engineering (talk)](../talks/day3-1630-future-of-engineering.md)

## Sources

- [The Future of Engineering — Addy Osmani](../talks/day3-1630-future-of-engineering.md)
- [Always-on agents run production without the on-call tax — Justin Smith (Resolve AI)](../talks/day4-1425-always-on-agents-production-on-call-tax.md)
