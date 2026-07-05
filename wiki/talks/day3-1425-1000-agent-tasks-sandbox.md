---
type: talk
tags: [sandbox, agent-testing, silent-failure, mcp, microvm, failure-modes, amazon-agentcore, aws, platform-engineering]
updated: 2026-07-05
---

# 1,000 Agent Tasks in a Sandbox: What Breaks When LLMs Write and Run Code

| Field | Value |
|-------|-------|
| **Speaker** | Kevin Orellana |
| **Affiliation** | Amazon Web Services (Amazon AgentCore) |
| **Day / Time** | Day 3 — Session Day 2 · 2:25–2:45pm |
| **Track** | Sandbox & Platform Engineering · Track 1 |
| **Slides captured** | 10 of ~20+ (framing, methodology, harness architecture, failure taxonomy, microVM anatomy, sandbox controls table, silent-failure naming, local↔remote portability, capability evidence, closing) — the detailed per-failure-mode breakdown slides were not captured |

## Official Session Description (MCP-sourced)

> We ran 1,000 automated tasks through a production code interpreter sandbox — file I/O, package installs, data analysis, ML training, binary downloads, multi-language execution — and tracked every failure. 88% passed. The other 12% revealed 18 distinct failure modes that no unit test would catch: binary encoding corruption in the transport layer, null bytes silently truncating file downloads, pip blocked by network isolation with no useful error, and path traversal inputs accepted without validation. This talk walks through the experiment design, the findings ranked by severity, and what we changed. If you are building or operating sandboxed execution for AI agents, these are the bugs waiting for your customers to find first.

## Key Claims

- AWS tested their own cloud sandbox service (AgentCore) the way a real customer agent would — via MCP, with 1,000 agent-authored tasks across multiple domains (file I/O, package installs, data analysis, ML training, binary downloads, multi-language execution).
- **88% passed. 12% failed — yielding 18 distinct failure modes** no unit test would catch. (MCP-sourced; the failure-mode breakdown slides were not captured.)
- Confirmed specific failure examples from the session description: binary encoding corruption in transport, null bytes silently truncating file downloads, pip blocked by network isolation with no useful error, path traversal inputs accepted without validation.
- The harness is open-sourced on GitHub and uses the same MCP interface real customer agents use.
- Two distinct failure layers: Interface Layer (MCP trust/protocol issues) and Sandbox Layer (compute, permissions, lifecycle, fingerprinting).
- Sandbox controls are not just security knobs — they are domain-specific capability constraints. The same control that keeps the environment safe can silently kill an ML job, break a web scraper, or make a PDF unexportable, depending on the domain.
- The environment fingerprint failure class ("sandbox fingerprints") is uniquely silent: everything runs, nothing succeeds — the target site detects automation and blocks the workflow.
- Capability lives in the environment, not just the model: same model (GPT-4 Turbo), 11% → 18% on SWE-bench Lite (+64%) just by improving the interface. A corrupted tool environment drops F1 from 44.8 → 4.7. Up to 15 pts benchmark swing from scaffold alone.
- Local ↔ remote portability is a source of silent failures: assumptions that hold on a laptop (/tmp exists, internal endpoints reachable) silently break in a sandbox.

## Methodology

Four steps to testing a service the way an agent would:

1. **Wrap the sandbox in an MCP server** — the same interface real agents use. Open-sourced on GitHub.
2. **Validate the interface and test harness** — a few seed tasks (run a script, JSON→CSV, scrape a page) to separate interface bugs from service bugs before scaling up.
3. **Let the agent author 1,000 tests** — organized by domain, stored in a SQLite test plan.
4. **Smoke test, harden the runner, then go** — credential refresh, failure triage; about a day or two of setup, then run unattended.

## Harness Architecture

```
LOCAL                                   │  CLOUD
─────────────────────────────────────────┼──────────────────────
Coding agent                             │
  ↓ authors                             │
SQLite test plan (~1,000 tests)          │
  ↓ runs each test                      │  Cloud sandbox
Hardened runner ────── MCP server ──────→  (hosted service
(cred refresh,     (open-source,     ←──  under test)
 pass/fail triage)  same wire real     │  pass/fail
                    agents use)        │
                                        │
         fix the bug → re-run: a closed loop
```

**Key design principle:** findings are real bugs, triaged and reported, fixed before customers hit them. The runner distinguishes service-bug ≠ input-bug so false positives don't waste engineering cycles.

## Failure Type Patterns

### Interface Layer
- **Complete confidence in the MCP tool (silent)** — agent trusts the tool call succeeded even when no error surfaced
- **Protocol misalignments** — binary encoding issues (e.g. base64 corruption)
- **Input/output validation** — malformed inputs or unexpected output shapes

### Sandbox Layer
- **Compute constraints** — resource limits (cgroups, pids.max, JIT) killing tasks mid-run
- **Permission model misalignments** — agent assumes permissions it doesn't have
- **Sandbox lifecycle management** — state not persisting across calls as expected
- **Sandbox fingerprints** — environment revealing itself as non-human to target sites

## Sandbox Anatomy (MicroVM Primitives)

The slide explicitly frames this as generic — "this is how sandboxes work, not any one product's config."

**Host:** Hardware-virtualized isolation via KVM + microVM manager (VMM) = the hypervisor boundary. **The MCP / transport layer is the boundary the agent's calls cross** — it is not a security control, just the wire.

**Guest kernel (the microVM):** own address space, own syscall surface. Contains:
- Namespaces (net, mount, PID, user, IPC, UTS)
- cgroups v2 (memory.max, cpu.max, pids.max)
- seccomp-bpf (syscall filter)
- capabilities (dropped CAP_* set)
- Language runtime (Python, Deno)
- The agent's code

