# Wiki Log

Append-only record of all operations. Format: `## [YYYY-MM-DD] <operation> | <title>`

---

## [2026-06-29] init | Wiki created for AIE World Fair 2026

## [2026-06-29] ingest | LLM Inference at Scale Workshop (Harshul Jain & Tanmay Sah)
- Created: wiki/talks/llm-inference-at-scale.md
- Created: wiki/speakers/harshul-jain.md
- Created: wiki/speakers/tanmay-sah.md
- Created: wiki/concepts/kv-cache.md
- Created: wiki/concepts/attention-mechanisms.md
- Created: wiki/concepts/inference-engines.md
- Created: wiki/concepts/memory-bandwidth-roofline.md
- Created: wiki/concepts/quantization.md
- Updated: wiki/index.md, wiki/overview.md

## [2026-06-29] ingest | What is an Inference Engine, Anyway? (Charles Frye)
- Source: raw/What is an Inference Engine, Anyway.md
- Created: wiki/talks/what-is-an-inference-engine-anyway.md
- Created: wiki/speakers/charles-frye.md
- Updated: wiki/concepts/inference-engines.md (added engine/server distinction, llm-d, statefulness; now 2 sources)
- Updated: wiki/index.md, wiki/overview.md, wiki/log.md

## [2026-06-29] ingest | Context Engineering in 2026: Compaction, Memory & Cost (Towards AI)
- Source: raw/towards-ai-context-engineering-workshop.md
- Created: wiki/talks/context-engineering-2026.md
- Created: wiki/speakers/louis-francois-bouchard.md
- Created: wiki/speakers/samridhi-vaid.md
- Created: wiki/speakers/omar-solano.md
- Created: wiki/concepts/context-engineering.md
- Updated: wiki/concepts/kv-cache.md (added application-layer prompt caching section; now 2 sources)
- Updated: wiki/index.md, wiki/overview.md, wiki/log.md

## [2026-06-29] ingest | Agents That Own Their Inference: Building Production AI Agents on Dedicated GPUs (Akamai)
- Source: raw/akamai-workshop-agents-own-inference.md
- MCP session: Day 1 — Workshop Day · 9:00am–11:00am · Track 7 (confirmed)
- MCP speaker: Du'an Lightfoot — Senior AI Engineer, Akamai Technologies (confirmed); Omer Aslan co-presented Modules 5–8, not in conference registry
- Created: wiki/talks/agents-own-inference.md
- Created: wiki/speakers/duan-lightfoot.md
- Created: wiki/concepts/speculative-decoding.md (new concept, first source)
- Created: wiki/concepts/mixture-of-experts.md (new concept, first source)
- Created: wiki/concepts/self-hosted-inference.md (new concept, first source)
- Updated: wiki/concepts/kv-cache.md (added per-token formula, concurrency table, prefix-cache measurement; now 3 sources)
- Updated: wiki/concepts/inference-engines.md (added vLLM tuning knobs, saturation signals, llama.cpp, cross-engine tension; now 3 sources)
- Updated: wiki/concepts/memory-bandwidth-roofline.md (added RTX 4000 Ada numbers, MoE roofline comparison; now 2 sources)
- Updated: wiki/concepts/quantization.md (added FP8 workflow, format ladder, quality-check discipline, FP8 vs INT4 debate; now 2 sources)
- Updated: wiki/index.md, wiki/overview.md (themes 9–11, updated debates and open questions)

## [2026-06-30] ingest | Agentic SDLC at Uber: Building Blocks for Uber's Software Factory (partial)
- Source: Chang's photos of slides 2–9 of a 16-slide deck (no file dropped in raw/; transcribed to raw/agentic-sdlc-at-uber.md)
- Coverage note: 5 of 6 "building blocks" captured (Model Gateway, MCP Gateway, DevPods, Agent Skills, Context Graph); AI Assistant block and the full SDLC stage walkthrough (Idea/Build/Validation/Maintenance) not yet captured — page marked partial, to be updated if more is shared
- MCP session: Day 2 — Session Day 1 · 11:40am–12:00pm · Leadership 1 · Track: AI-Native Enterprises (confirmed)
- MCP speakers: Uday Kiran Medisetty (Distinguished Engineer, Uber), Adam Huda (Sr Engineering Leader for AI Dev Tools, Uber)
- Created: wiki/talks/agentic-sdlc-at-uber.md
- Created: wiki/speakers/uday-kiran-medisetty.md
- Created: wiki/speakers/adam-huda.md
- Created: wiki/concepts/model-gateway.md (new concept, first source)
- Created: wiki/concepts/mcp-gateway.md (new concept, first source)
- Created: wiki/concepts/devpods.md (new concept, first source)
- Created: wiki/concepts/agent-skills.md (new concept, first source)
- Created: wiki/concepts/context-graph.md (new concept, first source)
- Updated: wiki/index.md, wiki/overview.md (new theme 12–13: agentic engineering as platform infrastructure; tool-call fan-out as a two-layer tax)

