---
type: concept
tags: [harness-engineering, agents, prompts, tools, state, feedback-loops, agentic-sdlc]
updated: 2026-07-05 (1 source)
---

# Harness Engineering

## Definition

The discipline of designing the scaffolding that turns a model into an agent.

`agent = model + harness`

The harness is everything the model is embedded in: prompts, tools, state, constraints, and feedback loops. The model reasons and decides; the harness controls what it sees, what it can do, and what happens with the results.

## The Harness Architecture

Osmani's diagram names the components:

| Component | Role |
|---|---|
| **MODEL** | Reasons and decides — "one chip on the board" |
| **CONTEXT** | Rules, memory — feeds into model |
| **CONTROL** | Plans, routing — feeds into model |
| **OBSERVE** | Tests, logs — bidirectional with model |
| **ACTION** | Tools, MCPs — receives model output |
| **PERSIST** | Files, git — receives model output |
| **HOOKS** | Block, retry — bidirectional with model |
| **RATCHET** | New rule — fed back into CONTEXT from the action path |
| **FAILURE** | Agent slipped — outer failure path |

The key insight: the model is fixed hardware ("one chip on the board"). Everything else — the loop it runs in, the context it receives, the tools it can call, the hooks that constrain it, the state it writes — is the engineer's design surface.

## Why It Matters

The shift in thinking: prompting a model is a user activity. Engineering the harness is an infrastructure activity. Teams that treat harness design as an afterthought (or as equivalent to prompt engineering) are designing the wrong thing.

This parallels the inference-layer lesson from the conference: the model is not the unit of performance. The system the model runs in is.

## Open Questions

- At what scale does harness configuration management become its own engineering domain (version control, testing, rollback for harness components)?
- How should harness components be tested independently vs. end-to-end?
- Is there a "clean harness" equivalent to clean code — i.e., do simpler harnesses make agents more reliable, or does complexity reflect genuine system requirements?

## See Also

- [Loop Engineering](loop-engineering.md) — designing the system that runs harnesses
- [Agent Environment Architecture](agent-environment-architecture.md) — the execution substrate harnesses run on
- [Agent Skills](agent-skills.md) — portable, eval-gated units of expertise that plug into harnesses
- [The Future of Engineering (talk)](../talks/day3-1630-future-of-engineering.md)

## Sources

- [The Future of Engineering — Addy Osmani](../talks/day3-1630-future-of-engineering.md) (addyosmani.com/blog/agent-harness)
