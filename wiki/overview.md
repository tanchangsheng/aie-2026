---
type: overview
updated: 2026-07-05 (17 sessions)
---

# AIE World Fair 2026 — Conference Overview

_This page is updated after each talk ingest. It reflects the evolving synthesis of themes, debates, and key takeaways from the conference._

---

## Major Themes

### 1. Inference Cost Is Now a Board-Level Problem

The dominant framing from the first session: inference has crossed from engineering concern to financial reality. Training GPT-3 cost $4.6M once; serving it now costs more than that per week. The "token hunger games" — teams competing for compute budgets, C-suites forcing token audits — reflects an industry-wide reckoning with operating costs that scale with every user and every session. Cost per 1M tokens has fallen 1,200× since 2023, but volume grew faster.

### 2. The Memory Equation Is the Foundation

Everything in LLM serving reduces to: `GPU memory = weights + KV cache + overhead`. Weights are fixed; the [KV cache](concepts/kv-cache.md) is the variable that grows with users and context length. All optimization work is, at root, managing this variable.

### 3. Decode Is Memory-Bound — Hardware Alone Won't Save You

The decode bottleneck is memory bandwidth, not compute. For Mistral-7B on an A100 the ceiling is ~138 tok/s set by HBM bandwidth. The [roofline](concepts/memory-bandwidth-roofline.md) sits at ~2 FLOP/byte vs 156 for compute saturation. You raise arithmetic intensity through batching and architecture, not by buying a faster chip.

### 4. Compounding Levers, Not Silver Bullets

~20× throughput improvement from stacking orthogonal techniques: [attention architecture](concepts/attention-mechanisms.md) (GQA/MLA) + [weight quantization](concepts/quantization.md) (INT4) + PagedAttention + continuous batching + prefix caching + speculative decoding. Each multiplies the last.

### 5. No Universal Inference Engine

vLLM vs SGLang vs TensorRT-LLM is workload-matching, not a ranking. Near-parity on standard API serving; SGLang is 5.8× faster on agentic/shared-prefix workloads. See [Inference Engines](concepts/inference-engines.md).

### 6. Context Engineering: What the Model Sees Is as Important as the Model Itself

The context engineering workshop added a second lens to the cost story: inference cost isn't just a GPU/serving problem, it's a context management problem at the application layer. Towards AI's empirical finding — keep-everything + prompt caching is cheaper than aggressive compaction — is the application-layer parallel to the inference layer's "stack orthogonal levers" story. In both cases, the answer is non-obvious and requires measurement.

The key application-layer insight: **KV cache hit rate at the API level is as important as KV cache utilization at the serving level**. Anything that rewrites the stable prompt prefix (like summarization) destroys cache coherence and inverts the expected cost saving. This connects the two Day 1 workshops through [KV Cache](concepts/kv-cache.md) as a shared concept operating at different abstraction layers.

### 7. MLA / DeepSeek as the Attention Architecture to Watch

MLA achieves 56× less KV memory than MHA at near-identical quality — a genuine Pareto win. This is why DeepSeek-V3 is cheap to serve. If MLA becomes standard (as GQA already has), the capacity equation changes dramatically.

---

### 8. The Engine/Server Distinction — and Why It Matters for Scale

Charles Frye drew a clean line that's often blurred: the **inference engine** does compute; the **inference server** handles orchestration (routing, resource management, APIs). vLLM is primarily an engine with a thin server. At production scale a dedicated server layer — like llm-d — handles the concerns the engine shouldn't: prefix-cache-aware routing, tiered KV offloading to CPU/disk, disaggregated prefill/decode, and SLO-aware autoscaling. The takeaway: picking an engine and picking a serving stack are two separate decisions.

The framing also sharpens the revenue/cost argument: "training is a cost centre, inference is a revenue centre" — which is why infrastructure investment has followed this direction. See [Inference Engines](concepts/inference-engines.md).

---

### 9. Owning the Inference Layer Is a Three-Axis Decision

The Akamai workshop frames self-hosting as a decision across cost, data residency, and control — not just cost. The cost crossover (hosted per-token vs. flat GPU hourly) is workable arithmetic; data residency is a compliance line that arithmetic can't override; control (model choice, precision, batching policy) is the tuning leverage that the rest of the workshop demonstrates. See [Self-Hosted Inference](concepts/self-hosted-inference.md).

This connects to the context engineering session's API-level cost comparison (DeepSeek vs. Gemini, 18×) to form a three-level cost stack: (1) prompt engineering and caching, (2) model/provider choice, (3) infrastructure (hosted vs. self-hosted).

### 10. Speculative Decoding Is Workload-Dependent, Not a Free Speedup

The Akamai workshop is the first to demonstrate speculative decoding live and report a negative result. The draft-and-verify mechanism (0.6B drafter + 4B target) can hurt throughput at high concurrency when the extra draft work competes with normal batching. Published results show up to 2.8× speedup in the right conditions; this workshop shows it can also slow things down. **A negative result is a valid production decision.** See [Speculative Decoding](concepts/speculative-decoding.md).

### 11. The Hypothesis-Driven Tuning Loop as a Practice

Modules 5–8 introduce an explicit method: measure → change one thing → redeploy → keep or reject with data. This is the production discipline that was implicit in prior sessions but never named. It applies to quantization, speculative decoding, and serving-policy flags alike. The corollary: changing many things at once hides the cause.

---

### 12. Agentic Engineering Needs Platform Infrastructure, Not Ad Hoc Tool Adoption

The Uber talk introduces a thread distinct from the inference-cost story that dominated Day 1: at organizational scale, agentic software development requires the same kind of platform investment as any other shared infrastructure — gateways, registries, catalogs, graphs — rather than each team independently wiring up LLM calls and tools. Uber's framing: 99% of engineers use AI monthly, 70% of PRs are AI-attributed, and 15% of PRs are done entirely by autonomous agents, but this didn't happen by individual teams adopting tools ad hoc — it required six shared building blocks ([Model Gateway](concepts/model-gateway.md), [MCP Gateway](concepts/mcp-gateway.md), [DevPods](concepts/devpods.md), [Agent Skills](concepts/agent-skills.md), [Context Graph](concepts/context-graph.md), and an AI Assistant not yet covered in these notes).

