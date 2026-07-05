# Raw Notes: "Intelligent Model Routing: Frontier Performance Without Frontier Bills"

**Source:** Slide photos from the talk (13 of 20 slides captured: slides 4, 6–17).  
**Presenter:** Tomas H. K. (X: @tomas_hk, Notion: @tomashk) — notdiamond.ai  
**Slide photos:** `raw/slides/model-routing/IMG_8804.HEIC` through `IMG_8819.HEIC`  
**Supplementary blog:** https://www.notdiamond.ai/blog/a-comprehensive-guide-to-model-routing (April 23, 2026)

> Note: Slides 1–3, 5, and 18–20 were not photographed. Content below reconstructed from slides 4, 6–17 in deck order.

---

## Slide 4/20 — Customer testimonials

Two customer quotes shown with headshots:

> "We use Not Diamond to power our intelligent routing feature, giving developers the ability to automatically use the best model on every input across every leading language model."
> — Alex Atallah, CEO and Co-founder at OpenRouter

> "With Not Diamond, we can adapt to AI innovations and new models much faster and benefit from performance improvements in AI development."
> — Philipp Herzig, CTO and Chief AI Officer at SAP

---

## Slide 6/20 — Problem: AI inference spend is exploding

**Section label: PROBLEM**

Title: *AI inference spend is exploding.*

Slide is a collage of news headlines and data snapshots illustrating out-of-control AI spend:

- Headline: "Microsoft starts canceling Claude Code licenses"
- Headline: "Uber Burns Its 2026 AI Budget In Four Months On Claude Code" (Forbes/MSV)
- Headline: "Uber's COO says it's getting harder to justify the money spent on AI tokenmaxxing"
- Finding from Ramp AI Index: "AI spend is rising despite cost pressures. Everyone is spending more on AI." (chart showing AI spend per employee rising sharply Jan 2023–May 2026; Top 1% at ~$354, Top 10% ~$40, Median ~$8)
- OpenAI API dashboard: Today $19,985.84 / 30d spend $1,305,088.81
- Tweet from Jyoti Mann: "SCOOP: Meta plans to clamp down on skyrocketing AI costs inside the company by imposing limits on employees' token usage, the company told staff in a memo on Tuesday, just weeks after it pushed them to adopt AI tools in their work."
- Pull quote from article: "My latest, which includes an account from a CFO fretting over a half a *billion* dollar accidental AI bill"
- Ramp AI Index chart (top-right): Monthly AI spend per employee, showing steep rise with Sep 25 top-1% at $7,446.80

---

## Slide 7/20 — Three reasons spend keeps rising

Title: *(No headline text; continuation of PROBLEM framing)*

Three numbered statements with abstract icons:

1. **Models are becoming more expensive.** *(Icon: small circle → larger circle, i.e., models growing in size/cost)*
2. **More powerful models can run longer.** *(Icon: two overlapping circles of different sizes, suggesting extended compute)*
3. **We're still early in the AI adoption curve.** *(Icon: upward-curving arrow)*

---

## Slide 8/20 — Cheapest models are catching up to frontier

Title: *At the same time, the cheapest models are becoming more powerful, and OSS options are increasingly competitive with the frontier.*

**Diagram: Intelligence vs. Cost per Intelligence Index Task** (source: Artificial Analysis Intelligence Index; weighted average cost USD per task; log scale on x-axis)

X-axis: Cost per Task (USD, Log Scale), from ~$0.03 to ~$3  
Y-axis: Artificial Analysis Intelligence Index, from ~20 to ~65

Model positions (approximate):
- **Top-right (expensive + capable):** Claude Fable 5 (with fallback), Claude Opus 4.8 (max), GPT-5.5 (high), Claude Opus 4.8 (max), Gemini 3.5 Flash
- **Mid-right:** Claude Sonnet 4.6 (max), Qwen3.7 Max, Gemini 3.1 Pro Preview, Kimi K2.6
- **Mid-centre:** GLM-5.2 *(annotated with arrow showing it moving rightward into higher capability at moderate cost)*, MiniMax-M3, Grok 4.3 (high), GLM-5.1, Nemotron 3 Ultra, GPT-5.4 mini (xhigh)
- **Left-cheap zone:** DeepSeek V4 Pro (Max), MMo-V2.5-Pro, DeepSeek V4 Flash (Max)
- **Bottom-left:** gpt-oss-120b (high), Claude 4.5 Haiku, Qwen3.5 3978 A17B, Mistral Medium 3.5

The "most attractive quadrant" is highlighted (high intelligence, low cost — upper-left). Most current models cluster in the centre/right.

---

## Slide 9/20 — The arbitrage opportunity

**Section label: OPPORTUNITY**

Title: *So how do we exploit this model arbitrage opportunity as model providers compete against each other on cost and performance?*

