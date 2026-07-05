# Workshop Notes: Agents That Own Their Inference — Building Production AI Agents on Dedicated GPUs

**Presenters:** Du'An Lightfoot (Modules 0–4, 9), Omer Aslan (Modules 5–8)  
**Format:** Jupyter notebooks on Akamai Cloud GPU (Kubernetes / LKE), vLLM serving Qwen3-4B  
**Repo:** https://github.com/akamai-developers/akamai-workshop-ai-inference  
**Source:** Text notes — no slide photos taken.

---

## Workshop premise

Every production agent today rents its intelligence — paying per token, sending customer data to someone else's servers, and hoping the provider doesn't rate-limit during a launch. This workshop flips that. You get a dedicated GPU, operate the inference server yourself with vLLM, and learn to read and tune the layer your agent sits on top of.

The focus is **not** agent frameworks. It is the inference layer underneath them.

---

## Module 0: Connect and Verify

**What it covers:** Orientation — confirming the environment works before doing anything else.

The setup uses four environment variables (`VLLM_HOST`, `MODEL_NAME`, `NAMESPACE`, `KUBECONFIG`) that every later module inherits. The notebook pod has no GPU; inference runs in the vLLM pod on a dedicated GPU node, accessed over HTTP.

**Learning points:**
- The notebooks never provision infrastructure — they read from environment variables, so the same code works whether the environment was handed to you or you built it yourself.
- `kubectl` is scoped to your namespace by design; you can't see a neighbour's resources and they can't see yours.
- The vLLM server speaks the OpenAI API. The only change from using the hosted OpenAI API is the `base_url`. Your existing client code barely changes.
- The metrics URL is derived from `VLLM_HOST` automatically (`/v1` → `/metrics`); it's not a separate variable.

---

## Module 1: The Inference Stack and Runtime Landscape

**What it covers:** Making the case for self-hosted inference; tracing one request end-to-end; the runtime landscape.

**Learning points:**

**The two phases of a request:**
- **Prefill** — the whole prompt is read in one parallel pass. Time to the first generated token is the prefill latency (TTFT: time to first token).
- **Decode** — one token is generated at a time, sequentially. Speed is measured as TPOT (time per output token) or tokens/second.

**Renting vs owning:**
- **Cost** — hosted billing scales with every token; a dedicated GPU is a flat hourly rate. There's a crossover volume where owning becomes cheaper, and it's lower than most people expect.
- **Data residency** — self-hosted requests never leave your cluster. For regulated data, this is a compliance line, not just a preference.
- **Rate limits** — a hosted provider sets your throughput ceiling. On your own server the only ceiling is the card you provisioned.
- **Control** — you pick the model, precision, context length, and batching policy. You tune for your traffic instead of accepting defaults.

**An agent is many requests:**
- A real agent loops: calls the model, reads a tool result, calls again, re-sending a growing prompt each step.
- The agent's total wall-clock time is the sum of its steps. The slowest step sets the pace.
- Prompt tokens grow every turn and never shrink — the "context tax." That's why you tune the worst step, not the average.

**Runtime landscape:**
- **vLLM** — broad default for GPU serving. PagedAttention KV cache, continuous batching, wide model/quantization support, OpenAI-compatible server. Best for running many models fast with no per-model compile step.
- **SGLang** — best when requests share a long prefix (multi-turn chat, agents, RAG). Prefix cache reuses KV cache across requests; fast at structured output.
- **TensorRT-LLM** — best for peak throughput/latency on NVIDIA hardware. NVIDIA-only, with an upfront tuning cost. Fits stable models in long-running production.
- **llama.cpp** — best for local, edge, and CPU-first inference. Runs heavily quantized models from a single file on a laptop or Arm server.
- All four expose an OpenAI-compatible server, so client code moves between them.

---

## Module 2: Units and the Memory Budget

**What it covers:** Sizing a model against a GPU from first principles — parameters, bytes, VRAM, KV cache headroom, concurrency limits.

**Learning points:**

**A model is files:**
- A model directory contains: a `config.json` (architecture), safetensors files (the weights), and a tokenizer.
- The `config.json` fields that matter: `num_hidden_layers`, `hidden_size`, `num_attention_heads`, `num_key_value_heads`, `head_dim`, `vocab_size`, `torch_dtype`.