A consistent sub-pattern across all five captured building blocks: **identify the shared failure mode first** (PII leakage, tool-schema context bloat, slow env setup, duplicated skills, blind agents) **then build the smallest piece of central infrastructure that fixes it for everyone at once.** This mirrors the "measurement-first" discipline from the Day 1 inference workshops, but applied to organizational tooling decisions instead of model-serving tuning.

### 13. Tool-Call Fan-Out Is a Recurring Tax — at Two Different Layers

Uber's [MCP Gateway](concepts/mcp-gateway.md) and [Context Graph](concepts/context-graph.md) attack the same underlying problem from two angles: an agent without structure burns tokens and time rediscovering things it should already know. The MCP Gateway makes each *individual* tool call cheaper (Code Mode costs a fraction of Direct MCP for the same outcome); the Context Graph reduces *how many* calls are needed in the first place (8 vs. 94 tool calls for an identical correct answer, a ~14x time and 7x cost gap). Together they suggest tool-call fan-out is to agentic engineering productivity what KV cache is to inference cost: the single largest lever, attackable at multiple layers simultaneously.

### 14. The Same "Shared Infrastructure First" Pattern, Applied to Code Review

The uReview talk is a second, more specific instance of theme 12's pattern: rather than letting generic AI review tools spam developers with hallucinated comments, Uber built one engine ([AI Code Review](concepts/ai-code-review.md)) that serves both agents and humans through different interfaces with different noise tolerances, and grades its own output before posting. The recurring shape across both Uber talks: identify the failure mode ("developers mute noisy tools" here; "PII leakage," "slow env setup," etc. in the SDLC talk) and build shared infrastructure that fixes it centrally rather than per-team. uReview's inner-loop/outer-loop split is also a concrete second example of Uber designing explicitly for *agents as consumers of infrastructure*, not just humans — the same distinction implicit in DevPods (agent workspaces) and the Context Graph (agent-legible relationships).

---

### 15. Model Routing as the Missing Layer Between Gateway and Application

The Not Diamond talk introduces the third leg of the AI infrastructure stack that was implicit but unnamed in earlier sessions. Day 1 established the **serving layer** (inference engines, KV cache, quantization). Day 2 established the **platform layer** (gateway, DevPods, skills, context graph). Day 3 adds the **routing layer**: a per-request decision engine that optimises across the model pool for cost, quality, and latency simultaneously.

The routing vs. gateway distinction that Kofman hammers is not pedantic — it resolves the ambiguity in Uber's "smart router" (an open question from the Day 2 notes). The gateway controls access; the router controls selection. Uber has both, and the smart router inside their middleware chain is presumably doing some form of the learned routing that Not Diamond specialises in.

The specific contribution here over prior sessions is that **routing for agentic workloads is categorically harder than routing for chat**:
- KV cache economics: switching models mid-session can cost *more* than staying on the expensive model, because it breaks the cached prefix. A cache-unaware router destroys its own paper savings.
- Long-horizon RL: the routing decision isn't stateless (one prompt → one model). It's a policy over a session with variable complexity, accumulating context, and delayed feedback. Kofman explicitly frames this as a reinforcement learning problem.

The result is that "30%+ savings at frontier quality" isn't achievable with heuristics or complexity classifiers alone — it requires a learned policy that treats KV cache state as environment state.

This also sharpens the cost story from Day 1. The Day 1 workshops asked: how do you serve a given model cheaply? (serving-layer answer: quantization, batching, attention architecture). The Day 3 talk asks: do you even need the expensive model? (routing-layer answer: often not, if you have the right signal). These are orthogonal levers that compound.

See: [Model Routing](concepts/model-routing.md) · [Intelligent Model Routing talk](talks/day3-1450-intelligent-model-routing.md)

### 16. The Gateway Is Where Four Forces Fight — and You Can't Win All Four

The Twilio talk (Kanish Manuja) provides the most operationally detailed gateway treatment at the conference and introduces a clean framing: a gateway is where availability, latency, guardrails, and cost fight for priority. Every safeguard trades one against another. This is a more honest framing than "gateways are infrastructure you should have" — it forces teams to name which axis they're willing to sacrifice before designing anything.

Three production insights that extend beyond the Uber platform framing from Day 2:

**Standard microservice patterns aren't safe defaults for LLM workloads.** Retries and circuit breakers — built for stateless, fast, homogeneous calls — fail for LLMs in specific ways: retrying the same provider burns latency budget, a tripped breaker fails requests even when a secondary was available, and blind retries multiply cost. The fix (eject the primary from the path rather than retry it) is a small change in mental model but a significant change in behaviour.

**Streaming is a one-way door.** Once the first token is sent, you lose the ability to fall back, retry, or rewrite. This is a concrete trade-off that most teams don't make explicitly — they stream because it feels responsive, not because they've accepted the loss of recovery options. Manuja's prescription: stream only when latency demands it.

**Guardrails are dependencies, not settings.** They can fail. That failure requires an explicit per-policy decision (fail-open vs. fail-closed) decided by blast radius, not a global switch. This is a new failure mode most gateway designs don't address. See [Guardrails](concepts/guardrails.md).

**Centralise governance, not traffic.** Manuja's architectural conclusion directly challenges the Uber pattern: a single central gateway process is a SPOF. The better split is decentralised resilience (each service owns its own fallback, timeouts, load shedding) with centralised governance (cost attribution, guardrail-failure events flow to a shared control plane). This is a genuine architectural disagreement in the conference record — Uber optimises for policy consistency at scale; Twilio optimises for blast-radius isolation.

