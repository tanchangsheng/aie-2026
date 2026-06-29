---
type: concept
tags: [context-engineering, compaction, memory, caching, agents, production]
updated: 2026-06-29
---

# Context Engineering

## Definition

**Context engineering** is the discipline of deciding what the model sees on every single call — instructions, history, retrieved content, memory, and tool outputs. It is the evolved form of prompt engineering: compaction + memory + skills under one discipline.

The two root problems it addresses:
1. **Finite window** — everything competes for one attention budget
2. **Stateless model** — every call starts from zero; nothing persists between sessions

---

## How Speakers Have Framed It

### Towards AI (Bouchard, Vaid, Solano) — Workshop Day 2026

Framed as the 2026 successor to prompt engineering. The key insight: what you *don't* put in context matters as much as what you do.

**The compaction toolkit** (three tiers):
- *Free:* truncation, sliding window, tool-result clearing
- *LM-based:* selective retention, summarization, summary+reset
- *Offload:* write to files, keep pointer in context (Karpathy's "LLM wiki")

**The surprise finding:** in 11–13 turn sessions with modern prompt caching, keeping everything (full_history) beat the production compaction preset on cost ($0.11 vs $0.24/turn), latency (17s vs 21s TTFT), and memory recall (92% vs 38%). Summarization broke cache coherence — rewriting the prefix caused cache misses that cost more than the tokens saved.

**The rule they landed on:** *Don't compact by default. Name the constraint first: window, cost, or throughput.*

---

## Key Agreements Across Sources

*(Only one source so far — will update as more talks are ingested.)*

---

## Contradictions / Open Questions

- At what session length does compaction start to pay? (The study only tested 11–13 turns; longer sessions may flip the result.)
- How does the window/cache trade-off change as context windows grow past 1M tokens?
- Does the "lost in the middle" effect apply to more reasoning-heavy workloads than single-fact retrieval?

---

## Related Concepts

- [KV Cache](kv-cache.md)
- [Compaction](compaction.md)
- [Hybrid RAG](hybrid-rag.md)

---

## Sources

- [Context Engineering in 2026: Compaction, Memory & Cost](../talks/context-engineering-2026.md) — Bouchard, Vaid, Solano (Day 1 Workshop)
