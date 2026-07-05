# Context Engineering in 2026: Compaction, Memory & Skills

**Presenters:** Louis-François Bouchard · Samridhi Vaid · Omar Solano  
**Organization:** Towards AI  
**Repo:** https://github.com/towardsai/ai-tutor-app  
**Source:** Text notes — no slide photos taken.

---

## The Problem: Context Rot

Every long agent session, ever:

```
turn 1     you:    "One rule: NEVER push directly to main."
turn 2     agent:  "Understood — feature branches only."
           ···  45 turns of greps, diffs and logs ···
turn 47    agent:  $ git push origin main
                   "Done! Pushed straight to main."
turn 48    you:    "NO FFS I TOLD YOU NOT TO F***ING DO IT. Switching to claude"
```

> **The model didn't get dumber. Its context did.**

---

## Workshop Agenda (80 minutes)

- **Part 1** — Compaction, memory, retrieval, skills (Louis-François): the techniques + why the tutor needs them
- **Part 2** — The architecture + eval harness on Gemini 3.5 Flash (Omar): system design + measuring tokens, cost, latency, caching
- **Part 3** — How to match the quality at a fraction of the cost (Samridhi): open & local models, experiments, compaction and how they chose

---

## Speakers

- **Louis-François Bouchard** — Co-founder & CTO @ Towards AI (2020–), "What's AI" on YouTube, Author of *Building LLMs for Production* (2024), ex-Mila PhD(c)
- **Omar Solano** — Senior ML Engineer @ Towards AI, B.Eng. in Automated Production Engineering
- **Samridhi Vaid** — Senior ML Engineer @ Towards AI, M.Sc Computer Science

---

## Part 1: Context Engineering Techniques

### Case Study: The AI Tutor

**Five requirements:**
1. Answers grounded in their content, not general knowledge
2. Stay in student's current lesson, not any random course
3. Hold long help sessions: debugging, follow-ups, "explain that again"
4. Handle code, read the student's, generate working examples
5. Stream fast — latency is UX for a tutor

→ **Context engineering (in 2026)**

---

### The Core Problems

**Finite context window** — everything the model sees competes for one attention budget:
- Instructions, retrieved lessons, tool use, code…

**The model is stateless** — every call starts from zero; between sessions, nothing persists.

→ Within a session: **context management** | Across sessions: **memory**

---

### What the Model Actually Sees, Every Call

The context window has a cost hierarchy — some components are expensive to include:

```
┌─────────────────────────────────────────┐
│ $$$  chat history                       │
│      - old tool outputs                 │
│      - (old) old lesson chunks          │
│      - searches + files it opened       │
│      - tool-call + tool-result pairs    │
│                                         │
│  $$  course chunks + memory             │
│                                         │
│   $  system prompt                      │
│      tool definitions                   │
│      the student's question             │
└─────────────────────────────────────────┘
          ↑ the window
```

---

### Why Long Context Hurts

**Quality:**
- Recall sags for facts buried deep in the window ("lost in the middle")
- "Context rot" — quality degrades as the window fills

**Cost & Latency:**
- The whole history is re-sent every turn — you re-pay for every token, every turn
- TTFT (time to first token) climbs with input size — the model reprocesses the entire window each call

→ Manage for **spend & speed** — not (only) because quality drops

---

## Compaction: Keep What Still Matters. Drop or Relocate the Rest.

Three tiers of compaction strategy:

```
Trivial tools  →  Spend to save  →  Offload
(free)            (LM-based)        (relocate)
```

---

### Tier 1: Cheap — No LLM Call Required

**Observation truncation (outliers)**
- Cut one huge tool output to head + tail before it enters history
- *When:* one result is huge but only input and outcome matter — e.g., a 300-line stack trace
- Fires the moment one huge output arrives

**Trim / sliding window**
- Drop the oldest turns, or keep the last N with overlap for continuity
- *When:* old turns no longer matter — previous unrelated bugs or questions
- Runs continuously

**Tool-result clearing**
- Swap old outputs for a re-fetchable placeholder, keep the call record
- *When:* lots of temporary outputs dominate — stale course-chunk retrievals
- The safest first move; fires later once output is stale

---

### Tier 2: Spend Tokens to Save Tokens (LM-based)

