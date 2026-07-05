---
type: talk
tags: [model-routing, inference-cost, kv-cache, agents, coding-agents, cost-optimisation, not-diamond, model-gateway]
updated: 2026-07-02
---

# Intelligent Model Routing: Frontier Performance Without Frontier Bills

## Metadata

| Field | Value |
|---|---|
| **Speaker** | [Tomás Hernando Kofman](../speakers/tomas-hernando-kofman.md) |
| **Affiliation** | Not Diamond (CEO & Co-Founder) |
| **Day / Time** | Day 3 (Session Day 2) · 2:50–3:10pm |
| **Room** | Leadership 2 |
| **Track** | Sandbox & Platform Engineering |
| **Status** | Confirmed |
| **Slides** | 20 slides total; 13 captured (slides 4, 6–17). Original photos: `raw/slides/model-routing/` |
| **Raw notes** | [raw/model-routing-slides.md](../../raw/model-routing-slides.md) |
| **Blog** | https://www.notdiamond.ai/blog/a-comprehensive-guide-to-model-routing |

**Official description:** It is Summer 2026 and the world is burning for token consumption—figuratively and literally. Accelerating frontier model capabilities increasingly allow agents to operate across long-running, highly parallelized tasks at exponential inference growth. In this talk, I explain how dynamic model routing—intelligently directing agent requests to the best-suited model at the best price—can reduce token costs by up to 90% while maintaining or improving accuracy. I walk through how routing works, when it doesn't, and why the world (and your agent) need routing to scale intelligence to infinity and beyond.

---

## Key Claims

1. **AI inference spend is in crisis.** Microsoft is canceling Claude Code licenses; Uber burned its 2026 AI budget in four months; a CFO described a "half a billion dollar accidental AI bill." Ramp data shows median AI spend per employee rising steeply, with the Top 1% at $7,446/month.

2. **Three compounding reasons spend keeps rising:** (a) frontier models are getting more expensive and more capable, meaning agent sessions run longer; (b) more powerful models incur more cost per session because they generate more tokens; (c) we're still on the early part of the AI adoption curve.

3. **The arbitrage opportunity:** Cheap/OSS models are converging on frontier capability. The Intelligence vs. Cost chart (Artificial Analysis data) shows models like GLM-5.2 approaching GPT-5.5 quality territory at a fraction of the cost. The gap between "what you need" and "what frontier models cost" is exploitable.

4. **Routing ≠ Gateway.** A gateway gives access to models (auth, API normalisation, billing). A router decides *which* model handles *which* request. Both are necessary; they are distinct layers. Uber's model gateway (Day 2 talk) contains a "smart router" — this talk explains what that component actually does.

5. **Naive routing breaks in agentic settings.** Heuristic, semantic, LLM-based, complexity-classifier, cascade, and session-based routing all have failure modes — particularly around multi-turn context and KV cache economics. "Routing is deceptively difficult."

6. **Agent-native routing is a reinforcement learning problem.** The routing policy must select from (model × reasoning effort) action pairs, operating over long horizons with variable task complexity and sparse rewards. The environment includes session state, KV cache state, and feedback signals.

7. **KV-cache-aware routing is essential.** Switching models mid-session invalidates the cached prefix. A cache read is ~10% of the uncached input cost; an injudicious model switch can cost more than staying on the expensive model. The router must track cache TTL (typically 5 min), compaction events, and context changes.

8. **Results:** On Terminal-Bench / Claude Code, Not Diamond achieves ~75% accuracy at ~$1.00/attempt — matching Claude Opus 4.8 xhigh (~$2.20/attempt) at roughly half the cost. 30%+ savings claimed across F500 engineering teams and industry benchmarks.

9. **Coding agent cost surface areas:** Model choice, reasoning effort, KV cache economics, context pressure, pricing changes, and session length all independently drive cost — and all change constantly as the model landscape evolves. This affects every major coding harness (Claude Code, Codex, Windsurf, Cline, Zed, opencode, GitHub Copilot, Gemini, etc.).

10. **Architecture is privacy-preserving:** Not Diamond routes on *derived, schematized metadata* — not raw prompt content. The ND agent runs locally; only metadata leaves the dev environment to the ND Optimization Server in the cloud. ISO 27001 and SOC 2 certified.

---

## Notable Quotes

> "We use Not Diamond to power our intelligent routing feature, giving developers the ability to automatically use the best model on every input across every leading language model."
> — Alex Atallah, CEO and Co-Founder at OpenRouter (customer testimonial shown on slide)

> "With Not Diamond, we can adapt to AI innovations and new models much faster and benefit from performance improvements in AI development."
> — Philipp Herzig, CTO and Chief AI Officer at SAP (customer testimonial shown on slide)