See: [Productionizing LLM Gateways](talks/day2-1425-productionizing-llm-gateways.md) · [Model Gateway concept](concepts/model-gateway.md)

---

## Key Debates

- **vLLM vs SGLang for agents**: most teams default to vLLM even for agentic workloads — the benchmark suggests this is wrong. The Akamai workshop is entirely vLLM-centric; their agent from Module 9 likely benefits from SGLang's RadixAttention on shared-prefix workloads.
- **Quantization vs attention architecture**: both address KV cache size from different angles. Which gives more headroom at scale?
- **Compact or keep?**: The context engineering workshop's finding (keep-everything wins at 11–13 turns) is at odds with the conventional wisdom that context should be aggressively managed. The question is at what session length / scale the calculus flips.
- **Local vs cloud for agents**: At ~10k turns/day, the gap between DeepSeek ($1.9k/mo) and Gemini ($34k/mo) is already an 18× decision. The Akamai workshop adds the self-hosted GPU as the third option. At what volume does a dedicated $1.50/hr card beat both?
- **Routing vs. serving optimisation**: Do you reduce cost by routing to a cheaper model, or by making the expensive model cheaper to run (quantization, caching, batching)? The Day 1 and Day 3 talks represent two different approaches to the same cost problem. The answer is both, but the interaction between them — does serving-layer optimisation change the routing economics? — is unexplored in the talks.
- **Cache-aware routing in practice**: Kofman claims naive routing that ignores KV cache state can lose all of its paper savings. This is a strong claim. How often does it hold in practice vs. short sessions where the cache economics don't matter much?
- **Central vs. decentralised gateway**: Uber centralises the full middleware chain for policy consistency. Manuja argues this creates a SPOF and advocates for each service owning its resilience. Which failure mode is worse in practice — policy inconsistency or shared-gateway outage? This likely depends on org size and compliance obligations.
- **FP8 vs INT4**: The Harshul/Tanmay session emphasised INT4; the Akamai workshop uses FP8. Both reduce weight bytes; FP8 is more conservative on quality risk and better supported by current serving stacks.

## Emerging Consensus

- GQA is now table stakes — it's shipped in every major model.
- FlashAttention is default-on everywhere; there's nothing to configure.
- Prefix caching is underutilized in agent frameworks.
- Measurement-first culture is a consistent thread: all four workshops emphasised running real harnesses rather than accepting conventional wisdom.
- An agent's wall-clock time is almost entirely inference. The inference layer is the agent latency budget.

## Open Questions

- Do compounding optimizations truly multiply in practice, or do some interfere?
- Does RadixAttention's advantage hold at very long contexts (>32K)?
- At what task complexity does INT4 quality degradation become meaningful?
- How does disaggregated prefill/decode fit in the stack at what scale does it become worth the complexity?
- At what session length does compaction become cost-effective over keep-everything + caching?
- Does "lost in the middle" context rot emerge for reasoning-heavy agent tasks at >800k token contexts?
- Under what prompt distributions does a 0.6B drafter consistently achieve >70% acceptance with a 4B target?
- When does scale-to-zero (KServe/Knative) change the self-hosted economics for bursty workloads?
- How does Uber's Agent Skills quality gate (120-pt eval, model gate) compare in rigor to the eval harnesses built by Towards AI for context engineering — is there a shared eval discipline emerging across "agentic infrastructure" generally?
- Does Uber's Context Graph face the same "context rot" risk at scale that the Towards AI workshop found for chat history, or does graph traversal sidestep that failure mode entirely?
- Does uReview's "Custom Agents" run on the same Agent Skills infrastructure (SKILL.md, `aifx`) described in the Agentic SDLC talk, or a separate skill system specific to code review?
- Where does the Generator-Verifier pattern (official abstract) map onto the uReview Architecture diagram (slides) — most likely Post-Process, but unconfirmed?
- Is Uber's "smart router" in the model gateway doing anything like the RL-based session routing described by Not Diamond, or is it closer to deterministic/heuristic routing?
- Manuja recommends in-parallel guardrail execution as the default. For very fast model calls (embeddings, classifiers), the model completes before the guardrail — is the parallel pattern still valid, or does it collapse into a post-hook in practice?
- The blended-latency problem (gateway-wide p99 is misleading when mixing reasoning and chat workloads) — do model routing systems like Not Diamond's account for per-route latency SLOs, or do they still work with aggregate metrics?
- What is in Not Diamond's "derived, schematized metadata"? If it includes task type classification or tool-schema fingerprints, how does that interact with the privacy claim?
- The Not Diamond Terminal-Bench result uses Anthropic models only. How does the Pareto curve shift when OSS models (DeepSeek, Qwen3, GLM) are added to the pool?
- Does the Factory sequential-worker design hold at very long missions — do earlier workers' decisions constrain later ones in ways that hurt quality despite the fresh-context handoff?
- Tížková claims agents will eventually run for a year without interruption. At 50% reliability today and task-length doubling every 7 months, what's the implied timeline to 99% reliability?
- Factory's deferred context engine cuts 15–51% of input tokens. How does this interact with the KV-cache hit rate — does deferring tool schemas that might have been in a stable prefix hurt or help caching?

---

### 17. The Software Factory as a Unifying Frame for the Conference

Tížková's talk is the most explicit attempt to synthesize what the conference is about. The "software factory" label names a pattern that was visible across earlier sessions without being named: Uber's six building blocks form a factory floor; Not Diamond's router is the factory's model-selection layer; Manuja's gateway is the factory's governance layer; the Towards AI context-engineering workshop is about what the factory's agents see on every call.

The three-property framework (agnostic, autonomous, always-improving) is also a useful retroactive map of where the conference's content falls:

| Property | Conference coverage |
|----------|-------------------|
| Agnostic | Model routing (Not Diamond, Factory Router), LLM gateways (Uber, Twilio), self-hosted inference (Akamai) |
| Autonomous | Long-horizon missions + validation architecture (Factory), Agentic SDLC (Uber), uReview (Uber) |
| Always-improving | Agent readiness + repo scoring (Factory), DevPods + Agent Skills (Uber), context engineering (Towards AI) |

The most direct tension in this talk against the rest of the conference: Tížková says capability is "largely solved" and reliability is the frontier. The Day 1 inference sessions suggest serving cost and throughput are still very much unsolved at scale. Both can be true — they're speaking to different parts of the stack — but the implied "the hard part is done" framing deserves scrutiny.

A second tension: the "environment problem not a model problem" thesis. The GitClear/DX data shows 3–4× metric degradation for Level 1 repos. That's real, but the same data shows Level 2+ repos still seeing +29% cognitive complexity and +34% duplication — not zero. The environment framing is correct directionally but may undersell genuine model-capability limits.

See: [Software Factory](concepts/software-factory.md) · [Agent Readiness](concepts/agent-readiness.md) · [Rise of the Software Factory](talks/day2-1110-rise-of-the-software-factory.md)

---

### 18. The Intern Model — Constraints as the Path to Reliable AI Adoption

Shashank Goyal's talk offers the clearest answer to a question the rest of the conference dances around: how do you actually get an entire organization using AI agents? The answer isn't better models or more capable agents — it's narrower ones.

The "intern" framing reframes reliability from a capability problem to a design problem. A generalist agent that can do anything will wander; an agent with a well-defined job, a Slack home, and a Git repo for its own configuration will not. OpenRouter's results (100% employee adoption, 21 sessions/employee/week, 84% autopilot rate across 31k sessions) are the strongest deployment claim at the conference and provide a real-world validation of the "constraints = reliability" thesis.

**What makes this a novel contribution** over the other agent platform talks (Uber's agentic SDLC, Factory's software factory):

- **Individual agent identity.** Uber builds shared infrastructure (gateways, DevPods, skills catalog). Factory builds a factory floor. OpenRouter gives *each agent its own Git repo, VM, and Slack presence* — agents as persistent, named, auditable entities rather than ephemeral sessions.
- **The egg pattern.** Fleet-wide propagation of a base template that each intern then evolves independently. This is a fleet management answer that neither Uber nor Factory explicitly addresses.
- **OS-level secrets enforcement.** The `systemd InaccessiblePaths` + privileged sibling pattern is the most technically specific secrets isolation design described at the conference, and the key to unlocking broader agent capabilities safely.
- **Self-evolving agents as a requirement.** The lesson "self-evolving agents are necessary for agents to become integral" names something that Uber's DevPods and Agent Skills touch on implicitly but don't state as a fleet management principle.

**Tension with the Software Factory frame (theme 17):** Tížková argues the software factory is the unifying frame for the conference. The intern model is complementary but different in emphasis — factory = throughput and automation of the dev cycle; interns = persistent, specialized, delegatable coworkers. The Uber talk sits between them (infrastructure for many agents) but doesn't resolve which frame wins at scale. Open question: is a fleet of specialized interns a software factory, or a different thing entirely?

**New open questions raised by this talk:**
- At what fleet size does the "one VM per intern" model become expensive relative to a shared compute pool? OpenRouter has 73 interns; Uber runs 2,500+ skills across many agents. The architectures seem different at different scales.
- "Model preference is personal to each intern" — if each intern is eval'd to its own cheaper model over time, does that create a heterogeneous fleet that's harder to debug? What's the governance model for model versions across 73+ interns?
- The egg propagation pattern: how do you push a breaking change to the base template without disrupting live interns? Versioning + migration strategy not described.
- 89% cache hit rate on input tokens — this is a striking validation of the context-engineering workshop's "keep-everything + caching" thesis. Does OpenRouter achieve this by having each intern's system prompt as a stable prefix, or by another means?

See: [Letting the Interns Loose](talks/day3-1110-letting-the-interns-loose.md) · [Shashank Goyal](speakers/shashank-goyal.md) · [Agent Skills](concepts/agent-skills.md) · [Guardrails](concepts/guardrails.md)

---

### 19. Agent Environment Architecture: The Loop Is Solved, the Environment Is Not

Adam Azzam's talk is the most direct statement of a pattern that was implicit across multiple prior sessions: the hard engineering problem has moved from the agent loop to the environment the agent runs in. Devboxes and orchestration are where teams repeatedly reinvent the same wheel.

The central contribution is naming the trust problem precisely: CI and agent devboxes look similar (isolated environments executing code) but differ on the dimension that matters most — who owns the control flow. CI executes a fixed, reviewed script; an agent generates and executes its own code in a loop. "The call is coming from inside the house" — the threat model is completely different, and the blast radius is much larger.

The structural answer is the **Control Plane / Data Plane split**: the agent lives in the control plane, provisioning and manipulating devboxes that run in the data plane behind a network boundary. The agent never runs directly on the compute it manipulates. Secrets live in sidecars/proxies accessible only through controlled interfaces, not in the agent process.

The Ramp case study (on Modal) gives this architecture a concrete instantiation: per-repo images defined as code, rebaked every 30 minutes asynchronously, mounted warm at devbox creation. Startup cost is never paid at task time. Secrets flow through proxies, never to the agent directly.

**How this connects the conference record:** three talks now describe solutions to the same problem — fast-booting, secrets-isolated agent workspaces — using very different stacks but converging on the same principles:
- **Ramp/Modal (Azzam):** managed image snapshots, 30-min cron, Modal as the substrate
- **Uber (Medisetty/Huda):** K8s balloon pods + warm pool + snapshot store at 14K+ scale
- **OpenRouter (Goyal):** one VM per agent, `systemd InaccessiblePaths` for OS-level secrets isolation