**Selective retention**
- Keep constraints, decisions, open tasks; drop exploration & duplicates
- *When:* you can judge what matters — keep the student's goal & env, drop dead ends

**Summarization**
- Continuously replace old history with a running gist — lossy by design
- *When:* the gist is enough — what the student already understood, not every word

**Compaction (summary + reset)**
- Collapse the whole history into one fresh summary, then start over from it
- e.g., when Claude Code reaches its context limit
- ⚠️ *Careful:* spend-to-save can backfire

---

### Tier 3: Move Tokens Out of the Window (Offload)

**Offload to files / memory**
- Persist details to disk or a memory tool; keep only a pointer in context
- *Reversible* — nothing is lost; retrieval pulls it back in just-in-time

**Karpathy's "LLM wiki"**
- A file tree the agent maintains and re-reads: `CLAUDE.md`, `AGENTS.md`, a growing KB
- \+ RAG DB!
- The file system becomes cheap, durable, inspectable context

**GraphRAG / course index** — ties plain RAG

---

### A Recipe from Production

```
session/
  01-auth-research.md     ← chunk
  02-db-decisions.md      ← links to 01
  index.md                ← the map
```

1. Write chunks to files, cross-linked with pointers
2. Keep one index file that maps them all
3. Reset the agent — only the index goes back in
4. It searches its way back to details, per task, limiting context pollution

→ **Kilobytes in context** — agent pulls context according to task complexity

> The same shape Claude Code & friends converged on: **durable files + a small live window.**

---

### Skills Parenthesis: Load Instructions Like Code

- More small, focused skills: `explain-concept` · `evaluation` · `review-my-code` — not one giant file
- **Progressive disclosure** — names/small descriptions always visible, bodies load only on use

---

### The Problem with Compaction (The Cache Trap)

> Reference: https://x.com/its_ao/status/2070556265906917860

**Summarization can be a trap:** rewriting the prefix **throws the cache away**.

---

### Caching: A Game-Changer

**Implicit (automatic, default-on):**
- Gemini 2.5+: ~90% off
- DeepSeek: $0.14 → $0.0028/M (≈50×)

**Explicit (you pin it):**
- Anthropic `cache_control` / Gemini `cache`
- Guaranteed, but TTL + storage rent

Not only helps with **cost** but also greatly reduces **latency**.

> As Manus AI puts it: *"If I had to choose just one metric, I'd argue that the KV-cache hit rate is the single most important metric for a production-stage AI agent."*

---

### 2026: Every Serious Harness Converged Here

**Claude Code** — five mechanisms:
- Microcompact (no LLM) trims tool outputs every turn
- Near ~95% context fill: a 9-section structured summary replaces history; files/skills/plan re-injected after
- The summary is readable & editable — and `CLAUDE.md` survives compaction

**Codex** — one move, summarize → replace:
- A local LLM writes a 'handoff summary' (prompt is in the codex-rs repo)
- For OpenAI models, `/responses/compact` returns an encrypted blob
- Fires pre-turn AND mid-turn; tries structured session memory first, full LLM summary only if needed

**The APIs themselves** — Anthropic & OpenAI both ship it server-side:
- Set `context_management` → a compaction block comes back, old turns dropped, system prompt stays cached
- Same model writes the summary — accurate recall matters more than speed or cost

**Best practices (they all share):**
- Start fresh sessions when switching tasks
- Scope file context narrowly
- Disconnect unused tools
- Compact
- Optimize for cache hits (model providers improve caching frequently)
- Use a model router based on task complexity
- Log everything (e.g., LangSmith, Opik…) and monitor: cache hit rate, user frustration, abnormally long outputs

---

### The Definition: Context Engineering

> **All of it together is context engineering: deciding what the model sees, every single call.**
>
> Prompt engineering grew up: compaction + memory + skills under one discipline.

---

### The Pattern Engineered Toward

**Keep in the window** (optimized with caching):
- The student's goal, their environment (current course, lesson, code setup/repo), decisions made, the current error…

**Drop or compact** (using various techniques):
- Old stack traces → truncate
- Dead-end attempts → selective retention
- Chit-chat → trim…

**Persist across sessions** (memory):
- Level & progress
- The session's gist…

---

## Part 2: The Tutor — Built & Measured

### Architecture: One Request End to End