## [2026-06-30] lint | Quote-fabrication audit across all talk pages
- Trigger: caught a paraphrased/uncertain slide annotation presented as a verbatim quote in wiki/talks/agentic-sdlc-at-uber.md; audited the other 4 talk pages against their raw notes for the same failure mode
- wiki/talks/llm-inference-at-scale.md — clean, no issues
- wiki/talks/what-is-an-inference-engine-anyway.md — clean, no issues
- wiki/talks/context-engineering-2026.md — removed 2 "Notable Quotes" entries ("Summarization (potentially) is a trap..." and "Don't compact by default...") that were bolded analytical text/headings in the raw note, not actual spoken quotes; the underlying claims are already correctly captured (unquoted) in Key Claims, so no information lost
- wiki/talks/agents-own-inference.md — removed 1 "Notable Quotes" entry ("The engine is a real layer...") attributed to Module 1; this exact phrasing does not appear anywhere in the raw note — it was a reconstructed paraphrase presented as verbatim. The underlying claim is already correctly captured (unquoted) in Key Claims
- No fabricated numbers, stats, or unsupported claims found in any of the four pages
- Swept wiki/concepts/*.md and wiki/overview.md for the same pattern: 2 blockquotes found (kv-cache.md's Manus AI quote, context-graph.md's demo question), both verified verbatim against their raw notes — clean
- Added a Convention to CLAUDE.md: only blockquote text the raw note itself marks as a direct/spoken quote; never promote bolded/heading-style text to quote formatting; flag illegible source material explicitly instead of smoothing it into a clean quote
- Added a Lint checklist item to CLAUDE.md for the same check going forward

## [2026-07-01] update | Sort talks chronologically, rename talk files for filesystem-sortable order
- Reordered the Talks table in wiki/index.md ascending by Day/Time (was insertion order)
- Renamed all 5 talk pages to `day<N>-<HHMM>-<slug>.md` so a plain directory listing sorts chronologically:
  - wiki/talks/agents-own-inference.md → wiki/talks/day1-0900-agents-own-inference.md
  - wiki/talks/what-is-an-inference-engine-anyway.md → wiki/talks/day1-1105-what-is-an-inference-engine-anyway.md
  - wiki/talks/llm-inference-at-scale.md → wiki/talks/day1-1210-llm-inference-at-scale.md
  - wiki/talks/context-engineering-2026.md → wiki/talks/day1-1420-context-engineering-2026.md
  - wiki/talks/agentic-sdlc-at-uber.md → wiki/talks/day2-1140-agentic-sdlc-at-uber.md
- Updated all inbound links in wiki/index.md, wiki/overview.md, wiki/speakers/*.md, wiki/concepts/*.md to the new filenames
- Documented the naming convention in CLAUDE.md's Wiki page types section for future ingests

## [2026-07-05] update | Archived Uber talk slide photos; corrected Context Graph cost figure; added Validation & Maintenance slides
- Chang re-uploaded 9 slide photos as files (previously only shared inline, which couldn't be saved — see prior conversation). Archived both the untouched originals and PNG conversions to `raw/slides/agentic-sdlc-at-uber/`, following the convention established by `raw/slides/model-routing/` and `raw/slides/productionizing-llm-gateways/`
- 7 of the 9 photos re-capture slides already transcribed (agenda, Model Gateway x2, MCP Gateway, DevPods, Agent Skills, Context Graph) at higher resolution; 2 are newly-captured slides (14 — Validation, 15 — Maintenance)
- **Correction:** the higher-resolution Context Graph photo (slide 9) revealed the demo cost was **$0.38**, not $0.36 as transcribed from the original lower-resolution photo. Fixed in raw/agentic-sdlc-at-uber.md, wiki/talks/day2-1140-agentic-sdlc-at-uber.md, and wiki/concepts/context-graph.md
- The previously-illegible second line of the Context Graph slide's on-slide annotation is now legible: "Without it: 93 Bash calls exploring..." — noted as-is alongside the (slightly different, 94) figure in the same slide's comparison table, not reconciled
- Added Validation and Maintenance content to raw/agentic-sdlc-at-uber.md and wiki/talks/day2-1140-agentic-sdlc-at-uber.md (2 of the 4 "What You Can Build" SDLC stages were previously uncaptured); noted their demo mockups were labeled "illustrative" on the slide
- Embedded slide images inline in the talk page for all captured architecture/diagram slides
- Updated "Notes coverage" in the talk page's metadata table to reflect current gaps (still missing: title slide, slide 6, AI Assistant, Idea, Build, closing)
- Still not stored: original photos from batch 1 (shared inline in chat, not as file uploads) — these were superseded by batch 2's higher-resolution re-capture of the same slides, so no content was lost, but if Chang has the original batch-1 files they could still be archived for completeness

## [2026-07-02] ingest | Intelligent Model Routing: Frontier Performance Without Frontier Bills (Tomás Hernando Kofman)
- Source: Chang's photos of 13 of 20 slides (slides 4, 6–17 captured; slides 1–3, 5, 18–20 not photographed); transcribed to raw/model-routing-slides.md; slide photos archived to raw/slides/model-routing/
- Supplementary: https://www.notdiamond.ai/blog/a-comprehensive-guide-to-model-routing (April 23, 2026) — fetched to fill gaps in uncaptured slides and enrich concept definitions
- MCP session: Day 3 — Session Day 2 · 2:50–3:10pm · Leadership 2 · Track: Sandbox & Platform Engineering (confirmed)
- MCP speaker: Tomás Hernando Kofman — CEO & Co-Founder, Not Diamond
- Created: wiki/talks/day3-1450-intelligent-model-routing.md
- Created: wiki/speakers/tomas-hernando-kofman.md
- Created: wiki/concepts/model-routing.md (new concept, first source)
- Updated: wiki/concepts/kv-cache.md (added KV-cache-aware routing section, model-switch invalidation cost; now 4 sources)
- Updated: wiki/concepts/model-gateway.md (added gateway vs. router clarification, resolved open question on Uber's smart router; now 2 sources)
- Updated: wiki/index.md, wiki/overview.md (new theme 15: model routing as missing infrastructure layer; new debates and open questions)

## [2026-07-01] ingest | Scaling Code Quality: Building uReview, Uber's Multi-Agent Code Review Engine (Will Bond, Ameya Ketkar)
- Source: Chang's photos of 4 slides (Requirements, Inner/Outer Loop, The Review Stack, uReview Architecture); no title slide captured; transcribed to raw/ureview-scaling-code-quality-uber.md
- MCP session: Day 2 — Session Day 1 · 12:05pm–12:25pm · Leadership 1 · Track: AI-Native Enterprises (confirmed)
- MCP speakers: Will Bond (Staff Software Engineer, Uber), Ameya Ketkar (Software Engineer, Uber Programming Systems Group)
- Note: official session description covers a Generator-Verifier trust layer, confidence scoring/suppression, and semantic deduplication that the 4 captured slides don't show directly — included in the talk page and concept page but clearly marked as MCP-sourced, not slide-confirmed
- Created: wiki/talks/day2-1205-ureview-uber.md
- Created: wiki/speakers/will-bond.md
- Created: wiki/speakers/ameya-ketkar.md
- Created: wiki/concepts/ai-code-review.md (new concept, first source)
- Updated: wiki/concepts/agent-skills.md (added open question / possible link to uReview's "Custom Agents," unconfirmed)
- Updated: wiki/index.md, wiki/overview.md (new theme 14: shared-infrastructure pattern applied to code review)

## [2026-07-05] ingest | Rise of the Software Factory (Tereza Tížková)
- Source: Speaker's blog write-up at terezatizkova.com/blog/rise-of-the-software-factory (full write-up with slides as images; not a verbatim transcript — speaker notes this explicitly); saved to raw/rise-of-the-software-factory.md
- MCP session: Day 2 — Session Day 1 · 11:10am–11:30am · Main Stage · Track: Software Factories (confirmed)
- MCP speaker: Tereza Tížková — Growth, Factory (twitter: @tereza_tizkova; linkedin; website terezatizkova.com)
- Created: wiki/talks/day2-1110-rise-of-the-software-factory.md
- Created: wiki/speakers/tereza-tizkova.md
- Created: wiki/concepts/software-factory.md (new concept, 2 sources: this talk + Uber SDLC talk)
- Created: wiki/concepts/agent-readiness.md (new concept, 2 sources: Factory scoring rubric + Uber's six-building-block infrastructure approach)
- Updated: wiki/concepts/model-routing.md (added Factory Router as second production implementation; factory-software tag; now 2 sources)
- Updated: wiki/concepts/kv-cache.md (added agent-platform prefix caching section — 10× second-turn cost reduction, pass-through economics; now 5 sources)
- Updated: wiki/index.md (added talk row, speaker row, 2 new concept rows)
- Updated: wiki/overview.md (new theme 17: software factory as unifying conference frame; new open questions; updated sessions ingested count to 9)

## [2026-07-05] update | Store original uReview slide photos, link diagrams from wiki
- Saved 4 original slide photos (uploaded as HEIC) to raw/slides/ureview-scaling-code-quality-uber/, converted to JPG: slide-01-requirements.jpg, slide-02-inner-outer-loop.jpg, slide-03-review-stack.jpg, slide-04-architecture.jpg — following the raw/slides/<slug>/ convention already used for model-routing and productionizing-llm-gateways
- Updated: raw/ureview-scaling-code-quality-uber.md (embedded each photo next to its transcription)
- Updated: wiki/talks/day2-1205-ureview-uber.md (embedded The Review Stack and uReview Architecture diagrams — the two slides with real diagram content — plus a link to the photo folder)
- Updated: wiki/concepts/ai-code-review.md (same two diagrams embedded)

## [2026-07-05] ingest | Letting the Interns Loose — How We Accelerated AI Adoption (Shashank Goyal, OpenRouter)
- Source: raw/letting-the-interns-loose.md (transcribed from ~14 of ~33 slides, HEIC photos converted to JPG for OCR)
- MCP session: Day 3 — Session Day 2 · 11:10–11:30am · Track 1 · Track: Sandbox & Platform Engineering (confirmed)
- MCP speaker: Shashank Goyal — Head of Provider Ecosystem & Founding Engineer, OpenRouter
- Created: wiki/talks/day3-1110-letting-the-interns-loose.md
- Created: wiki/speakers/shashank-goyal.md
- Updated: wiki/concepts/agent-skills.md (added OpenRouter's skill marketplace as 2nd implementation, staged rollout lesson; now 2 sources)
- Updated: wiki/concepts/guardrails.md (added OS-level secrets isolation pattern — systemd InaccessiblePaths + privileged sibling; now 3 sources)
- Updated: wiki/index.md (new talk row, new speaker row, count → 10 talks)
- Updated: wiki/overview.md (new theme 18: intern model / constraints as reliability path; 4 new open questions; sessions count → 10)
- No new concept pages: "intern model," "config-as-code for agents," "one VM per intern," and "self-evolving agents" are 1st-source-only from this talk — flagged in open questions for concept creation when a 2nd source appears

## [2026-07-05] ingest | Don't Build Agents, Build Environments (Adam Azzam)
- Source: raw/dont-build-agents-build-environments.md (9 slides, full deck; transcribed from Chang's HEIC photos)
- MCP session: Day 3 — Session Day 2 · 10:45–11:05am · Track 1 · Track: Sandbox & Platform Engineering (confirmed)
- MCP speaker: Adam Azzam — Member of Product Staff, Modal (formerly VP of Product at Prefect; maintainer of Prefect and FastMCP; PhD in mathematics)
- Created: wiki/talks/day3-1045-dont-build-agents-build-environments.md
- Created: wiki/speakers/adam-azzam.md
- Created: wiki/concepts/agent-environment-architecture.md (new concept; 3 sources: Azzam/Ramp, Uber DevPods, OpenRouter VM-per-intern)
- Updated: wiki/concepts/devpods.md (added Ramp/Modal comparison section; now 2 sources)
- Updated: wiki/index.md (new talk row, speaker row, concept row; count → 11)
- Updated: wiki/overview.md (new theme 19: agent environment architecture as emerging consensus; sessions count → 11)

## [2026-07-02] ingest | Productionizing LLM Gateways: Architecture, Tradeoffs, and Hard Lessons from the Trenches
- Source: raw/productionizing-llm-gateways.md (14 slides, full deck)
- Created: wiki/talks/day2-1425-productionizing-llm-gateways.md
- Created: wiki/speakers/kanish-manuja.md
- Updated: wiki/concepts/model-gateway.md (added Twilio's four-axis framing, ejecting-breaker pattern, per-route latency tracking, decentralised-gateway architecture)
- Created: wiki/concepts/guardrails.md (new concept, 2 sources: Uber AI Guard + Twilio talk)
- Updated: wiki/index.md (new talk, speaker, concept)
- Updated: wiki/overview.md (new theme 16: four-axis gateway tradeoffs; new debate: central vs. decentralised gateway; 3 new open questions)

## [2026-07-05] ingest | Your Agent Needs a Sandbox, Not a Desert (Samuel Colvin / Pydantic)
- Updated: raw/agent-needs-sandbox-not-desert.md (added metadata header)
- Created: wiki/talks/day3-1205-agent-needs-sandbox-not-desert.md
- Created: wiki/speakers/samuel-colvin.md
- Updated: wiki/concepts/agent-environment-architecture.md (added Colvin's minimal-sandbox counter-argument and Pydantic Monty; 4th source)
- Updated: wiki/index.md (new talk, speaker)
- Updated: wiki/overview.md (new theme 20: minimal runtime argument; new debate item)

## [2026-07-05] ingest | 1,000 Agent Tasks in a Sandbox: What Breaks When LLMs Write and Run Code (Kevin Orellana, Amazon AgentCore)
- Source: raw/1000-agent-tasks-sandbox.md (10 slides captured from HEIC photos; framing, methodology, harness, failure taxonomy, microVM anatomy, controls table, silent-failure naming, local↔remote portability, capability evidence, closing — detailed per-failure-mode breakdown slides not captured)
- MCP session: Day 3 — Session Day 2 · 2:25–2:45pm · Track 1 · Track: Sandbox & Platform Engineering (confirmed)
- MCP speaker: Kevin Orellana — Software Engineer, Amazon Web Services (twitter: @KevssOrellana; linkedin: kevinorellana)
- Created: wiki/talks/day3-1425-1000-agent-tasks-sandbox.md
- Created: wiki/speakers/kevin-orellana.md
- Created: wiki/concepts/silent-semantic-failure.md (new concept; 1st source — Orellana synthesizing Lamport 1982, CMU EMNLP 2024, Mehta arXiv 2026, Tian Pan 2026)
- Created: wiki/concepts/agent-driven-testing.md (new concept, 1st source)
- Updated: wiki/concepts/agent-environment-architecture.md (added microVM anatomy section and primitive-to-control table; 5th source)
- Updated: wiki/index.md (new talk, speaker, 2 new concept rows; count → 13)
- Updated: wiki/overview.md (new theme 21: environment controls as capability constraints; sessions count → 13)

## [2026-07-05] ingest | The Future of Engineering — Addy Osmani (Closing Keynote)
- Created: wiki/talks/day3-1630-future-of-engineering.md (32 slides; 1 slide missing from upload batch)
- Created: wiki/speakers/addy-osmani.md
- Created: wiki/concepts/harness-engineering.md (new concept; 1st source)
- Created: wiki/concepts/loop-engineering.md (new concept; 1st source)
- Created: wiki/concepts/human-verdict.md (new concept; 1st source)
- Created: wiki/concepts/taste-as-alpha.md (new concept; 1st source)
- Created: wiki/concepts/cognitive-debt.md (new concept; 1st source — covers cognitive debt, cognitive surrender, orchestration tax)
- Updated: wiki/concepts/software-factory.md (added Osmani framing; now 3 sources)
- Updated: wiki/index.md (new talk, speaker, 5 new concept rows; count → 14)
- Updated: wiki/overview.md (new theme 22: closing keynote synthesis; sessions count → 14)

## [2026-07-05] ingest | Operating Distributed Inference Systems at Scale (Nishant Gupta & Naman Ahuja)
- Created: wiki/talks/day4-1045-operating-distributed-inference-systems-at-scale.md
- Created: wiki/speakers/nishant-gupta.md (new speaker; Meta Staff SWE & Tech Lead)
- Created: wiki/speakers/naman-ahuja.md (new speaker; Meta Senior SWE)
- Created: wiki/concepts/inference-control-plane.md (new concept; 2 sources: Gupta/Ahuja + Azzam)
- Created: wiki/concepts/gpu-scheduling.md (new concept; 2 sources: Gupta/Ahuja + prior inference workshops)
- Updated: wiki/concepts/kv-cache.md (added source; KV hit rate as scheduler input + observability signal)
- Updated: wiki/concepts/inference-engines.md (added source; serving runtime layer framing)
- Updated: wiki/index.md (new talk, 2 new speakers, 2 new concept rows; count → 16)
- Updated: wiki/overview.md (new theme 23: inference as infrastructure orchestration problem; sessions → 15)

## [2026-07-05] ingest | Routing LLM Inference in Production: From Engine Signals to Policy (Qianru Lao & Lu Zhang, OpenAI)
- Source: raw/routing-llm-inference-in-production.md (10 slides, full deck; transcribed from Chang's HEIC photos); slide photos archived to raw/slides/routing-llm-inference-in-production/
- MCP session: Day 4 — Session Day 3 · 11:10–11:30am · Track 9 · Track: Inference (confirmed)
- MCP speakers: Qianru Lao (MTS, OpenAI Inference), Lu Zhang (MTS, OpenAI Inference)
- Talk page already created in prior session: wiki/talks/day4-1110-routing-llm-inference-in-production.md
- Speaker pages already created in prior session: wiki/speakers/qianru-lao.md, wiki/speakers/lu-zhang.md
- Updated: wiki/concepts/inference-control-plane.md (added OpenAI ILB as 3rd source; Optimizer formulation; engine vs. model routing distinction; new open questions; now 3 sources)
- Updated: wiki/concepts/kv-cache.md (added KV cache match as engine-routing signal; 3-tier KV cache signal stack; now 6 sources)
- Updated: wiki/index.md (new talk row, 2 new speaker rows; count → 17)
- Updated: wiki/overview.md (new theme 25: engine routing layer; sessions count → 17)
- No new concept pages: all concepts from this talk (ILB, optimizer, control/data plane, protection mechanisms) are captured under existing pages

## [2026-07-05] ingest | Always-on agents run production without the on-call tax (Justin Smith, Resolve AI)
- Source: raw/always-on-agents-production-on-call-tax.md (10 slides, full deck; transcribed from Chang's HEIC photos + MCP metadata); slide photos archived to raw/slides/always-on-agents-production-on-call-tax/
- MCP session: Day 4 — Session Day 3 · 2:25–2:45pm · Track 8 · Track: Agentic Engineering (confirmed)
- MCP speaker: Justin Smith — Founding Product Engineer, Resolve AI
- Created: wiki/talks/day4-1425-always-on-agents-production-on-call-tax.md
- Created: wiki/speakers/justin-smith.md
- Updated: wiki/concepts/human-verdict.md (added Resolve AI as 2nd source — "you open Resolve AI to verify findings" as the operational instantiation of the human-verdict pattern; now 2 sources)
- Updated: wiki/index.md (new talk row, speaker row, human-verdict summary note; count → 16)
- Updated: wiki/overview.md (new theme 24: background agents and the invisible toil problem; new open questions; sessions count → 16)
- No new concept pages: "Production Context," "Background Agents," and "Invisible Toil" are 1st-source-only from this talk — flagged in talk page and overview for concept creation when a 2nd source appears

## [2026-07-18] update | Softened Uber/Twilio gateway framing in overview.md — per Chang's review, Uber notes are silent on resilience ownership, so Manuja's "centralise governance, decentralise traffic" is a complementary caution, not a genuine disagreement