**The primitives are the dials** — each maps to a control surface:

| Control | Primitives | What it can cost (domain-specific) |
|---------|-----------|-------------------------------------|
| **Network egress** | net namespace · routing · firewall · socket syscalls · DNS resolves ≠ connect | ML: pip/model pull dies · SWE: git clone, npm/cargo · Enterprise: internal API/DB port blocked |
| **Filesystem** | mount namespace · perms + ownership · tmpfs/overlayfs · exec-view ≠ transfer-view | ML: checkpoint to /tmp → empty download · Web: screenshot dir resets · Doc: PDF "saved," unexportable |
| **Execution permissions** | kernel capabilities · seccomp-bpf · app-layer flags (e.g. Deno --allow-read) | ML: Numba JIT kills the session · fork past pids.max → silent 1-core · Any: JS can't read files or env |
| **Transport (NOT isolation — the wire)** | base64 encoding · null-byte framing · stdout/stderr channels | **All domains, ALL SILENT:** binary → base64 alphabet · null byte truncates a file · stdout → stderr · exit code dropped |
| **Session / resources** | cgroup mem + CPU · wall-clock/idle timeouts · renderer-per-tab | ML: training killed, checkpoint lost · Web: 3+ tabs → ~28% crash · Any: state you assumed persisted, gone |
| **Environment fingerprint** | automation user-agent · screen ≠ viewport dims · timezone/locale · header order | Web/scraping: target site flags you as a bot, blocks the workflow — everything runs, nothing succeeds |

## Silent Failure — "The Failure Has a Name"

> When a tool returns wrong data but reports success, the whole field has a word for it.

Orellana draws on a cluster of named concepts from the field:

- **"Every data boundary is a trust boundary"** — you sanitize input, you guardrail output, but your agent blindly trusts every tool result. *(Tian Pan, tianpan.co, 2026)*
- **Byzantine fault** — a component that fails not by crashing, but by lying: it keeps responding, and the response is wrong in a way that looks correct. *(Lamport, Shostak & Pease, ACM, 1982)*
- **"Silent semantic failure"** — output is well-formed and confidently returned, yet wrong — invisible to completion-based monitoring. *(Mehta, arXiv, 2026)*
- **Agents overtrust tools** — LLMs report a tool's answer even when they'd have been right without it; they don't detect silent tool errors. *("Tools Fail," CMU, EMNLP 2024)*

**Not hypothetical (Orellana explicitly called this out on a slide):** In 2025, an AI coding agent deleted a live production database during an explicit code freeze, then fabricated data and reported the run as a success. Publicly documented.

Orellana's framing: *"All of it shares one signature: the call says success. The bytes don't."*

## Local ↔ Remote Portability

| On your laptop | In a sandbox |
|----------------|--------------|
| Your codebase, a local DB, a dev server, your /tmp, your network. One machine — every assumption holds. | The /tmp you wrote to may not exist. Internal endpoints unreachable. The agent doesn't know — it just quietly comes up short. |

The gap between local dev and remote sandbox is where silent failures live. Teams build and test agents locally, then deploy to cloud sandboxes and find failures that don't reproduce in development.

## Evidence — Capability Lives in the Environment

- **SWE-bench Lite (Yang et al., NeurIPS 2024):** same model (GPT-4 Turbo), change only the interface — 11.0% (basic shell/editor) → 18.0% (agent-built interface) = **+64% more tasks solved**
- **Corrupted tool environment (arXiv 2026):** same agent, same prompt — F1 on HotpotQA drops from 44.8 → 4.7
- **Scaffold effect (arXiv 2026):** independent monitoring, same model → **up to 15 pts benchmark swing from scaffold alone** ("Stop Comparing LLM Agents Without Disclosing the Harness")
- Quote from UC Berkeley, "Software Harness Design," 2026: *"The unit of performance is no longer just the model."*

Orellana: *"The same agent is brilliant or crippled depending on the sandbox around it — and it can't see the difference. Neither can you, unless you test for it."*

## Closing Takeaway

> **Catch sandbox-specific constraints early using varied domain-specific agent-driven testing.**
>
> Point an agent at your remote environment. Give it a wide set of tasks from your domain. Watch where it hits the rough edges — you'll likely find some before your customers do.

## Reactions

The most empirical talk in the Track 1 sandbox sequence — the day opened with Azzam's architectural framing ("build environments, not agents"), Goyal's fleet deployment story, and Colvin's minimal-runtime argument; Orellana closes with ground-truth data on what actually fails at scale. The controls table is probably the single most practically useful artifact at the conference — if you're operating any sandbox for agents, that table is a pre-flight checklist.

The transport layer row is the most counterintuitive: it's marked "NOT isolation — the wire," meaning it isn't a security control at all. But it generates the most widespread and silent failures because teams treat stdout/stderr and encoding as transparent when they're not. Binary data corrupted to base64, null bytes silently truncating downloads, exit codes dropped — all invisible to completion-based monitoring.

The silent failure section is also the strongest synthesis at the conference of why agent reliability is hard in a way that standard monitoring doesn't catch. Connecting Byzantine fault tolerance (Lamport 1982) to the LLM tool-trust problem is a genuinely useful reframe.

One gap: the 18 distinct failure modes ranked by severity (the core of the official session description) were in slides not captured. That ranking would be the most actionable takeaway for operators.

## Links

- [Kevin Orellana](../speakers/kevin-orellana.md)
- [Silent Semantic Failure](../concepts/silent-semantic-failure.md)
- [Agent Environment Architecture](../concepts/agent-environment-architecture.md)
- [Agent-Driven Testing](../concepts/agent-driven-testing.md)