```
┌─────────────┐     POST /api/chat     ┌──────────────┐
│  Next.js UI │ ────────────────────▶  │   FastAPI    │
│ browser chat│ ◀────────────────────  │              │
└─────────────┘      stream            └──────┬───────┘
                                              │ request
                                    ┌─────────▼──────────────┐
                                    │       ONE AGENT         │
                                    │  LangChain create_agent │
                                    │  + InMemorySaver        │
                                    │                         │
                                    │  middleware stack:       │
                                    │  summarize · clear      │
                                    │  tool-outputs ·         │
                                    │  source preference      │
                                    │                         │
                                    │  ┌──────────────────┐   │
                                    │  │retrieve_tutor_   │   │
                                    │  │context (hybrid   │   │
                                    │  │RAG)              │   │
                                    │  └──────────────────┘   │
                                    │  ┌──────────────────┐   │
                                    │  │run_kb_command    │   │
                                    │  │(browse the KB)   │   │
                                    │  └──────────────────┘   │
                                    │                         │
                                    │    Gemini 3.5 Flash     │
                                    └─────────────────────────┘
```

> The agent is small. The grounding — the course and docs knowledge base — is what makes it a tutor.

---

### Grounding Tool 1: `retrieve_tutor_context`

**Corpus:** 8.62M tokens, 3,203 docs, 14 sources (5 courses + 9 doc sets). No window holds it — embed once, retrieve per question.

```
the student's question
         │
         ▼
  scope to source          ← selected source only, lifts precision
         │
         ▼
   hybrid search           ← dense embed-v4 top 15 + BM25 top 30
         │
         ▼
   fuse + rerank           ← RRF to merge to 45, Cohere rerank to top 5, drop <0.10
         │
         ▼
   token budget            ← fill up to 100k, into the window
```

> Top-k is precise but narrow. What if the answer is spread across the corpus?  
> Reference: https://arxiv.org/pdf/2605.05242

---

### Grounding Tool 2: `run_kb_command` — Browse the KB

**Knowledge base structure:**
```
raw/           3,203 markdown mirrors (254 course + 2,949 docs)
generated/     machine indexes for greps: doc manifest, headings, code symbols
wiki/          29 pages: 11 topics, 9 frameworks, 5 courses
```

**A read-only shell over the corpus:** `rg`, `grep`, `find`, `ls`, `sed`, `head`, `cat`, `wc`

**Limits:**
- 8-second timeout
- Output capped at 40,000 chars
- Max 20 commands/turn
- Sandboxed to `data/kb/`

**Built offline, read at runtime:** an agent maintains the wiki via `data/kb/MAINTAINER.md` when sources change; the tutor only reads it.

**Key finding:**
- With current system prompt, browse is the main grounding path (~89% of turns)
- Turning `run_kb_command` OFF → tutor is **50% faster** (~19s vs ~38s) with **same retrieval recall**

---

### Context Management: The Middleware Stack

```
Production preset — 3 middlewares in sequence:

1. clear stale tool-outputs    ← at 5k tokens, keeps retrieval + latest 5
2. summarize old turns         ← at 30k tokens, keeps last 20
3. source preference           ← scopes the corpus
```

---

### The Eval Harness

**Demo:** https://huggingface.co/spaces/towardsai-tutors/context-engineering-experiments

**Telemetry logged per turn:**
- Input / output tokens · cached
- Estimated $ · time-to-first-token
- Model calls · did compaction fire?

---

### Vocabulary

| Term | Definition |
|------|-----------|
| **Preset** | A specific setup for the AI tutor tested; everything else held fixed |
| **Task type** | One test set for one specific task (single-turn or sessions) |
| **Run** | One preset × one model × one task type |
| **Bundle** | Saved JSON for one turn: answer, tools, sources, tokens, timing, did-compaction-fire |

---

### Test Sets: Tasks from Real Students

**Single-turn** (60 questions, asked once)
- *Measures:* retrieval (auto-graded), right facts (key points), right kind of response (teach/redirect)
- *Example:* "repo only has lessons 4,5,6,8,10 — is that expected?"
- From 151 real academy posts; kept 60 after cleaning/reviewing

**Sessions** (multi-turn, 11–13 turns)
- *Measures:* in-session memory under compaction
- *Example:* plant "I want to learn about RAG" at turn 0, probe it at turn 11
- Invented facts + middle filler messages from real academy posts

---

