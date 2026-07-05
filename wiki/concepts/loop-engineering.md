---
type: concept
tags: [loop-engineering, agents, verification, isolation, verdict, agentic-sdlc]
updated: 2026-07-05 (1 source)
---

# Loop Engineering

## Definition

The discipline of designing the systems that prompt agents — rather than writing individual prompts.

"Design the loop, not the prompt."

`loop = goal + cadence + isolated work + verification + state`

## The Loop Architecture

Osmani's diagram is a cycle with a recursive center:

**Outer cycle** (clockwise from verdict):
- **VERDICT** — owns the outer loop; decides ship/block/queue
- **AUTOMATE** — cadence finds work
- **STATE** — memory lives outside the agent (persistent, not in-context)
- **ACT** — agents in worktrees (isolated execution)
- **LEARNING** — tomorrow reads today (loop output feeds next run)
- **VERIFY** — maker ≠ checker (the agent that generates does not verify)
- **ISOLATION** — parallel runs without chaos
- **DECIDE** — ship, block, or queue

**Center**: RECURSIVE GOAL — iterate until done

## Core Principles

**Maker ≠ checker.** The agent that generates work should not be the agent that verifies it. Verification by the same system that produced the output cannot catch systematic errors in that system.

**State lives outside the model.** The loop's memory — what was tried, what was learned, what's queued — must persist outside any single model context window. This is what makes the loop durable across sessions and model boundaries.

**Isolation enables parallelism.** Agents run in worktrees (isolated branches/environments) so parallel runs don't corrupt each other. This is the agent-level equivalent of the devbox isolation pattern described in [Agent Environment Architecture](agent-environment-architecture.md).

**Loops change the work; they do not delete the engineer.** The VERDICT node is owned by the engineer. The engineer sets goals, decides what earns production trust, and blocks or redirects when the evidence is insufficient.

## Relationship to Harness Engineering

Loop engineering operates one level above harness engineering. A harness is the scaffolding around a single model invocation; a loop is the system that orchestrates many harness invocations toward a goal. Multiple harnesses can participate in a single loop.

## Relationship to the Agentic Software Factory

The loop is the operational unit of the [Software Factory](software-factory.md). The factory's pipeline — intent → agent inner loop → evidence → human verdict → prod — is an instance of loop engineering applied to the full software development cycle.

## Tension with Ad Hoc Prompting

Most teams today operate by writing better prompts and iterating manually. Loop engineering is the claim that this doesn't scale — that the bottleneck shifts from "can the model do this?" to "is the system designed to make this repeatable, verifiable, and improvable?" The two approaches are not equivalent.

## Open Questions

- At what task complexity does loop-level design start yielding returns over prompt-level improvement?
- How do you test a loop design before running it at scale? Is there a "loop lint"?
- The maker ≠ checker principle — how do you prevent the checker from being trained on or biased by the maker's style?

## See Also

- [Harness Engineering](harness-engineering.md) — the single-invocation scaffolding inside the loop
- [Human Verdict](human-verdict.md) — the VERDICT node that owns the outer loop
- [Software Factory](software-factory.md) — the loop applied to the full dev cycle
- [Agent Environment Architecture](agent-environment-architecture.md) — the isolation substrate for ACT nodes
- [The Future of Engineering (talk)](../talks/day3-1630-future-of-engineering.md)

## Sources

- [The Future of Engineering — Addy Osmani](../talks/day3-1630-future-of-engineering.md) (addyosmani.com/blog/loop-engineering)