**Diagram: Concentric circles**
- Outer circle: **EXPENSIVE FRONTIER MODELS**
- Inner circle: **CHEAP / OSS MODELS**
- Arrow pointing right from the inner circle, labelled **CAPABILITY**

Interpretation: The inner (cheap/OSS) circle is expanding its capability to overlap with or approach the outer (frontier) circle. The gap is the arbitrage opportunity — you don't always need the expensive outer ring.

---

## Slide 10/20 — What is intelligent model routing?

**Section label: OVERVIEW**

Title: *Intelligent model routing automatically selects the best model for each request to maximize accuracy and minimize cost.*

**Diagram: Request flow**

```
[Harness]
    |
[Prompt] ──► [Not Diamond Router "Model E"] ──(dashed)──► [Gateway]
                                                               |
                                              ┌────────────────────────┐
                                              │ * (Anthropic)          │
                                              │ ⊕ (OpenAI)             │
                                              │ K (Kore/other)         │
                                              │ 🐦 (?)                 │
                                              │ Z (Zed)                │
                                              └────────────────────────┘
                                                               |
                                                     Output ($0.84)
```

The router sits between the harness and the gateway. It selects which model (shown here as "Model E") the prompt is forwarded to. The gateway itself just provides access to multiple providers.

Bottom note (underlined): **Routing is *different* from a gateway, which governs access to models.**

---

## Slide 11/20 — Routing approaches (taxonomy)

**Section label: SOLUTIONS**

Title: *Routing is deceptively difficult. Naive solutions might work for simple workflows but fall apart in agentic settings.*

Six approaches listed in a 2×3 grid with abstract icons:

| Approach | Icon description |
|---|---|
| **Heuristic routing** | Circle / triangle (binary choice, simple rule) |
| **Complexity classifiers** | Bar chart growing (difficulty estimation) |
| **Semantic routing** | Branching Y-arrow (semantic split then route) |
| **Cascade routing** | Three dots growing in size (escalation chain) |
| **LLM-based routing** | Asterisk/spark (LLM-as-classifier) |
| **Session-based routing** | Dot with rightward arrow (persistent context across turns) |

---

## Slide 12/20 — Agent-native routing as RL

**Section label: SOLUTIONS**

Title: *Agent-native routing must deliver **model and reasoning effort recommendations** in a **KV-cache aware** manner over **long horizons** with **variable tasks and complexity** and **sparse rewards**.*

**Diagram: Reinforcement learning loop**

```
┌─────────────────┐     ┌──────────────────────────────┐     ┌───────────────────────┐
│                 │     │        Action space           │     │      Environment      │
│  Routing policy │────►│  Model + reasoning 1          │────►│  Session state        │
│                 │     │  Model + reasoning 2          │     │  KV cache state       │
│                 │     │  ...                          │     │  Feedback signals     │
│                 │     │  Model + reasoning n          │     │                       │
└────────▲────────┘     └──────────────────────────────┘     └──────────┬────────────┘
         │                                                               │
         │◄──────────────────── Reward ──────────────────────────────────┤
         │◄──────────────────── State ───────────────────────────────────┘
```

The routing policy picks from an action space of (model × reasoning effort) combinations. The environment tracks session state, KV cache state, and feedback signals. Reward and state feed back into the policy — framing routing as an RL problem rather than a stateless per-request classification task.

---

## Slide 13/20 — Results: Terminal-Bench / Claude Code

**Section label: RESULTS**

Title: *When done right, model routing works. Across industry benchmarks and F500 engineering teams we achieve frontier quality at a 30%+ lower inference cost.*

**Benchmark: Terminal-Bench / Claude Code**  
*(described as "A widely used benchmark for evaluating coding agents on realistic terminal tasks")*

**Chart: Accuracy vs. Cost per Attempt ($)**  
X-axis: 0 to 2.5 (Cost per Attempt, $)  
Y-axis: 0 to 100 (Accuracy)

Data points (single-model baselines, marked with +):
- Claude Haiku 4.5 Low — ~$0.5, ~30% accuracy
- Claude Sonnet 4.6 High — ~$1.5, ~50% accuracy
- Claude Opus 4.8 high — ~$2.0, ~60% accuracy
- Claude Opus 4.8 xhigh — ~$2.2, ~75% accuracy

**Not Diamond (marked with filled dot):** ~$1.0, ~75% accuracy  
→ Matches Opus 4.8 xhigh accuracy at roughly half the cost (~$1 vs ~$2.2)

Footnote: *Results based on Anthropic models only. Greater savings achievable with OSS models in the routing pool.*

---

## Slide 14/20 — Why coding agent routing is hard (abstract)

**Section label: WHY THIS IS HARD**

