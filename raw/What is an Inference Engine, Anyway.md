---
title: "What is an Inference Engine, Anyway?"
speaker: Charles Frye
affiliation: Modal (Member of Technical Staff)
track: Workshops Day 1
day: Day 1 — Workshop Day
time: 11:05am–12:05pm
room: Track 8
speaker_twitter: https://x.com/charles_irl
speaker_linkedin: https://www.linkedin.com/in/charles-frye-38654abb/
speaker_website: https://charlesfrye.github.io
---

Why companies focus more on LLM Inference than LLM training? Because training is a cost centre but inference is a revenue centre

Inference servers wrap inference engines and provide a way to serve models for inference. They handle requests, manage resources, and provide APIs for clients to interact with the models.

VLLM contains both inference server and engine but it's more of an engine with a simple server.

llm-d is an inference server, can work on top of vllm or other inference engines. It does orchestration and optimizations above model servers to serve high-scale real-world traffic efficiently and reliably, organized into four core themes:
- Intelligent Routing: Maximize performance with prefix-cache and load-aware balancing, including experimental predicted latency-based scheduling to decrease latency and increase throughput.
- Advanced KV-Cache Management: Increase the effective "working set size" for multi-turn requests with tiered offloading to CPU or disk and precise global indexing of the KV cache state.
- Serving Large Models: Optimize massive models (e.g., DeepSeek-R1, GPT-OSS) using prefill/decode disaggregation and wide expert-parallelism over fast accelerator interconnects.
- Operational Excellence: Ensure production stability with intelligent flow control for multi-tenant serving and proactive, SLO-aware autoscaling based on real-time inference signals.
- Batch Processing: Efficiently manage large-scale offline inference with OpenAI-compatible Batch APIs and asynchronous processing to maximize hardware utilization.

Inference engine is the core component that performs the actual inference computations. It takes input data, processes it through the model, and produces output predictions. The inference engine is optimized for performance and can handle various model architectures and data types.

It is stateful when we care about performance, e.g. there is KV cache. It is stateless by default when we run vllm vanilla but the performance will not be very good.