**Weight footprint:**
- Weight footprint (GB) = parameter count × bytes per weight.
- BF16 = 2 bytes/weight. FP8 = 1 byte/weight. FP32 = 4 bytes/weight.
- Qwen3-4B at BF16 ≈ 8 GB. At FP8 ≈ 4–5 GB (some tensors stay at higher precision).

**The GPU memory budget — three things share VRAM:**
1. **Weights** (fixed cost, paid at startup, doesn't change with traffic)
2. **KV cache** (variable — grows with concurrent requests and sequence length)
3. **Overhead** (activations, CUDA context, engine bookkeeping)

Weights are paid first. What's left (scaled by `--gpu-memory-utilization`) becomes the KV pool. The pool size determines how many concurrent requests fit.

**Bandwidth ceiling for decode:**
- To make one token, the GPU reads all model weights once.
- Single-stream decode ceiling ≈ bandwidth (GB/s) ÷ weight size (GB).
- RTX 4000 Ada: 360 GB/s ÷ 8 GB (BF16 4B) ≈ 45 tok/s ceiling.
- FP8 halves the weight size → raises the ceiling proportionally.

**KV cache memory formula:**
```
KV bytes per token = 2 × layers × kv_heads × head_dim × precision_bytes
```
For Qwen3-4B (BF16): ≈ 144 KiB per token.

**Grouped-query attention (GQA):**
- Query heads outnumber key/value heads (e.g., 8:1 ratio in Qwen3-4B).
- This makes the KV cache 4–8× smaller than full multi-head attention would require.
- More agents fit on one card as a direct result.

**Context tax in practice (Qwen3-4B, BF16):**
- 1 request × 8192 tokens ≈ 1.2 GB
- 32 requests × 8192 tokens ≈ 39 GB → exceeds the whole pool
- Shorter sequences pack far more requests per card.

**Key insight:** KV pool tokens ÷ sequence length = concurrent requests that fit. This is the concurrency limit Module 7 pushes against.

---

## Module 3: Prefill, Decode, and the KV Cache

**What it covers:** Building the intuition for attention and the KV cache from scratch; measuring real server metrics.

**Learning points:**

**Self-attention is quadratic in sequence length:**
- For T tokens: the attention scores matrix is T×T. Double the tokens → 4× the matrix.
- Keys and values are linear in length. The KV cache stores these linear structures to avoid quadratic recompute.

**The KV cache trade:**
- Without it: every decode step recomputes keys and values for the entire sequence — O(N²) work.
- With it: store keys/values once, reuse them — O(N) work per step.
- Cost: GPU memory (sized in Module 2). Benefit: dramatically faster decode.
- At N=300 tokens: cached generation does ~150× fewer key/value computations than naive generation.

**Prefill vs decode behaviour:**
- Prefill processes all prompt tokens in parallel → fast per token, compute-bound.
- Decode generates one token at a time → sequential, memory-bound (limited by weight read bandwidth).

**Metric discipline:**
- vLLM renames gauges across versions. Resolve metric names at runtime rather than hardcoding them.
- The KV cache gauge is live — you can watch one request fill it during generation.
- Under load, the KV gauge is the first signal of saturation.

**Prefix caching (automatic in vLLM):**
- vLLM hashes each cache block by its token IDs. Requests sharing a common prefix reuse those blocks — prefill for the shared prefix is skipped.
- A fixed system prompt, tool schemas, or a shared RAG document is prefilled once and reused for every subsequent request.
- Practical effect: a warm request with a long shared prefix can have dramatically lower TTFT than a cold one (measured: 2–5× faster to first token in the demo).
- Breaking the prefix (changing even one early token) invalidates the cache for everything after it.

---

## Module 4: Dense vs MoE on the GPU

**What it covers:** The roofline model; why decode is memory-bound; mixture-of-experts (MoE) as a way to read fewer bytes; the reasoning tax; a look forward to prefill-decode disaggregation.

**Learning points:**

**The roofline model:**
- Two limits: a sloped memory-bandwidth line and a flat compute line. Where they cross is the **ridge point**.
- Below the ridge → memory-bound. Above it → compute-bound.
- Ridge point for RTX 4000 Ada: ~297 FLOP/byte (107 TFLOP/s ÷ 0.36 TB/s).
- Decode at batch 1 sits at ~1 FLOP/byte — far down the memory side, reaching only ~0.3% of peak compute.
- Prefill (many tokens in one pass) reuses each loaded weight across all tokens → sits near the compute line.
- The ridge clusters near 300 FLOP/byte across every GPU (compute and bandwidth scale together), so **decode is always memory-bound**.

**Mixture-of-Experts (MoE):**
- A dense model reads all its weights per token.
- An MoE model holds many experts but routes each token to only a few active experts.
- **Total parameters** set the VRAM footprint. **Active parameters** set the per-token bandwidth cost.
- Example — Qwen3-30B-A3B: 30.5B total, but only ~3.3B active per token (8 of 128 experts fire).
- Decode reads ~6.7 GB/token vs ~61 GB for a dense 30B → ~9× faster decode at the cost of storing the full 30.5B in VRAM.
- An MoE trades VRAM footprint for a smaller per-token memory read.

**The reasoning tax:**
- Qwen3 models are "thinking models" — by default they write a reasoning trace before the answer.
- Each reasoning token is full decode work. A trace of hundreds of tokens adds seconds per agent step.
- Practical knob: thinking off for steps that just need a tool call; thinking on for steps that need a plan.
- Runaway traces can fill the output context and return no answer.

**Look forward — prefill-decode disaggregation:**
- Prefill is compute-bound, decode is memory-bound. On one GPU they compete: a big prefill stalls everyone's decode.
- The production answer is to separate them onto different GPU pools and ship the KV cache between them (vLLM's `--kv-transfer-config` with NIXL, NVIDIA Dynamo, llm-d).
- The win is at fleet scale with smart routing — size and scale the two phases independently.

**The optimisation ladder (preview for Modules 5–8):**

| Layer | What it wins | Mechanism |
|---|---|---|
| Algorithm | skip recompute | KV cache (Module 3) |
| Reuse | skip re-prefill | prefix caching (Module 3) |
| Architecture | a smaller cache | GQA, fewer KV heads (Module 2) |
| Precision | fewer bytes per weight | FP8 quantization (Module 5) |
| System | share the weight read | continuous batching (Module 7) |

---

## Module 5: Quantization

**What it covers:** Establishing a BF16 baseline; switching the vLLM deployment to FP8 via a Kubernetes manifest edit; comparing performance before and after.

**Learning points:**

**Why quantization is the first hypothesis after the roofline:**
- Decode is memory-bound → fewer bytes per weight = fewer bytes read per token = higher throughput ceiling.
- FP8 also frees VRAM previously occupied by weights, leaving more room for the KV cache.

**Quantization format map (ordered by compression):**

| Method | Bytes/weight | Main upside | Main risk |
|---|---|---|---|
| BF16 / FP16 | 2 | Broad support, low surprise | Larger weight reads |
| FP8 weights | ~1 | Smaller reads, more cache room | Hardware/engine support needed |
| INT8 weights | 1 | Conservative compression | Less gain than lower-bit paths |
| INT4 / W4A16 | 0.5 | Large memory drop | More quality risk |
| AWQ / GPTQ | varies | Better low-bit quality via calibration | Calibration set can miss failures |
| KV-cache quantization | 1 (cache) | More concurrent context | Long-context drift |
| GGUF | varies | Great for llama.cpp / edge | Different serving stack |

**The workflow — hypothesis-driven, one change at a time:**
1. Measure BF16 baseline (throughput, TTFT) with a fixed prompt shape and concurrency levels.
2. Edit `manifests/vllm.yaml` — change `--model` argument only.
3. `kubectl apply` → `kubectl rollout status` → confirm via `/v1/models`.
4. Run the same sweep with the same prompt shape and concurrency levels.
5. Compare before/after; keep or reject.

**Quality is not free:**
- Production teams must run workload evals before accepting a precision change.
- Eval types: deterministic checks (JSON validity, keyword presence, refusal behaviour), LLM-as-judge, human review.
- A passing performance result that fails a quality check is not a ship.
- Tools: promptfoo, DeepEval, Ragas, OpenAI Evals, lm-evaluation-harness.

**This module's result is cumulative:** the FP8 deployment becomes the baseline for Modules 6, 7, and 8.

---

## Module 6: Speculative Decoding

**What it covers:** Adding a 0.6B draft model alongside the FP8 4B target; measuring whether the accepted tokens pay for the extra work.

**Learning points:**

**How speculative decoding works:**
1. A cheap proposer (drafter) generates several candidate tokens.
2. The expensive target model verifies those candidates in parallel (one forward pass per draft batch).
3. The server accepts the prefix the target agrees with, discards the rest.
4. The target's output distribution is preserved — the drafter doesn't unilaterally decide.

**The key metric: accepted tokens per target step.**
- If the drafter proposes 4 tokens and the target accepts 1 → mostly overhead.
- If the target accepts 3–4 → the server advances several tokens for one verification step.
- Speedup ≈ (verify_step_time / normal_step_time) × accepted_per_step.

**Drafter method landscape:**

| Method | Proposer | Best fit | Tradeoff |
|---|---|---|---|
| Draft model | Smaller autoregressive model | Easy mental model; this workshop | Extra model memory; acceptance depends on alignment |
| N-gram / suffix lookup | Repeated token sequences from history | Repetitive prompts, code, templates | No extra model; modest gain |
| Native MTP | Extra model checkpoints | Models trained with multi-token prediction | Requires model support |
| Medusa | Extra decoding heads on the target | When Medusa heads are available | Needs trained heads + tree verification |
| EAGLE / EAGLE-3 | Predicts future hidden features | Strong general-purpose method | Needs compatible EAGLE weights |
| DFlash | Block diffusion drafter | Emerging parallel drafting path | Newer stack |

**When speculative decoding helps and when it doesn't:**
- **Helps:** low-to-medium concurrency, predictable output (code, templates, repetitive text), drafter trained for the exact target.
- **Hurts or does nothing:** high concurrency (GPU already saturated; extra draft work competes with normal batching), low acceptance rate (drafter family doesn't match target well), draft depth too high (later positions rejected more often), single-GPU contention (drafter shares VRAM and compute with target).

**Published benchmarks as reference:**
- vLLM benchmark: up to 1.5× speedup with draft-model speculation on ShareGPT; up to 2.8× with n-gram on summarisation — but also reports slowdowns at higher QPS.
- EAGLE-3: up to 2.5× improvement when speculators are trained for the specific target model.
- Medusa: 2.2–3.6× speedup from extra decoding heads.

**Key discipline:** A negative result is not a failed lab — it is the production decision you would want to make before shipping a slower server.

**The module's output:** Whether speculative decoding helped or hurt, that deployment state becomes the baseline for Module 7's saturation measurement.

---

## Module 7: Engine Mechanics and Saturation

**What it covers:** How vLLM manages the KV cache and batches requests; finding the saturation knee by driving increasing concurrency.

**Learning points:**

**Why continuous batching matters:**
- A single request rarely fills the GPU during decode. Batching lets one weight read serve many requests simultaneously.
- Continuous batching (vLLM's default): requests are added and removed from the running batch as tokens are generated — no waiting for a full batch to form.
- Throughput rises with concurrency until another resource becomes the limit.

**PagedAttention:**
- vLLM's memory manager for the KV cache.
- Lets many uneven requests share GPU memory without needing one huge contiguous block per request.
- Enables fine-grained allocation and preemption.

**The saturation curve:**
- Throughput rising, TTFT stable → GPU had idle room; batching is helping.
- Throughput flat, TTFT rising → useful knee crossed. This is the operating boundary.
- Waiting requests rising → scheduler or batch limits are binding.
- KV cache near full, preemptions rising → cache pressure is the limit.

**What the knee means:**
- The exact knee is less important than the method: sweep → observe → name the first bottleneck → change one thing.
- Long prompts move the knee earlier (more prefill work and more KV per request).
- Short outputs emphasise request overhead; long outputs emphasise decode.

**This baseline is required for Module 8** — you can't tune responsibly without knowing where the current operating point is.

---

## Module 8: Tune and Evaluate

**What it covers:** A one-hypothesis-at-a-time tuning loop against the FP8 deployment and saturation baseline from Module 7.

**Learning points:**

**The tuning loop:**
1. Measure current deployment (same prompt shape, same concurrency levels as Module 7).
2. Pick one bottleneck from the metrics.
3. Change one flag in `manifests/vllm.yaml`.
4. Apply manifest, wait for rollout.
5. Run same measurement again.
6. Keep, adjust, or reject based on data. Document the decision.

Changing many flags at once hides the cause. One hypothesis at a time keeps the result explainable.

**Key vLLM serving-policy flags:**

| Flag | What it controls | When to raise it |
|---|---|---|
| `--gpu-memory-utilization` | Fraction of VRAM reserved for vLLM | KV cache is tight; more room needed |
| `--max-model-len` | Context length the engine plans around | Can be lowered to free cache for more concurrency |
| `--max-num-seqs` | Max requests in the running batch | Throughput flattening while requests queue |
| `--max-num-batched-tokens` | Token budget per scheduler step | Long prompts waiting too long; TTFT adjustment |

**Reading a result:**
- A higher peak throughput is useful only if p95 latency still meets your bar.
- If throughput rises but latency becomes unacceptable, the operating point is too hot.
- The output of this module is a defensible configuration: model, precision, flag values, workload shape, throughput, latency, quality result.

**Quality must come back:**
- Scheduler and cache flags normally don't change answers — but production changes aren't done until quality is checked.
- Reuse the eval scaffold from Module 5; expand it with workload-specific prompts.

---

## Module 9: Agents on Kubernetes (optional capstone)

**What it covers:** Deploying a real agent as a Kubernetes Deployment in your namespace on top of the tuned inference stack; watching its traffic in the same vLLM metrics.

**Learning points:**

**Where the agent's time actually goes:**
- In a tool-using agent turn (two model calls + one CPU tool call): inference over HTTP is essentially 100% of wall-clock time.
- A CPU tool that runs arithmetic rounds to zero. Even a network-calling tool adds its own latency on top — but you can't tune that here. What you can tune is inference, which dominates.
- This is why tuning the inference layer is tuning the agent.

**Deploying an agent on Kubernetes:**
- An agent is just an HTTP service that calls your vLLM. No CRDs, no custom controllers, no cluster admin needed.
- Objects required: a ConfigMap (code), a Deployment, and a Service — all namespaced.
- Because the agent and vLLM share a namespace, the agent reaches the model by its short Service name (`http://vllm:8000/v1`) with no cross-namespace FQDN.

**Continuous batching is visible from the agent:**
- Six concurrent agent sessions finish in far less than 6× the time of one session.
- The server folds their model turns into one running batch — you see peak `num_requests_running` rise and KV cache usage bump in the metrics.
- This is continuous batching (Module 7) in action, not prefix caching (Module 3), because the prompts differ.

**Timeout hierarchy for production agents:**
- Total request timeout (e.g., 300 s) — bounds the whole call.
- First-token timeout (e.g., 60 s) — catches a stuck prefill on an overloaded server.
- Inter-token timeout (e.g., 30 s) — catches stalled generation under preemption.

**Scaling in production:**
- **Scaling replicas:** tools like KServe or Knative scale inference and agent pods up under load and to zero when idle. Scale-to-zero trades a cold start for paying nothing while idle — the right deal for bursty traffic.
- **Scaling nodes:** a node autoscaler (Karpenter, cluster autoscaler) adds and removes GPU machines underneath. When pods scale to zero, the expensive card goes away too.
- The workshop deliberately ran one dedicated card driven to saturation — the right mode for predictable latency on steady traffic. Scale-to-zero is the other half.

---

## Cross-cutting themes

**The hypothesis-driven method** runs throughout Modules 5–8: measure baseline → make one change → remeasure → keep or reject. Applied to: quantization, speculative decoding, engine saturation, and serving-policy tuning.

**Ownership as a stack**, not just an endpoint: you own the model, the server, the metrics, the tuning, and the agent on top of it. That's what "owning inference" means.

**Everything is open source:** vLLM, the OpenAI-compatible Python client, Kubernetes, and the Qwen models. No vendor lock-in on the inference layer.

**Metrics are live and actionable:** the same `/metrics` endpoint you read in Module 2 to size the KV pool is the one you read in Module 6 to measure prefix cache hits and in Module 9 to watch agent traffic batch. One URL, used all day.