### One Session Probe, Start to Finish

```
Turn 0 (plant)
"I'm a Unity dev. My weak topic is RAG evaluation. Dialogue must stream within 300ms."

Turns 1–10 (grow)
Real course questions push the chat past the 30k trigger → compaction fires.

Turn 11 (probe)
"One free evening, which topic do I drill and name two metrics?"

Grade
expected: RAG evaluation, hit rate, MRR

Gate
check_triggers confirms compaction fired before turn 11;
if not → run rejected (probe would not test memory)
```

---

### The Harness: Run → Grade → Gate → Report

```
┌─────────────┐     ┌─────────┐     ┌────────────────┐     ┌──────────┐
│ run_battery │────▶│  grade  │────▶│ check_triggers │────▶│  report  │
│             │     │         │     │                │     │          │
│ drives the  │     │ code    │     │ THE GATE: did  │     │ side-by- │
│ real agent  │     │ checks  │     │ compaction fire│     │ side     │
│ 1 bundle/   │     │ (free,  │     │ before probe?  │     │ tables + │
│ turn        │     │ offline)│     │                │     │ token-   │
│             │     │ + LLM   │     │ NO → reject    │     │ by-turn  │
│ only step   │     │ judge   │     │    the run     │     │ curves   │
│ that costs $│     │ (sub.)  │     │                │     │          │
└─────────────┘     └─────────┘     └────────────────┘     └──────────┘
```

> The judge sees only question, answer, and grading criterion — cannot favor a strategy. Matched human grades **98%** on 96 probes.

---

### What Ran: 11 Presets, One Variable

- 2 reference points: `full_history` (keep everything) and `production` (live default)
- 6 one-change variants: `sliding-window`, `prompt-compression`, `selective-retention`, `context-reset`, `in-context-history-retrieval`, `profile_memory`
- Everything else held fixed: model, prompt, retrieval, sources, dataset
- 660 turns, 0 API errors
- ~$590 across the whole Gemini study

---

### Results: Session Memory Recall by Method

**Broad screen · 8 presets · 1 trial (3 sessions) ~$88.12**

| Preset | Recall |
|--------|--------|
| full_history | 100% |
| prompt-compression | 100% |
| in-context-retrieval | 83% |
| profile_memory | 75% |
| production | 58% |
| sliding-window | 42% |
| selective-retention | 25% |
| context-reset | 17% |

**Deeper exp · 3 presets · 2 trials (3 sessions) ~$62.17**

| Preset | Recall |
|--------|--------|
| full_history | 92% |
| aggressive | 42% |
| production | 38% |

→ Keeping everything wins memory in both runs.

---

### The Surprise: Compaction Didn't Pay

**Sessions, 11–13 turns:**

| | full_history | production |
|--|--|--|
| Cost / turn | **$0.11** | $0.24 |
| First token | **17s** | 21s |
| Memory recall | **92%** | 38% |

> **Keep-everything wins all three.**

**Why production loses:**
1. Summarizing **rewrites the cached prefix** → cache miss. Production sends ~42% fewer tokens than full_history yet pays ~2× more (full_history bills ~87% of input at the 4× cache discount).
2. Cleared tool-outputs get **re-retrieved** — pay, drop, pay again.

> These are 11–13 turn sessions → can't justify compaction. That sets up Part 3.

---

## Part 3: When Does Compaction Actually Matter?

### First: What Fills the Context?

| Type | Break pattern | Fix |
|------|--------------|-----|
| Long chat history | Detail gets buried | Different per case |
| Pasted document | Too big to fit the window | RAG / retrieve |
| Big tool output | Pages of useless verbatim logs | Truncate |

Each type needs a **different fix**.

---

### First Lever: A Cheaper Model

| Model | Cost / turn | Cache discount | Keep-all wins? |
|-------|------------|----------------|----------------|
| Gemini 3.5 Flash | ~$0.11 | ~10× | yes |
| DeepSeek V4 Flash | ~$0.006 | ~50× | yes |

Same test, swap the model → ~18× cheaper per turn, and keep-all **still wins**.

---

### The Cheapest Run Sends the Most Tokens

Keep-all bills the most tokens (296k/turn) yet is the cheapest setup — **97% cached**.  
At 36 turns: 1.78M tokens.

---

### Memory: Keep-All vs. Summarize