The convergence is notable: independently, three production teams arrived at "pre-bake, mount warm, isolate secrets from the agent process." This is close to an emerging consensus on devbox design.

The talk also adds a new open question to the debate: at what scale does the "one VM per agent" model (OpenRouter, 73 interns) become more expensive than a shared warm pool (Uber, 14K pods)? And how do the economics change if the devbox MCP is built as a "headless background agent" rather than a static infrastructure component?

See: [Agent Environment Architecture](concepts/agent-environment-architecture.md) · [DevPods](concepts/devpods.md) · [Don't Build Agents, Build Environments](talks/day3-1045-dont-build-agents-build-environments.md)

---

### 20. Do You Even Need a Full Sandbox? The Minimal Runtime Argument

Samuel Colvin's talk is the sharpest provocation in Track 1: while Azzam, Goyal, and the Uber team spent the morning arguing about *how* to build better sandboxes, Colvin asks whether most agents need a full sandbox at all.

The "desert" framing is memorable: a raw Linux shell exposes dozens of binaries (`awk`, `gcc`, `bash`, `ssh`, `apt`...) that a typical embedded workflow never needs. More concretely, every full sandbox — VM, container, gVisor, Firecracker — carries a **1–3s boot tax**. For heavy tasks (clone a repo, run a build), that's noise. For light tasks (run a query, render a chart) that take milliseconds, paying 1–3s of overhead is a 1,000–3,000× ratio. Colvin quotes a 100,000× latency gap in the worst case.

The codemode lens makes this more precise. In codemode / CodeAct, the LLM writes Python that orchestrates many tool calls in a single pass — the agent only ever exposes two capabilities (`run_code`, `find_tools`). The actual domain tools (`sql_query`, `create_chart`, etc.) live behind that interface. For this pattern, the execution surface the agent needs is tiny and well-defined — not a full computer.

**Pydantic Monty** is the product answer: a Python interpreter written in Rust, safe by default (no filesystem, env, or network unless granted), type-checked before execution via `ty`, and snapshottable. Available as `pydantic-monty` (Python), `@pydantic/monty` (npm), or `monty` (Rust).

**How this fits the conference record:** this is not a refutation of the Azzam/Goyal/Uber talks — those address a real problem for *heavy* agent workloads. It's an argument that a large class of *light/embedded* agent workflows has been mis-categorized as requiring heavy infrastructure. The two positions bracket the design space rather than contradict each other. The open question — which real-world workflows actually fall on which side of the line — is unanswered and is now the most interesting open question in Track 1.

**A new debate:** Colvin's implicit critique of sandbox vendors ("the people who benefit most from codemode are those selling sandboxes") is the most pointed commercial critique in the conference record. Whether or not the critique lands technically, it names an incentive misalignment that the field should grapple with.

See: [Your Agent Needs a Sandbox, Not a Desert](talks/day3-1205-agent-needs-sandbox-not-desert.md) · [Agent Environment Architecture](concepts/agent-environment-architecture.md) · [Samuel Colvin](speakers/samuel-colvin.md)

---

### 21. The Environment's Security Controls Are Its Capability Constraints — and They Fail Silently

Kevin Orellana's talk is the empirical bookend to the day's architectural framing. Where Azzam, Goyal, and Colvin each argued about *how* to structure agent environments, Orellana ran 1,000 tasks through a real one and counted what broke.

The result — 88% pass rate, 18 distinct failure modes — is important not just as a number but as a categorization. Orellana splits failures into two layers. The **Interface Layer** (MCP trust, protocol misalignments, input/output validation) is about how the agent communicates with the environment. The **Sandbox Layer** (compute constraints, permission misalignments, lifecycle, fingerprinting) is about the environment itself.

The controls table is the most practically actionable artifact in the Track 1 sequence. Its central insight is that **the same primitive that isolates the environment also silently breaks domain-specific tasks**:

- Network egress controls protect the sandbox → also kill pip installs for ML agents and block internal APIs for enterprise agents
- Filesystem controls prevent data exfiltration → also route ML checkpoints to /tmp that doesn't persist and make PDFs "unexportable"
- The transport layer (MCP wire) is explicitly *not* an isolation control — yet produces the most widespread silent failures (binary coerced to base64, null bytes truncating files, stdout → stderr)

**The "silent semantic failure" framing** connects this to a named class of failure that predates LLMs: Byzantine fault (Lamport 1982), overtrusting tool results (CMU EMNLP 2024), silent semantic failures (Mehta arXiv 2026). The LLM-specific contribution is that agents don't just silently pass wrong data forward — they can *fabricate* a plausible continuation on top of it. The publicly documented 2025 case (agent deletes DB during code freeze, reports success with fabricated data) is the most severe instance.

**How this connects the conference record:**

The "capability lives in the environment" evidence (same model, +64% tasks on SWE-bench just from a better interface; F1 drops from 44.8 → 4.7 with a corrupted tool environment) is the strongest empirical statement at the conference of the "unit of performance is no longer just the model" thesis. It provides the factual grounding for a claim that was implicit in Azzam's architectural framing and Colvin's boot-latency argument but never backed by numbers.

The agent-driven testing methodology itself is a contribution distinct from the findings: using an LLM to author 1,000 tests against the same interface real agents use, storing them in SQLite, running them with a hardened runner that separates service bugs from input bugs. This is a practical answer to the question "how do you test a sandbox for agent use?" that no prior talk at the conference addressed.

**Tension with theme 20 (minimal sandbox):** Colvin asked whether most agents need a full sandbox at all. Orellana's data comes from a production full-sandbox service and reveals 18 failure modes that operators would not have caught without this testing. This doesn't settle the Colvin debate — the minimal sandbox argument is about light/embedded workloads, and AgentCore handles heavy ones. But it does add a caveat: if you *do* run a full sandbox, the failure surface is larger than you think, and silent failures are more common than crashes.

See: [1,000 Agent Tasks in a Sandbox](talks/day3-1425-1000-agent-tasks-sandbox.md) · [Silent Semantic Failure](concepts/silent-semantic-failure.md) · [Agent-Driven Testing](concepts/agent-driven-testing.md) · [Agent Environment Architecture](concepts/agent-environment-architecture.md)

---

### 22. The Closing Keynote: A Theory of the Engineer in the Agent Era

Addy Osmani's closing keynote is the conference's only talk that addresses the future of engineering as a *role* rather than as a set of tools or infrastructure patterns. Where every prior talk asks "how do you build better AI systems?", this one asks "what is left for the engineer when agents run the factory?"

The answer is structured as a triad — choose, own evidence, own verdict — and elaborated through a set of frameworks that deserve to be read as a unified theory:

**The accountability stack.** Osmani distinguishes Quality (system that produces evidence), Verdict (human decision from evidence), and Answerability (ability to explain and stand behind it later). This is the first talk at the conference to name *answerability* as a required property — not just correctness or reliability. The governance data backs the urgency: 92% of teams report AI code governance challenges, 43% can't distinguish AI from human code, and the trust-verification gap (96% distrust, 48% verify) is accelerating as AI code share reaches 42% of commits.

**The loop split as the operational answer.** The inner loop (INVESTIGATE → IMPLEMENT → TEST → REPORT) is the agent's domain. The outer loop (DECIDE → VERIFY → APPROVE → OWN) is the engineer's. The boundary is evidence. "Explain it or don't ship it. You cannot answer for what you cannot understand." This is the clearest operational definition of "human in the loop" in the conference record — it specifies *which* loop the human owns and what they must do in it, rather than leaving it as a vague principle.

**Taste and decay as the career framework.** The alpha/decay framing is the conference's most direct address of career risk. The decay curve — speed and recall gone, verification automating, taste and judgment surviving — maps well onto the evidence from earlier sessions: the inference workshops automated token routing and context management; the agent environment talks automated sandbox provisioning; Uber automated 15% of PRs end-to-end. What's left is choice, judgment, and accountability. "The half life of an edge is a release. The half life of a signature is a career."

**Three failure modes that don't show up in productivity metrics.** Cognitive debt (17% comprehension drop from AI-learned code, Anthropic), cognitive surrender (73% accept wrong answers when AI is confident, Wharton), and orchestration tax are the shadow side of every productivity gain discussed at this conference. None of these failure modes appear in PR counts or lines-of-code metrics — they accumulate invisibly. This is the strongest empirical challenge to the pure-productivity narrative in the conference record.

**How this connects to the rest of the conference.** The Osmani keynote is the retroactive frame for much of what was presented across three days:

| Osmani concept | Conference parallel |
|---|---|
| Harness engineering | Uber's SDLC building blocks; Agent Skills; DevPods |
| Loop engineering | Factory's sequential-worker design; uReview's inner/outer loop |
| Human verdict | uReview's confidence-scored approval gate; Twilio's gateway governance |
| Taste as alpha | Factory's "humans decide what to build, agents handle how" |
| Cognitive debt | Context engineering: keep-everything wins (don't abstract away too much) |
| Governance gap | GitLab/Sonar data: 92% governance challenges — the audience's immediate reality |

The strongest tension in the keynote: Osmani says "loops change the work; they do not delete the engineer." The conference's data on autonomous agent reliability (80.6% on 30-min tasks, 25.6% on 8-hour tasks) suggests we are still far from deleting the engineer. But the trend is clear enough that Osmani's frameworks — particularly the Agency Ladder and the inner/outer loop split — read as genuinely useful near-term guides rather than distant speculation.

See: [The Future of Engineering](talks/day3-1630-future-of-engineering.md) · [Addy Osmani](speakers/addy-osmani.md) · [Human Verdict](concepts/human-verdict.md) · [Loop Engineering](concepts/loop-engineering.md) · [Harness Engineering](concepts/harness-engineering.md) · [Taste as Alpha](concepts/taste-as-alpha.md) · [Cognitive Debt](concepts/cognitive-debt.md) · [Software Factory](concepts/software-factory.md)

---

### 23. Inference Is an Infrastructure Orchestration Problem — Not an ML Problem

Gupta & Ahuja's talk is the Day 4 capstone on the inference thread that began on Day 1. Where the earlier workshops focused on serving-layer mechanics (engines, KV cache, quantization), this talk zooms out to the system-level question: what does it actually take to *operate* inference at scale?

The central reframe is striking: of the 12 steps that happen when a prompt arrives, only 2 (Generate and Safety) are ML. The other 10 are infrastructure. **"Which of these are actually ML? Almost none. They're infrastructure."**

This is the clearest statement at the conference that the inference reliability problem is fundamentally a distributed systems problem, not an ML problem. Every prompt is a distributed transaction — each hop (gateway, router, cache, scheduler, serving runtime) can fail, time out, or degrade. Classical distributed systems patterns apply: idempotent retries, per-hop and end-to-end timeouts, fallbacks to smaller/colder/degraded models, and distributed tracing as a requirement.

**Agent-aware scheduling** extends the classic bin-packing problem from 2 dimensions (CPU, memory) to 7 (model, GPU memory, latency SLA, tenant, context size, cost, reasoning workflow). The critical addition is the reasoning workflow dimension — the scheduler needs to know where a request sits in a multi-step agent chain to optimize the *whole workflow*, not just the immediate GPU assignment.

**The impossible triangle** (latency / cost / throughput) resolves the "which configuration is best?" question cleanly: there is no global optimum, only a continuous repositioning based on real-time load and tenant SLOs. Optimization is continuous, not a config file.

**The inference control plane** names the architectural destination: a dedicated system handling routing, scheduling, autoscaling, cost optimization, observability, policy, reliability, admission control, and multi-tenancy — the Kubernetes equivalent for AI serving. This is the closest thing in the conference record to a "what should we build next" prescription. See [Inference Control Plane](concepts/inference-control-plane.md).

**Five operational lessons** with the feel of hard-won production wisdom: (1) infrastructure bottlenecks appear before model bottlenecks; (2) elasticity beats overprovisioning; (3) scheduling decisions dominate efficiency; (4) visibility precedes optimization; (5) control loops outperform manual operations.

The AI infra progression they describe — Better Models (past) → Better Serving (present) → Better Orchestration (future) — is a clean map of the full conference. This talk declares the serving phase largely done and points to orchestration as the open frontier.

See: [Operating Distributed Inference Systems at Scale](talks/day4-1045-operating-distributed-inference-systems-at-scale.md) · [Inference Control Plane](concepts/inference-control-plane.md) · [GPU Scheduling](concepts/gpu-scheduling.md)

---

### 24. The Engine Routing Layer: From Heuristic Feedback Loops to Optimizer-Based Policy

Lao & Zhang's Day 4 talk (immediately after Gupta & Ahuja) provides the most detailed production implementation of the inference control plane's routing responsibility — and reveals how much engineering depth sits in what sounds like a simple question: which GPU engine serves this request?

The talk traces a natural evolution. The "early days" approach — weighted consistent hashing with periodic EWMA/PID-style weight updates — works well enough to ship: it considers many signals, self-corrects, and "just works." But it has a structural flaw: the feedback loop creates oscillations, and those oscillations directly degrade KV cache hit rates. When routing weights swing, requests that would have hit a warm cache miss instead. The optimization is self-undermining.

The solution is a clean architectural separation: **control plane** (global view, async, runs an Optimizer that produces routing weight matrices) and **data plane** (local, synchronous, uses pre-computed weights with no per-request blocking). The key design constraint — **no request waits on the data plane** — is the same principle that motivates zero-copy, lock-free, and pre-computed-index patterns across distributed systems. The routing decision must be instantaneous; the optimization work happens off the hot path.

**The Optimizer formulation** is worth highlighting as a production artifact. It frames engine routing as a constrained optimization:
- Minimize: expected end-to-end latency across all routed traffic
- Subject to: route all demand, stay within effective engine capacity, keep weights non-negative

This is a linear (or close to linear) program solved repeatedly as engine signals update. The inputs — per-CPU-cluster demand, network latency, engine capacity + health, TTFT/TBoT latency profiles — are exactly the signals that the control plane collects async from the fleet.

**How this connects the conference record:**

The Gupta & Ahuja talk (theme 23) declared inference orchestration the open frontier and named the inference control plane as the architectural destination. This talk provides one of the most detailed views into what that plane's routing component actually looks like in production at OpenAI. Two talks, back-to-back in the same track, at different levels of abstraction: Gupta/Ahuja on what the control plane must do; Lao/Zhang on how one critical piece of it actually works.

The KV cache connection is notable. The talk explicitly names KV cache match as a routing signal alongside TBoT, TTFT, load, and network overhead. This adds a third tier to the KV cache signal stack that has been building across the conference:
- Tier 1 (API layer): cache hit rate for prompt prefixes (Towards AI, Factory)
- Tier 2 (model routing): don't switch models mid-session; it breaks the prefix (Not Diamond)
- Tier 3 (engine routing): prefer the replica that already holds the prefix (OpenAI ILB)

At every level, the same economic intuition applies: recomputing what you already computed is the most expensive per-token operation in the system. The routing layer should minimize it.

**A new debate:** the early-days system described here is not entirely unlike the "periodic feedback loop" with PID-like weight updates used in many serving systems, including those built on vLLM/SGLang. The specific failure mode — oscillations hurting KV cache hit rate — is worth checking against any system running consistent hashing with EWMA weight updates. This is a concrete, measurable failure mode that teams can test for.

See: [Routing LLM Inference in Production](talks/day4-1110-routing-llm-inference-in-production.md) · [Inference Control Plane](concepts/inference-control-plane.md) · [KV Cache](concepts/kv-cache.md) · [Qianru Lao](speakers/qianru-lao.md) · [Lu Zhang](speakers/lu-zhang.md)

---

### 25. From Dev to Ops: Background Agents and the Invisible Toil Problem

Justin Smith's Day 4 talk is the first at the conference to address AI agents applied to *operations* rather than development. Where every prior talk in the Agentic Engineering track asked "how do you build software faster with AI?", this one asks "who runs the system once it's built?"

The central contribution is the **Production Context formula**: `Task = Execution × Production Context`. Execution — the actual analysis or change — is small and roughly fixed. Production Context — knowing how to execute and evaluate the work in your specific environment — is large and growing. **Production Context >> Execution.** The implication: the dominant cost in operational work is not the task itself but the overhead of re-navigating the environment every time a person takes it on.

This is a cross-conference resonance worth naming explicitly. The same pattern appeared at three other abstraction layers this week:

| Layer | Talk | The "context dominates" finding |
|-------|------|---------------------------------|
| LLM serving | Towards AI (Context Engineering) | KV cache hit rate at the API level is as important as serving efficiency |
| Tool access | Uber (Context Graph) | 8 vs. 94 tool calls for the same correct answer — navigating tool schemas is the agent's time cost, not the query |
| Agent workspace | Azzam / Goyal / Uber (Devboxes) | Environment boot time and secrets plumbing are the dominant setup costs, not the task execution |
| Production ops | Smith (Resolve AI) | Navigating the production environment is the dominant operational cost, not the work |

The conference's implicit thesis is becoming explicit: **at every layer of the AI stack, context overhead dominates execution cost.**

**Background agents** are the architectural answer Smith proposes: always-on, schedule- or trigger-driven agents with persistent state and environmental knowledge that accumulate over time. They differ from on-call and incident agents in one critical way: they run without a trigger from a human. The agent wakes on a schedule or event, carries durable state, and surfaces findings — the engineer only opens the tool to verify.

Three new operational concepts from this talk (1st sources only — concept pages pending a second source):
- **Invisible toil**: operational work with no critical trigger, no sprint slot, no owner — accumulates silently, disappears when people are stretched thin
- **Production Context**: the environmental knowledge that dominates operational task cost; large and growing; the primary contributor to toil
- **Background Agents**: always-on, schedule/trigger-driven agents with durable state and environmental reasoning

**How this extends the Human Verdict pattern:**

This talk gives the Human Verdict concept its clearest operational instantiation. Osmani named it abstractly ("the human renders the verdict on agent-generated evidence"); uReview implemented it for code review; Resolve AI implements it for production operations. Smith's exact framing — "you open Resolve AI to verify findings, and stand up new work in a sentence" — is operationally the same outer-loop step Osmani called VERIFY → APPROVE → OWN, now applied to whether a deployment looks healthy rather than whether a PR should merge.

**How this extends the Software Factory frame:**

Theme 17 described autonomous agents running the dev cycle (spec → build → validate → deploy). This talk describes autonomous agents running the operational lifecycle (deploy monitoring → health checks → incident digests → anomaly response). Together they suggest a complete picture: AI agents handling the full continuous lifecycle — build, ship, run, learn, repeat. The "always-improving" property of the Software Factory and the "learning" layer of background agents (explicit feedback + implicit signals) point at the same capability from opposite ends.

**New open questions:**

- The "Production Context >> Execution" ratio is argued qualitatively. Is it measurable, e.g., as the ratio of time-spent-navigating vs. time-spent-executing in a real production incident?
- Background agents accumulate knowledge via explicit feedback and implicit signals. How does this compare architecturally to Uber's Agent Skills eval gating (120-pt eval, model gate)? Is environment-specific operational context a different problem from task-specific skill reliability?
- If Resolve AI exposes an MCP server, can background-agent capabilities be composed via Uber's MCP Gateway without re-architecting? That would make "always-on production monitoring" a callable skill inside any agentic workflow.

See: [Always-on agents run production without the on-call tax](talks/day4-1425-always-on-agents-production-on-call-tax.md) · [Justin Smith](speakers/justin-smith.md) · [Human Verdict](concepts/human-verdict.md) · [Software Factory](concepts/software-factory.md) · [Context Engineering](concepts/context-engineering.md)

---

## Sessions Ingested

1. [LLM Inference at Scale Workshop](talks/day1-1210-llm-inference-at-scale.md) — Harshul Jain & Tanmay Sah
2. [Context Engineering in 2026: Compaction, Memory & Cost](talks/day1-1420-context-engineering-2026.md) — Louis-François Bouchard, Samridhi Vaid, Omar Solano
3. [What is an Inference Engine, Anyway?](talks/day1-1105-what-is-an-inference-engine-anyway.md) — Charles Frye (Modal)
4. [Agents That Own Their Inference](talks/day1-0900-agents-own-inference.md) — Du'an Lightfoot / Omer Aslan (Akamai)
5. [Rise of the Software Factory](talks/day2-1110-rise-of-the-software-factory.md) *(full write-up)* — Tereza Tížková (Factory)
6. [Agentic SDLC at Uber: Building Blocks for Uber's Software Factory](talks/day2-1140-agentic-sdlc-at-uber.md) *(partial — notes cover 5 of 6 building blocks; SDLC stage walkthrough pending)* — Uday Kiran Medisetty, Adam Huda
7. [Scaling Code Quality: Building uReview, Uber's Multi-Agent Code Review Engine](talks/day2-1205-ureview-uber.md) *(4 slides captured; Generator-Verifier trust-layer detail from official abstract only)* — Will Bond, Ameya Ketkar
8. [Intelligent Model Routing: Frontier Performance Without Frontier Bills](talks/day3-1450-intelligent-model-routing.md) *(13 of 20 slides captured)* — Tomás Hernando Kofman (Not Diamond)
9. [Productionizing LLM Gateways: Architecture, Tradeoffs, and Hard Lessons from the Trenches](talks/day2-1425-productionizing-llm-gateways.md) *(14 slides, full deck)* — Kanish Manuja (Twilio)
10. [Letting the Interns Loose — How We Accelerated AI Adoption](talks/day3-1110-letting-the-interns-loose.md) *(~14 of ~33 slides)* — Shashank Goyal (OpenRouter)
11. [Don't Build Agents, Build Environments](talks/day3-1045-dont-build-agents-build-environments.md) *(full deck, 9 slides)* — Adam Azzam (Modal)
12. [Your Agent Needs a Sandbox, Not a Desert](talks/day3-1205-agent-needs-sandbox-not-desert.md) *(6 of 18 slides; missing Intro, Security, Demo)* — Samuel Colvin (Pydantic)
13. [1,000 Agent Tasks in a Sandbox: What Breaks When LLMs Write and Run Code](talks/day3-1425-1000-agent-tasks-sandbox.md) *(10 framing/methodology slides; detailed per-failure-mode breakdown slides not captured)* — Kevin Orellana (Amazon AgentCore)
14. [The Future of Engineering (Closing Keynote)](talks/day3-1630-future-of-engineering.md) *(32 slides; 1 slide missing from upload)* — Addy Osmani
15. [Operating Distributed Inference Systems at Scale](talks/day4-1045-operating-distributed-inference-systems-at-scale.md) *(Act 1 slides not captured; Acts 2–4 complete)* — Nishant Gupta & Naman Ahuja (Meta)
16. [Routing LLM Inference in Production: From Engine Signals to Policy](talks/day4-1110-routing-llm-inference-in-production.md) *(10 slides, full deck)* — Qianru Lao & Lu Zhang (OpenAI)
17. [Always-on agents run production without the on-call tax](talks/day4-1425-always-on-agents-production-on-call-tax.md) *(10 slides, full deck)* — Justin Smith (Resolve AI)
