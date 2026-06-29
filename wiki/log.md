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