---

## Diagrams

### Routing vs. Gateway flow (slide 10)

```
[Harness]
    │
[Prompt] ──► [Not Diamond Router] ──► [Gateway]
                                          │
                               ┌──────────────────────┐
                               │ Anthropic / OpenAI   │
                               │ Other providers...   │
                               └──────────────────────┘
                                          │
                                   Output ($0.84)
```

The router sits *between* the harness and the gateway; it selects the model, then the gateway handles access.

### Agent-native routing as RL (slide 12)

```
┌─────────────────┐   ┌──────────────────────────────┐   ┌─────────────────────┐
│  Routing policy │──►│  Action space                │──►│  Environment        │
│                 │   │  Model + reasoning 1          │   │  Session state      │
│                 │   │  Model + reasoning 2          │   │  KV cache state     │
│                 │   │  ...                          │   │  Feedback signals   │
│                 │   │  Model + reasoning n          │   │                     │
└────────▲────────┘   └──────────────────────────────┘   └──────────┬──────────┘
         │◄──────────────────── Reward ──────────────────────────────┤
         │◄──────────────────── State ───────────────────────────────┘
```

### Cost curve: default vs. routed (slide 16)

A line chart (TIME vs. TOTAL COST) shows two trajectories:
- **Default coding harness:** accelerating cost curve (each turn compounds as the session lengthens)
- **With routing:** near-linear, much flatter trajectory

Routing achieves flatness through: model and reasoning recommendations per turn; KV cache monitoring and optimisation; task boundary detection and compaction; continuous learning from feedback.

### Architecture (slide 17)

```
Dev local                              Client gateway
┌─────────────────────────────┐        ┌─────────────┐
│ Coding agent harness ◄──► ND agent │◄──►│  Model 1    │
│                  (metadata,   │        │  Model 2    │
│                  local cache) │        └─────────────┘
└─────────────────────────────┘               │
                                         OSS model
Not Diamond cloud                       (ND inference)
┌─────────────────────────────────────────────┐
│  Cost dashboard ◄──► ND Optimization Server │
│  (account, usage, billing)  (selects model) │
└─────────────────────────────────────────────┘
```

Raw prompts stay local. Only derived metadata crosses the boundary to the cloud.

---

## Routing Approach Taxonomy (slide 11)

| Approach | Mechanism | Key limitation |
|---|---|---|
| **Heuristic** | Keyword/length/regex rules | Brittle; breaks on edge cases; doesn't scale |
| **Semantic** | Embed prompt + cosine-match to example phrases | Only as good as example set; struggles with multi-turn context |
| **LLM-based** | LLM classifies prompt into route | Adds a full inference call per request — often self-defeating |
| **Complexity classifier** | Trained model estimates difficulty | Difficulty is coarse; doesn't capture domain specialisation; needs retraining as models evolve |
| **Cascade** | Try cheap model first; escalate if quality insufficient | Requires cheap verification mechanism; adds latency; not viable for latency-sensitive apps |
| **Session-based** | Routing decision persists across turns | (Not Diamond's approach — RL policy with environment state) |

---

## Reactions / Questions

- The RL framing for routing is striking — treating the KV cache as environment state is exactly the right abstraction. Sparse rewards over long agent sessions is a hard RL problem though; curious how they handle exploration vs exploitation in production.
- The Terminal-Bench result (75% accuracy at $1 vs $2.20 for Opus xhigh) is impressive but only uses Anthropic models. The claim "greater savings achievable with OSS models" is plausible but unverified.
- The "half a billion dollar accidental AI bill" is extraordinary if accurate — not sourced on slide, worth tracking down.
- The privacy claim (routing on metadata, not prompts) is important for enterprise adoption. Would be good to understand what "derived, schematized metadata" actually contains — does it include task type, prompt length, tool schemas?
- The slide deck is 20 slides; slides 1–3, 5, and 18–20 were not photographed. Missing the close/CTA and potentially a product demo.
- This talk directly answers the open question from the [Agentic SDLC at Uber](day2-1140-agentic-sdlc-at-uber.md) notes: *"How is the 'smart router' choosing between providers/models?"* — the answer is a learned RL policy that's KV-cache aware, not simple heuristics.

---

## Links

- [Tomás Hernando Kofman](../speakers/tomas-hernando-kofman.md)
- Concepts: [Model Routing](../concepts/model-routing.md) · [KV Cache](../concepts/kv-cache.md) · [Model Gateway](../concepts/model-gateway.md)
- Related talks: [Agentic SDLC at Uber](day2-1140-agentic-sdlc-at-uber.md) (Model Gateway with smart router) · [Context Engineering in 2026](day1-1420-context-engineering-2026.md) (compaction, cache economics)
