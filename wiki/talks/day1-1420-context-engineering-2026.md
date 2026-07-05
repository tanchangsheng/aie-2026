---
type: talk
tags: [context-engineering, compaction, memory, rag, caching, agents, llm, eval, production]
updated: 2026-06-29
---

# Context Engineering in 2026: Compaction, Memory & Cost

**Speakers:** [Louis-François Bouchard](../speakers/louis-francois-bouchard.md) · [Samridhi Vaid](../speakers/samridhi-vaid.md) · [Omar Solano](../speakers/omar-solano.md)  
**Day:** Day 1 — Workshop Day  
**Time:** 2:20pm–4:20pm  
**Room:** Track 6  
**Type:** Workshop (Sponsor)  
**Format:** 80-minute live workshop with open-source AI tutor as running case study  
**Repo:** https://github.com/towardsai/ai-tutor-app  
**Demo:** https://huggingface.co/spaces/towardsai-tutors/context-engineering-experiments

---

## Official Description

Every long agent session eventually breaks: the assistant that swore it would "never push to main" does exactly that forty turns later. The model didn't get dumber — its context did. This workshop is about engineering the context window so that stops happening, shown with Towards AI's open-source AI tutor. Context engineering is deciding what the model sees on every single call — instructions, history, retrieved course content, memory, and tool outputs — and it's the line between a tutor that holds a coherent session and one that forgets the student's setup halfway through.

---

## Key Claims

- **Compaction is the exception, not the rule** — under modern prompt caching, keeping everything in context is often cheaper and more accurate than compacting it.
- **Summarization can be a trap** — rewriting the prompt prefix kills cache hits. Production sent ~42% fewer tokens than full_history yet paid ~2× more (full_history billed ~87% of input at the 4× cache discount).
- **KV-cache hit rate is the single most important metric** for a production AI agent (per Manus AI).
- **Hybrid retrieval is essential at long context** — dense-only retrieval collapsed at 400k tokens (buried fact recall: 0%); BM25 keyword held at 100%.
- **Local models can't cache their way out** — a 32k context window forces compaction regardless; every method lands in the same 27–40% memory-recall band.
- **Context rot for single-fact retrieval is overblown** — single-fact recall held to 800k tokens in their tests. "Lost in the middle" matters more for reasoning-heavy workloads.

---

## The Core Numbers (Towards AI Tutor, Gemini 3.5 Flash, 11–13 turn sessions)

| Setup | Cost/turn | First token | Memory recall |
|-------|-----------|-------------|---------------|
| full_history (keep everything) | **$0.11** | **17s** | **92%** |
| production (compaction preset) | $0.24 | 21s | 38% |

**DeepSeek V4 Flash (same sessions):**

| Setup | Cost/turn | Memory recall |
|-------|-----------|---------------|
| keep-all | **$0.006** | **95%** |
| summarize | $0.006+ | 32% |

At scale (~10k turns/day): Gemini 3.5 ~$34k/mo, DeepSeek V4 Flash ~$1.9k/mo, local SLM ~$0/token (throughput-bound, ~3 Macs).

---

## The Compaction Toolkit (3 tiers)

**Tier 1 — No LLM call required (free):**
- Observation truncation — cut huge tool outputs to head + tail on arrival
- Trim / sliding window — drop oldest N turns continuously
- Tool-result clearing — swap old outputs for re-fetchable placeholders

**Tier 2 — Spend tokens to save tokens (LM-based):**
- Selective retention — keep constraints/decisions, drop dead ends
- Summarization — running gist, lossy by design
- Compaction (summary + reset) — collapse history into one fresh summary; what Claude Code does near ~95% fill

**Tier 3 — Offload (relocate, don't delete):**
- Write to files / memory, keep a pointer in context (Karpathy's "LLM wiki" pattern)
- The file system becomes cheap, durable, inspectable context

---

## The Production Architecture (Towards AI Tutor)

```
Next.js UI → FastAPI POST /api/chat → LangChain agent (InMemorySaver)
    → middleware: summarize · clear tool-outputs · source preference
    → retrieve_tutor_context (hybrid RAG)
    → run_kb_command (read-only shell over KB)
    → Gemini 3.5 Flash
```

**Grounding corpus:** 8.62M tokens, 3,203 docs, 14 sources (5 courses + 9 doc sets).

**Hybrid RAG pipeline:** student question → scope to source → dense (embed-v4, top 15) + BM25 (top 30) → RRF merge to 45 → Cohere rerank to top 5 (drop <0.10) → token budget (fill to 100k).

**KB tool:** read-only shell (`rg`, `grep`, `find`, `cat`, etc.) over mirrored markdown corpus + machine-generated indexes + 29-page wiki. Used in ~89% of turns with current system prompt; turning it OFF is 50% faster with same retrieval recall.

---

## The Eval Harness

**Two test sets from real students:**
- *Single-turn:* 60 questions asked once — measures retrieval, key facts, response type
- *Sessions:* 11–13 turns — plants a fact at turn 0, probes at turn 11 (compaction must fire in between to count)

**Flow:** `run_battery` (real agent, costs $) → `grade` (code checks + LLM judge) → `check_triggers` (gate: compaction fired?) → `report`

LLM judge matched human grades 98% on 96 probes. Total Gemini study cost: ~$590, 660 turns, 0 API errors.

---

## What Every Serious Harness Converged On (2026)

| System | Approach |
|--------|----------|
| **Claude Code** | Microcompact (no LLM) every turn; 9-section structured summary at ~95% fill; CLAUDE.md survives compaction |
| **Codex** | Local LLM writes handoff summary; tries structured session memory first; fires pre-turn AND mid-turn |
| **Anthropic/OpenAI APIs** | Server-side `context_management`; compaction block returned, system prompt stays cached |

Common additional practices: fresh sessions when switching tasks, narrow file context, disconnect unused tools, model router by complexity, log everything (LangSmith, Opik…).

---

## Notable Quotes

> "The model didn't get dumber. Its context did."

> "If I had to choose just one metric, I'd argue that the KV-cache hit rate is the single most important metric for a production-stage AI agent." — Manus AI

---

## Reactions

The headline finding is counter-intuitive: compaction made things worse in the production preset — higher cost, slower TTFT, lower memory recall — because it broke cache coherence. This is a concrete empirical argument against a received wisdom (compress your context aggressively) that most teams take for granted.

The 32k local model result is sobering: the window is the hard constraint, not model quality, and no compaction method escapes the band. That makes the cloud/local tradeoff primarily about privacy and throughput, not cost at moderate scale (they found 3 Macs sufficient for ~1,300 turns/day/GPU).

The Karpathy "LLM wiki" framing as cheap durable context is the most transferable idea — it's also how this very wiki operates.

---

## Linked Concepts

- [Context Engineering](../concepts/context-engineering.md)
- [KV Cache](../concepts/kv-cache.md)
- [Compaction](../concepts/compaction.md)
- [Hybrid RAG](../concepts/hybrid-rag.md)
