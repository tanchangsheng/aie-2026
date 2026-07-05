---
type: concept
tags: [code-review, multi-agent, generator-verifier, uber, developer-tools]
updated: 2026-07-01
---

# AI Code Review

## Definition

An architecture for running AI-generated code review at organizational scale without either bottlenecking on human reviewers or drowning developers in noisy, hallucinated comments. Uber's instance of this is **uReview**: a single review engine that serves two different audiences (agents and humans) through different interfaces, routes work across a mix of generic and team-customized review generators, and grades its own output before it reaches a PR.

## The Inner Loop / Outer Loop Split

The core design decision is separating consumers of review output, not generators of it:

| | Inner Loop | Outer Loop |
|---|---|---|
| Audience | Agents (agent loop, no human visibility) | Humans (GitHub/Phabricator reviews) |
| Interface | Purpose-built API / CLI | Native UI & webhooks |
| Accuracy bar | Very high — avoid cavitation *(as written on the slide; term unconfirmed, see raw note)* | High — no false positives |
| Feedback signal | Fix rate | Replies & reactions |

The same engine produces both, but the tolerance for noise and the feedback mechanism differ — an agent consuming review output in a loop needs a different interface and a stricter false-positive bar than a human reading GitHub comments.

## The Review Stack

Two axes: generic vs. team-customized, and single-file vs. multi-file scope.

| | Generic | Customized |
|---|---|---|
| Single File | General Purpose — zero-shot prompt, research-based bug/logic-error finding | AI Linters — few-shot, team-authored rules scoped to one concern |
| Multi File | Deep Reviews — GEPA-tuned agent skill, in-depth multi-file review | Custom Agents — skill + knowledge base, domain-specific team guidelines |

Implementation rests on three legs: **Ownership** (rules live co-located in the repos they govern, on the existing system — not centrally authored), **Dispatch** (deterministic routing by risk profile and complexity, not model judgment), and **Observability** (volume, cost, sentiment, addressal rate, agent trajectory).

![The Review Stack slide](../../raw/slides/ureview-scaling-code-quality-uber/slide-03-review-stack.jpg)

## Architecture

GitHub, Phabricator, and an Agent Loop (gRPC) all feed a central Dispatch stage, which routes to Generators (Single Prompt, Agent, Third-Party), through Post-Process (rate, categorize, deduplicate), back out to the Outer Loop (native review) or Inner Loop. A separate Feedback path (reactions, replies, sentiment) writes to an Eval Dataset that loops back into the Generators, closing the loop between how a comment performed and how future comments get generated.

![uReview Architecture slide](../../raw/slides/ureview-scaling-code-quality-uber/slide-04-architecture.jpg)

## Generator-Verifier Pattern, Confidence Scoring, and Deduplication

*(Source: AIE MCP official session description — not confirmed against the captured slides, which didn't show this part of the deck.)*

Uber frames the trust-layer problem as: a primary agent generates suggestions, a secondary high-reasoning "verifier" model audits them against coding guidelines before anything reaches a PR. Every comment gets a numerical confidence score; comments below a calibrated threshold are silently dropped rather than shown. Overlapping warnings from different sources (e.g., a static analyzer and an LLM agent both flagging the same null pointer) are semantically merged into one comment instead of posted twice. Success is tracked via **Actionability Rate** (% of AI comments accepted as commits) and reduction in **Mean Time To Merge** — explicitly not raw comment count, which the abstract calls a vanity metric.

## Relationship to Other Concepts

- **[Agent Skills](agent-skills.md)** — uReview's "Custom Agents" are tagged "Skill + Knowledge Base" on the Review Stack slide. It's plausible this reuses Uber's org-wide Agent Skills infrastructure (`SKILL.md`, the `aifx` toolchain) described in the Agentic SDLC talk, but the uReview slides don't confirm this explicitly — treating as an open question rather than an assumed link.

## Open Questions

- Does the Generator-Verifier pattern from the official abstract map onto any specific box in the uReview Architecture diagram (most likely inside Post-Process, but not confirmed)?
- Do "Custom Agents" in the Review Stack actually run on Uber's general-purpose Agent Skills infrastructure, or is this a separate skill system specific to code review?
- What is "GEPA-tuned" tuning — GEPA appears to be a specific prompt/agent optimization method; not explained further on the captured slides.
- What does "avoid cavitation" mean in the Inner Loop accuracy row — likely a mis-transcription or Uber-internal term; unconfirmed.
- How is the confidence-scoring threshold calibrated, and does it differ between Inner and Outer Loop given their different accuracy bars?

## Sources

- [Scaling Code Quality: Building uReview, Uber's Multi-Agent Code Review Engine](../talks/day2-1205-ureview-uber.md) — Will Bond, Ameya Ketkar
