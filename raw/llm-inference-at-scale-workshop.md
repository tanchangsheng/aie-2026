# LLM Inference at Scale — With First Principles

**AIE Workshop 2026 · 2 Hours · Live Demos**  
*From the memory equation to production engines*  
Repo: [github.com/harshuljain13/llm-inference-at-scale](https://github.com/harshuljain13/llm-inference-at-scale)

---

## Speakers

**Harshul Jain** — Sr. Software Engineer, Audible Inc. Building ML/AI platforms, LLM Inference Handbook.

**Tanmay Sah, PhD** — Senior Quantitative Modeler. AI researcher building agent verifiers and world models.

---

## Agenda

| # | Section | Topics |
|---|---------|--------|
| 00 | Problem Statement | Pain points, metrics |
| 01 | Foundations | Attention mechanisms + internals |
| 02 | Model Optimizations | Quantization |
| 03 | KV Cache, Serving & Engines | Serving optimizations |
| 04 | Other Inference Optimizations | SGLang, TensorRT, Nvidia Dynamo |
| 05 | Putting It Together | Compounding effects |

---

## 00 · Problem Statement

### What Is LLM Inference?
Every interaction with AI is an inference call:
- **52%** of new code is AI-authored; 1 in 7 PRs involve an agent; 84–91% of devs use AI tools
- **$23B** inference-as-a-service market in 2026 (from $18.6B in '25); cost per 1M tokens fell 1,200× — but volume grew faster
- **$72K** cost per year for a 25-person team on Opus; one 50-turn coding session ≈ 1M tokens

### Why It Matters
- Replacing Google search queries with LLMs is a **$36B profit drain**
- At scale, **inference cost far exceeds training cost**
- Query cost must drop below 0.5¢ to keep search profitable
- C-suites are forcing teams to audit and budget token spend — "Token Hunger Games"

### Training vs. Inference Cost (OpenAI)
- **Training GPT-3**: one-time $4.6M
- **Inference (serving)**: every second, growing — **> $4.6M / week** and rising
- Training is a one-time capital cost. Inference is an operating cost that scales with every user, every token, every session.

### The Field Moves Quickly
| 2023 | 2024 | 2025 | 2026 |
|------|------|------|------|
| vLLM — PagedAttention ships | SGLang — Radix Attention | DeepSeek-V3 — MLA goes mainstream | Nvidia Dynamo · M* — Distributed KV, new attention |

> You will not memorize these. You will learn the foundations that let you evaluate whatever ships next.

### Demo A: Feel the Pain
`demos/demo_a_feel_the_pain.ipynb` — Load Mistral-7B on a real GPU. Measure what the API never shows you:
- Memory compounds with static weights + new context + users
- Slow TTFT that scales linearly to input text
- One user at a time limitation

---

## 01 · Foundations

> One memory equation governs how many users fit, which GPU to buy, and what a token costs.

### The Inference Pipeline (Mistral 7B)
```
Input text → Tokenizer → Embedding (128K × 4096) → 32 Transformer layers → LM head (4096 × 128K) → Next token
```
- **95% of compute + memory** happens in the transformer layers
- This runs once per token — 100 tokens out = the whole pipeline 100 times over

### One Transformer Layer
```
Input hidden state → [RMS Norm + Attention (Q·K·V, reads KV cache)] → [Feed-forward: 2 big matmuls, 42% of weights] → Output
```
- Attention decides what to look at; the MLP stores the knowledge

### Attention: Scaled Dot-Product
```
out = softmax( Q · Kᵀ / √d ) · V
```
| Step | Description |
|------|-------------|
| Project | Every token becomes Q, K, V |
| Score | Query × every past Key → relevance score, softmax → weights |
| Mix | Weighted sum of Values — the attention output |

- Each new token must read back the K and V of every previous token → **KV Cache**

### KV Cache Growth (Mistral 7B)

**KV per token formula:**
```
2 (K + V) × 32 layers × 8 KV heads × 128 dim × 2 bytes = 131,072 bytes = 131 KB per token per user
```

| Scenario | KV Memory |
|----------|-----------|
| 1 user · 4K context | 524 MB |
| 1 user · 16K context | 2.1 GB |
| 80 users · 4K context | 42 GB |
| 80 users · 16K context | 168 GB — OOM |

> Weights are fixed at 14.5 GB. The KV cache is the variable that kills you.

### GPU Memory Budget (A100 · 80 GB)
| Component | Size | Type |
|-----------|------|------|
| Weights | 14.5 GB | Fixed |
| KV Memory | variable — grows with users | Variable |
| Overhead | ~8 GB | Fixed |

KV cache = whatever's left — and it decides how many users fit.

### Two Phases of Inference

| Phase | Type | Characteristics |
|-------|------|-----------------|
| **Prefill** | parallel · compute-bound · matrix × matrix | N input tokens all at once; ~100–400 FLOPs/byte |
| **Decode** | sequential · memory-bound · matrix × vector | One token at a time; ~1–2 FLOPs/byte |

> Almost every optimization in this workshop targets decode. That's where the time and money go.

### Prefill + Decode Timeline
- **Prefill**: one wide bar — whole prompt in parallel. Sets TTFT ≈ 50 ms.
- **Decode**: the staircase — one token per step, ITL ≈ 10 ms apart — each waits on memory.

### The Bandwidth Wall
- Model weights (14.5 GB) live in HBM (80 GB, 2 TB/s)
- Every decode step streams 14.5 GB from HBM to chip
- **Decode ceiling: 14.5 GB ÷ 2 TB/s = 7.25 ms/token ≈ 138 tok/s**
- The model is 700× larger than all SRAM combined — cannot fit on-chip
- **Decode is memory-bandwidth bound, not compute bound**

### The Roofline
- Prefill: ~156 FLOP/byte — GPU is saturated
- Decode: ~2 FLOP/byte — deeply memory-bound
- You cannot buy your way out with a faster chip. You raise arithmetic intensity — mostly by batching.

### Capacity Equation (Mistral 7B on A100-80)
```
available = 80 - 14.5 - 8 = 57.5 GB
max_users = 57.5 GB ÷ (0.131 MB × 4096) = 107 concurrent users
```
(without any optimizations)

### Latency SLO
- **TTFT** < 500 ms (time to first token)
- **ITL** < 50 ms (inter-token latency)

```
ITL = (model_bytes + batch × kv_bytes_per_user) / mem_bandwidth
throughput = batch × (1000 / ITL_ms)
```

Higher batch → higher throughput but higher latency. The SLO is the ceiling.

### Trade-off Triangle: Pick Two
| Combination | Use Case |
|-------------|----------|
| Quality + Latency | Premium chat |
| Quality + Throughput | Async agent workload |
| Latency + Throughput | Batch / Offline |

### GPU Selection (Mistral 7B)
| GPU | VRAM | Max Users | $/hr | $/M tokens |
|-----|------|-----------|------|------------|
| A100-40 | 40 GB | 32 | $2.50 | $0.45 |
| A100-80 | 80 GB | 107 | $3.50 | $0.28 |
| H100-80 | 80 GB | 107 | $4.00 | $0.16 |
| H200-141 | 141 GB | 237 | $5.50 | $0.13 |
| B200-192 | 192 GB | 324 | $7.00 | $0.09 |

> The most expensive GPU per hour is the cheapest per token.

---

## 02 · Model Optimizations

> Three changes to attention shrink the KV cache — one you already have, one already on by default, one that's the future.

### Weight Quantization (Mistral 7B)
| Precision | Size | Freed for KV Cache |
|-----------|------|-------------------|
| FP16 | 14.5 GB | baseline |
| INT8 | 7.2 GB | +7.2 GB freed |
| INT4 / NF4 | 3.6 GB | +10.9 GB freed |

Minor quality loss at INT4 on reasoning tasks.

### GQA — Grouped Query Attention (Already Free)
| Design | Queries | Keys + Values |
|--------|---------|---------------|
| MHA (multi-head) | 8 | 8 (one KV per query head) |
| GQA-8 (grouped) | 8 | 2 (groups share one KV) |

- **4× KV reduction** — 32 KV heads → 8 KV heads
- No measurable quality loss
- Mistral, Llama, Qwen — all ship GQA. This is your baseline, not an optimization.

### MLA — Multi Latent Attention (DeepSeek V3)
Why DeepSeek-V3 is cheap to serve — dramatic compression of the KV cache.

### FlashAttention
- Bit-exact output — same softmax, reordered
- Up to **6× faster**, default in every engine
- No accuracy tradeoff. It's already on.

### KV Memory Comparison
| Mechanism | Memory per Token |
|-----------|-----------------|
| MHA | 524 KB |
| GQA-8 | 131 KB · **4×** |
| MLA | 9.3 KB · **56×** |

### Attention Mechanism Scorecard
| Mechanism | KV/Token | Quality | Verdict |
|-----------|----------|---------|---------|
| MHA | 524 KB · 1× | Reference | Memory-bound |
| GQA-8 | 131 KB · 4× | ~MHA | ~free 4× win |
| MQA | ~16 KB · 32× | Dips | Max KV cut |
| MLA | ~36 KB · 14× | Near-MHA | Pareto winner (DeepSeek) |
| Sliding window | constant | Long ctx | Weak retrieval |
| Linear attn | constant | O(N) | Recall bottleneck |
| Mamba | constant | Selective SSM | Linear time |

> MLA breaks the pattern — near-MHA quality at 14× less KV.

---

## 03 · KV Cache, Serving & Engines

> Model is smaller — GQA + INT4. Now optimize serving.

### Why the KV Cache Exists
| Approach | Complexity | Tradeoff |
|----------|-----------|---------|
| Without cache | O(n²) — recompute all tokens every step | No memory needed |
| With cache | O(n) — write once, reuse | ~131 KB per token per user |

### The KV Cache Toolbox — Four Levers
| Lever | Technique | Gain |
|-------|-----------|------|
| 01 | Paged Attention | ~4× users |
| 02 | Continuous Batching | ~10× context reclaim |
| 03 | Prefix Caching | ~20× TTFT improvement |
| 04 | KV Quantization | ~2× users |

These are orthogonal — stack all four for ~8× users on the same GPU.

### Lever 01: PagedAttention
- Without paging: fixed 2048-slot blocks → ~80% wasted to fragmentation
- With paging: small blocks on demand → 95%+ utilization, ~4× concurrent users
- A block table maps each request's logical blocks to physical pages anywhere in the pool

### Lever 02: Continuous Batching
- No GPU slots remain idle
- Refills freed slots every iteration
- Source: Orca (Yu et al., OSDI '22)

### Lever 03: Prefix Caching
- First user pays. Everyone else is free.
- Each multi-turn conversation reuses the cached full prior context; only the new query is computed
- Across 100 users sharing a system prompt: **up to 15× faster TTFT**
- Problem: static prefix caching — differ one byte early, lose the whole cache (hash of full prefix)

### Lever 04: KV Quantization
- FP16 → FP8: **2× more users**, quality loss < 0.1%
- KV values are less sensitive than weights
- vLLM flag: `--kv-cache-dtype fp8`

### The Engine: vLLM
```
API Server → Scheduler → KV Block Manager → GPU Workers
```
- OpenAI-compatible endpoint
- Continuous batching (refills freed slots every iteration)
- PagedAttention (paged KV blocks, near-zero waste)
- Fused attention kernels
- **24× throughput** over HuggingFace baseline
- Start with one command: `vllm serve mistralai/Mistral-7B-v0.1`

---

## 04 · Other Inference Optimizations

### Lever 05: Speculative Decoding
1. Small 1B model drafts 5 tokens
2. Large 70B model verifies all 5 in a single forward pass
3. Accept correct tokens, replace only misses
4. Output quality is **identical** to the large model alone

### SGLang: RadixAttention
- Problem with static prefix caching: if prompts differ at byte 3, the full shared context is recomputed
- SGLang solution: **tree-based prefix matching** — stores shared prefix at the root; branches reuse it
- **5× cache hit rate vs vLLM** for agentic workloads
- Detects shared system prompts automatically

### TensorRT-LLM
- Compile once (offline): fuse + select kernels for your specific GPU — takes minutes
- Runtime: CUDA graphs, XQA, FP8 — zero kernel-launch overhead
- **2× throughput vs vLLM**
- Tradeoff: compile step required, NVIDIA-only

### Benchmark: vLLM 0.14.1 vs SGLang 0.5.13
*Qwen2.5-7B · fp16 H100 80GB PCIe · 20 concurrent · 1,000 ShareGPT reqs*

**Standard API (independent requests):**
| Metric | vLLM | SGLang |
|--------|------|--------|
| Requests/sec | 29.79 | 29.03 |
| Tokens/sec | 7,453 | 7,279 |
| Avg TTFT | 0.51 s | 0.52 s |
| P99 latency | 3.61 s | 3.97 s |

Near-parity — vLLM slightly ahead.

**Agentic shared-prefix (~1k-token prefix, 3 branches):**
| Engine | TTFT |
|--------|------|
| vLLM · default | 6.30 s |
| vLLM · prefix cache | 4.80 s |
| SGLang · native | 1.07 s |

SGLang: **4.4× vs tuned vLLM · 5.8× vs default**.

### Engine Decision Matrix
| Use Case | Engine | Why |
|----------|--------|-----|
| General production | vLLM | Broadest support, easy setup |
| Agents + multi-turn | SGLang | RadixAttention, constrained decode |
| Max throughput (NVIDIA) | TensorRT-LLM | AOT compile, CUDA graphs, FP8 |
| Agentic session routing | NVIDIA Dynamo | KV-aware load balancing |
| Prototyping | HuggingFace | Simple, no server |

> Most teams start at vLLM and move only when a workload demands it.

---

## 05 · Putting It Together

### The Compounding Effect
| Stack | Multiplier |
|-------|-----------|
| Baseline | 1× |
| + GQA | 4× |
| + INT4 | 7× |
| + Paged Attention | 10× |
| + Continuous Batching | 13× |
| + Prefix Cache | 16× |
| + Speculative Decode | 20× |

**No new hardware.** Each lever multiplies the last.

---

## Where to Go From Here

### Key Papers
- FlashAttention 1 & 2
- PagedAttention / vLLM
- Speculative Decoding
- Alt attention mechanisms: Linear, Log-Linear

### Engines to Explore
- [vllm-project/vllm](https://github.com/vllm-project/vllm)
- [sgl-project/sglang](https://github.com/sgl-project/sglang)
- TensorRT-LLM
- NVIDIA Dynamo

### Advanced Topics
- KV eviction: H2O, SnapKV
- Cache compression: KIVI, KVQuant, PALU, LMCache
- Hybrid memory: InfiniGen, LayerKV
- Disaggregated prefill/decode
- Distributed multi-node inference
- Multimodal Serving — M*

---

## Resources

- **Repo**: [github.com/harshuljain13/llm-inference-at-scale](https://github.com/harshuljain13/llm-inference-at-scale) — 55+ modules · 12 chapters · 8 runnable demo notebooks
- **Molab**: [molab.marimo.io](https://molab.marimo.io/notebooks)
- **Book**: Manning In Action · 2027
