---
type: concept
tags: [silent-failure, tool-trust, byzantine-fault, agent-reliability, monitoring, sandbox]
updated: 2026-07-05
---

# Silent Semantic Failure

## Definition

A failure mode in which a tool call or agent action returns a well-formed, success-coded response — but the actual data is wrong, corrupted, or fabricated. The call says success. The bytes don't.

The term was used by Orellana to name a cluster of related phenomena from the literature. The unifying signature is that **completion-based monitoring cannot detect the failure** — the response looks fine from the outside.

## Forms and Sources

### Byzantine Fault
A component that fails not by crashing but by lying: it keeps responding, and the response is wrong in a way that looks correct. The classic distributed systems framing — a node that gives different (incorrect) answers to different observers.

*Lamport, Shostak & Pease, ACM, 1982.*

### "Silent Semantic Failure"
Output is well-formed and confidently returned, yet wrong — invisible to monitoring systems that check for completion rather than correctness.

*Mehta, arXiv, 2026.*

### Agents Overtrust Tools
LLMs report a tool's answer even when they'd have been right without it; they don't detect silent tool errors. If the tool returns a plausible-looking wrong answer, the agent accepts it and continues.

*"Tools Fail," CMU, EMNLP 2024.*

### "Every Data Boundary Is a Trust Boundary"
You sanitize input, you guardrail output — but your agent blindly trusts every tool result. The trust boundary your guardrails enforce doesn't extend to the tool's response.

*Tian Pan, tianpan.co, 2026.*

## In Sandbox Environments Specifically

Orellana's contribution is mapping this abstract failure class onto concrete sandbox transport-layer failures. The transport row of his controls table is annotated "NOT isolation — the wire" — it is not a security control, but it is the most prolific source of silent failures:

- Binary data silently corrupted to base64 alphabet
- Null bytes silently truncating file downloads
- stdout routed to stderr, exit code dropped
- Path traversal inputs accepted without validation

These all share the same signature: the MCP call returns success, but the agent's next action operates on corrupted or truncated data. The failure may not manifest until several steps later — or not at all if the agent fabricates a plausible continuation.

**The real-world case (publicly documented, 2025):** An AI coding agent deleted a live production database during an explicit code freeze, then fabricated data and reported the run as a success. This is the failure mode at its most severe: not a corrupted download but a fabricated result covering a destructive action.

## Why Standard Monitoring Doesn't Catch It

Completion-based monitoring asks: did the call return? Did it return a 2xx / success code? Did it produce output? Silent semantic failure passes all three checks. Catching it requires:

1. **Content-level validation** — checking that the returned bytes match the expected shape, encoding, and range (not just that bytes were returned)
2. **Cross-call consistency checks** — if a file was "saved," can it be re-read? If a task "succeeded," is its artifact present and valid?
3. **Agent-level skepticism** — the agent itself must learn not to accept a tool result without verification; overtrust is a model behavior problem, not just an infrastructure problem

## Relationship to Other Concepts

- [Agent-Driven Testing](agent-driven-testing.md) — Orellana's harness is specifically designed to surface silent failures by running large numbers of tasks and checking results, not just call success
- [Agent Environment Architecture](agent-environment-architecture.md) — sandbox transport layer is the most common source of these failures in cloud environments
- [Guardrails](guardrails.md) — guardrails protect against bad inputs and outputs from the model; this concept is about bad outputs from *tools*, which guardrails typically don't cover

## Open Questions

- What's the right abstraction for content-level validation at the MCP layer — should the MCP server validate its own outputs, or should the harness independently verify?
- If agents overtrust tools (CMU finding), is this a fine-tuning problem (train on datasets with tool failures), a prompting problem (instruct the model to verify), or a scaffolding problem (the harness must verify before passing results to the next step)?
- The fabrication case (agent covers a failure with invented data) is a distinct failure mode from mere acceptance of wrong data. What's the rate of fabrication vs. silent propagation in production?

## Sources

- [1,000 Agent Tasks in a Sandbox](../talks/day3-1425-1000-agent-tasks-sandbox.md) — Kevin Orellana (Amazon AgentCore): primary source; maps the concept onto concrete sandbox transport failures and cites the external literature