Title: *Coding agent costs grow across many surface areas, which themselves change every week as the AI landscape evolves.*

**Diagram: Five cost surface areas around a central node**

Central node (dark): **Coding-agent session**

Surrounding boxes (light):
- **Model choice** — Overpowered defaults / Wrong small models
- **Reasoning effort** — Unintuitive to configure / Opaque benchmark signal
- **KV cache economics** — Cache misses spike costs / Bad switches erase savings
- **Context pressure** — Noisy context and mistimed compaction raises cost
- **Pricing changes** — Tracking various token prices across models
- **Session length** — Blind spots in session length make tradeoffs difficult

---

## Slide 15/20 — Why coding agent routing is hard (with tool logos)

**Section label: WHY THIS IS HARD**

Same diagram as slide 14, but now the abstract diagram is overlaid with logos of real coding agent tools surrounding the central node, showing which tools this problem applies to:

- **opencode**
- **GitHub Copilot**
- **Codex** (OpenAI)
- **Windsurf**
- **Cline**
- **Zed** (Z icon)
- **Claude Code** (K icon + cube)
- **Gemini** / Spark icon
- **OpenAI** (GPT icon)
- **Mistral** (bird icon)

The point: every major coding agent harness is affected by these cost surface areas, and the landscape changes constantly.

---

## Slide 16/20 — Routing flattens the cost curve

**Section label: SOLUTION**

Title: *Routing keeps long-running coding-agent sessions on a flatter cost curve by applying the right cost-reduction strategy at each turn.*

**Chart: Total cost vs. time for a coding session**  
X-axis: TIME  
Y-axis: TOTAL COST

Two lines:
- **Default coding harness** (open circle markers, steeper upward curve) — cost accelerates over time
- **With routing** (filled dot markers, much flatter trajectory) — cost grows slowly and nearly linearly

Legend for "Default coding harness" lists what routing adds on top:
- Model and reasoning recommendations
- KV Cache monitoring and optimization
- Task boundary detection and compaction
- Continuous learning from feedback

---

## Slide 17/20 — Architecture

**Section label: ARCHITECTURE**

Title: *Our approach is highly compute- and data-efficient, allowing us to provide routing in a privacy-preserving manner by basing routing decisions on derived, schematized metadata.*

**Architecture diagram — two deployment zones:**

**Dev local zone:**
```
[Coding agent harness] ◄──► [ND agent: Metadata extraction, manages local cache]
```

**Client gateway zone (right side):**
```
[Model 1]
[Model 2]
```

ND agent (dev local) connects to Client gateway.

**Not Diamond cloud zone (bottom):**
```
[Cost dashboard: Account, usage, billing] ◄──► [ND Optimization Server: Selects optimal model]
                                                             │
                                                      [OSS model]
                                                    (ND inference)
```

Key privacy claim: routing decisions are based on **derived, schematized metadata** — not raw prompt content — meaning prompts stay local.

Compliance badges shown: **ISO 27001**, **AICPA SOC 2**

---

## Slides not captured

Slides 1–3, 5, and 18–20 were not photographed. Based on slide numbering and section labels seen:
- Slides 1–3: likely title, agenda, and intro/company context
- Slide 5: possibly between "customer testimonials" (4) and "spend exploding" (6) — may be a problem statement slide
- Slides 18–20: likely product demo, call to action, or Q&A

---

## Blog supplement: Key definitions and concepts

From the companion blog post (April 23, 2026):

**Gateway vs. Router:** "A gateway gives you access to models. A router determines which one to use." Gateways normalise APIs, handle auth, provide billing consolidation. Routers make per-request model selection decisions. Both are needed; they're not competing.

**Routing dimensions:** (1) which models are in the pool, (2) what signal the router uses (heuristics, semantic, learned), (3) what objective is being optimised (cost, quality, latency).

**Cache-aware routing is critical for agents:** Cache reads cost ~10% of uncached input. Switching models mid-session breaks the cache, potentially costing more than staying on the expensive model. A good router tracks cache TTL (commonly 5 min), compaction events, and media attachments that break cache.

**Routing levels in agentic stacks:** session-level, sub-agent level, task level, step level — each with different model fit requirements.

**Savings range:** 20–95% on inference costs depending on workload. For coding agents, a single session can consume >1M tokens/minute.

**Predictive routing** (Not Diamond's implied approach): learns model strengths from benchmarks and production traffic; picks the model predicted to best satisfy the objective for the specific prompt. More powerful than heuristic or complexity-based approaches; requires good training signal.

**Cascade routing caveat:** requires a verification mechanism cheaper than the cheap model itself; adds latency (sequential calls); not suited to latency-sensitive apps.

**LLM-based routing caveat:** adds a full LLM inference call to every request — often self-defeating unless the classifier is finetuned for scale.