Keep-all **95%** vs. summarize **32%** — summarizing drops the detail a later question needs.

---

### Does It Hold at Long Context?

Single-fact recall held to **800k tokens** (1M is a hard wall). The job is single-fact retrieval, not long-history reasoning — by design.

---

### At Scale, Compaction Is Economics

Cheap per turn isn't cheap at scale: **$18k–180k/mo** at 100k–1M turns/day, even on DeepSeek.

---

### Going Local: Can We Just Cache?

- At scale, the bill pushes running your own model
- Own hardware = small model, ~32k context window
- The cloud's move was keep-everything + caching — locally, **it doesn't fit**
- Lessons are already bigger than 32k; students paste long logs that blow past it
- No fit → nothing to cache → **forced to compress or retrieve**

---

### Local Results: Every Method Lands in the Same Band

Forced to compact, every method lands in a **27–40% band**. A 4–5× bigger 32B model stays in it. The 32k window is the constraint, not the model.

---

### Local Docs: Retrieve, Don't Stuff

Stuffing one long doc overflows the window at every size (1-token stub).  
**RAG wins everywhere:** ~25–65s vs ~340s (~5× throughput).

---

### Retrieve — And Make It Hybrid

Dense retrieval collapsed at 400k (buried fact: 0%).  
Keyword/BM25 held **100%**.  
The tutor keeps a keyword path.

---

### Does Local Match the Cloud?

| | Local 32B | DeepSeek V4 Flash | Gemini 3.5 |
|--|--|--|--|
| Cost / turn | ~$0 (own GPU) | ~$0.006 ✓ | ~$0.11 ✓ |
| Chat memory (keep-all) | 33% † | 95% † | 92% † |
| Doc Q&A (RAG) | 100% ✓ | n/a ‡ | n/a ‡ |
| Speed (first token) | ~20–350s | ~1–20s ✓ | ~17–76s ✓ |
| Throughput | ~1,300 turns/day/GPU | elastic | elastic |

Local doesn't match cloud on chat (window-capped), but **kills the per-token bill** and RAG carries docs.

---

### What They Found

| Finding | Detail |
|---------|--------|
| Keep-all 95% vs. summarize 32% | Don't summarize your history away |
| Finding a fact in long context is easy | No rot for this job (checked to ~800k) |
| The cheapest run sends the most tokens | Caching makes the repeated context nearly free |

**Cost at ~10k turns/day:**

| Model | $/month |
|-------|---------|
| Gemini 3.5 | ~$34k |
| DeepSeek V4 Flash | ~$1.9k |
| Local SLM | ~$0/token (throughput-bound, ~3 Macs) |

> Gemini to DeepSeek is the big lever (~18×). Local is about privacy and throughput, not cost at this scale.

---

### The Recommended Stack

**Don't compact by default. Name the constraint first: window, cost, or throughput.**

| Component | Choice | Rationale |
|-----------|--------|-----------|
| **Model** | DeepSeek V4-Flash | Caching makes keep-all the cheapest |
| **Retrieval** | Hybrid: dense + BM25 + rerank | Dense collapses at long context; BM25 holds |
| **Memory** | Keep everything | Compaction is off-by-default fallback (~30k-history knob) |
| **Hard stop** | ~1M window | |

**Result:** ~$0.006 / turn · first reply ~1–2s · 95% recall

---

## Key Takeaways

1. **Context rot is real** but the fix isn't always compaction — caching often wins instead
2. **Summarization can backfire** — it rewrites the prefix, killing cache hits and costing more
3. **KV-cache hit rate** is the single most important production metric (per Manus AI)
4. **Compaction is the exception, not the rule** — only compact when the window is the actual constraint
5. **Hybrid retrieval** (dense + BM25) is essential — dense alone collapses at long context
6. **The shape that won:** durable files (Karpathy-style wiki) + a small live window
7. **At scale:** going local is about privacy and throughput, not cost at moderate scale

---

## Resources

- **Repo:** https://github.com/towardsai/ai-tutor-app
- **Demo:** https://huggingface.co/spaces/towardsai-tutors/context-engineering-experiments
- **Course:** Full Stack AI Engineering — academy.towardsai.net
- **Cache trap thread:** https://x.com/its_ao/status/2070556265906917860
- **RAG paper:** https://arxiv.org/pdf/2605.05242
