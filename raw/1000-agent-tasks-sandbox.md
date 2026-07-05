# 1,000 Agent Tasks in a Sandbox: What Breaks When LLMs Write and Run Code

**Speaker:** Kevin Orellana  
**Affiliation:** Amazon AgentCore (AWS)  
**Source:** Slides captured at AIE World Fair 2026

## Slide Photos

10 slides captured. Photos saved in `raw/slides/1000-agent-tasks-sandbox/`.

| File | Slide content |
|------|---------------|
| [IMG_8792.HEIC](slides/1000-agent-tasks-sandbox/IMG_8792.HEIC) | Title slide — "1,000 Agent Tasks in a Sandbox / What breaks when LLMs write and run code in your sandboxes" · Kevin Orellana · Amazon AgentCore |
| [IMG_8793.HEIC](slides/1000-agent-tasks-sandbox/IMG_8793.HEIC) | "So we tested our service the way an agent would" — 4-step methodology overview |
| [IMG_8794.HEIC](slides/1000-agent-tasks-sandbox/IMG_8794.HEIC) | "The harness — an agent testing a sandbox through its own interface" — full architecture diagram (LOCAL/CLOUD, MCP wire, closed loop) |
| [IMG_8795.HEIC](slides/1000-agent-tasks-sandbox/IMG_8795.HEIC) | "Failure type patterns" — Interface Layer vs. Sandbox Layer taxonomy |
| [IMG_8796.HEIC](slides/1000-agent-tasks-sandbox/IMG_8796.HEIC) | "The same controls that define your sandbox define domain-specific success for your agent" — full controls × domain-cost table |
| [IMG_8797.HEIC](slides/1000-agent-tasks-sandbox/IMG_8797.HEIC) | "Anatomy of a sandbox — what's actually isolating your agent" — microVM diagram (namespaces, cgroups, seccomp, capabilities) |
| [IMG_8798.HEIC](slides/1000-agent-tasks-sandbox/IMG_8798.HEIC) | "I'm not the first to hit this — the failure has a name" — Byzantine fault / silent semantic failure / agents overtrust tools / not-hypothetical 2025 incident |
| [IMG_8799.HEIC](slides/1000-agent-tasks-sandbox/IMG_8799.HEIC) | "Why this matters now" — Local ↔ Remote portability gotchas |
| [IMG_8801.HEIC](slides/1000-agent-tasks-sandbox/IMG_8801.HEIC) | "Don't take my word for it — capability lives in the environment" — SWE-bench +64%, HotpotQA 44.8→4.7, 15pt scaffold swing |
| [IMG_8802.HEIC](slides/1000-agent-tasks-sandbox/IMG_8802.HEIC) | Closing slide — "Catch sandbox-specific constraints early using varied domain-specific agent-driven testing" |

_Note: slides captured are the framing/methodology/conclusion slides. The detailed per-failure-mode breakdown (18 failure modes ranked by severity, per the official session description) was not photographed._

---

## Talk overview

AWS ran 1,000 LLM-authored agent tasks against their own cloud sandbox service (AgentCore), using the same MCP interface real customer agents use. The goal: find and fix bugs before customers hit them. The talk walks through the methodology, what broke, and why sandbox environment controls are now a first-class concern for agent reliability.

---

## Methodology — how they set it up (4 steps)

1. **Wrap the sandbox in an MCP server** — The same interface real agents use. Open-sourced on GitHub.
2. **Validate the interface and test harness** — A few seed tasks (run a script, JSON→CSV, scrape a page) to separate interface bugs from service bugs before scaling up.
3. **Let the agent author 1,000 tests** — Organized by domain, stored in a SQLite test plan. The coding agent writes the tests itself.
4. **Smoke test, harden the runner, then go** — Credential refresh, failure triage. About a day or two of setup, then run it unattended.

## Architecture — "the harness"

> An agent testing a sandbox through its own interface. Same MCP wire a customer's agent uses — tested the way it's actually used.

**LOCAL side:**
- Coding agent → writes tests and drives each run
- SQLite test plan (~1,000 tests, organized by domain, agent-authored)
- Hardened runner: cred refresh, pass/fail, triage logic (service-bug ≠ input-bug)

**CLOUD side (isolated environment):**
- Cloud sandbox (hosted service under test)
- Tool calls go over MCP → results stream back → pass/fail

**Key design principle:** Findings are real bugs, triaged and reported, fixed before customers hit them. It's a closed loop: fix the bug → re-run.

---

## Failure type patterns (two layers)

### Interface Layer
- **Complete confidence in the MCP tool (silent)** — agent trusts the tool call succeeded even when it didn't surface an error
- **Protocol misalignments** — e.g. binary encoding issues
- **Input/output validation** — malformed inputs or unexpected output shapes

### Sandbox Layer
- **Compute constraints** — resource limits killing tasks mid-run
- **Permission model misalignments** — agent assumes permissions it doesn't have
- **Sandbox lifecycle management** — state not persisting across calls as expected
- **Sandbox fingerprints** — environment revealing itself as non-human (bot detection, etc.)

---

## Sandbox anatomy — what's actually isolating the agent

Generic microVM primitives (not product-specific):

- **Host:** Hardware-virtualized isolation via KVM + microVM manager (VMM) = the hypervisor boundary
- **Guest kernel (the microVM):** own address space, own syscall surface
  - **Namespaces:** net, mount, PID, user, IPC, UTS
  - **cgroups v2:** memory.max, cpu.max, pids.max
  - **seccomp-bpf:** syscall filter
  - **capabilities:** dropped CAP_* set
  - **Language runtime:** Python, Deno
  - **The agent's code:** what actually runs

**The primitives ARE the dials** (each maps to a control surface):
- Network egress → namespaces (net) + routing/firewall
- Filesystem → namespaces (mount) + perms
- Execution permissions → seccomp + capabilities
- Session/resources → cgroups + timeouts
- Transport (MCP wire) → NOT a control, just the boundary

---

## The controls table — sandbox constraints and their domain costs

| Control | Primitives underneath | What it can cost (domain-specific) |
|---|---|---|
| **Network egress** | net namespace · routing · firewall · socket syscalls · DNS resolves ≠ connect | ML: pip/model pull dies · SWE: git clone, npm/cargo · Enterprise: internal API/DB port blocked |
| **Filesystem** | mount namespace · perms + ownership · tmpfs/overlayfs · exec-view ≠ transfer-view | ML: checkpoint to /tmp → empty download · Web: screenshot dir resets · Doc: PDF "saved," unexportable |
| **Execution permissions** | kernel capabilities · seccomp-bpf · app-layer flags (e.g. Deno --allow-read) | ML: Numba JIT kills the session · fork past pids.max → silent 1-core · Any: JS can't read files or env |
| **Transport (NOT isolation — the wire)** | base64 encoding · null-byte framing · stdout/stderr channels | All domains, ALL SILENT: binary → base64 alphabet · null byte truncates a file · stdout → stderr · exit code dropped |
| **Session/resources** | cgroup mem + CPU · wall-clock/idle timeouts · renderer-per-tab | ML: training killed, checkpoint lost · Web: 3+ tabs → ~28% crash · Any: state you assumed persisted, gone |
| **Environment fingerprint** | automation user-agent · screen ≠ viewport dims · timezone/locale · header order | Web/scraping: target site flags you as a bot and blocks the workflow — everything runs, nothing succeeds |

---

## Silent failure — "the failure has a name"

> When a tool returns wrong data but reports success, the whole field has a word for it.

Key concepts cited:

- **"Every data boundary is a trust boundary"** — You sanitize input, you guardrail output — but your agent blindly trusts every tool result. (Tian Pan, tianpan.co, 2026)
- **Byzantine fault** — A component that fails not by crashing, but by lying: it keeps responding, and the response is wrong in a way that looks correct. (Lamport, Shostak & Pease, ACM, 1982)
- **"Silent semantic failure"** — Output is well-formed and confidently returned, yet wrong — invisible to completion-based monitoring. (Mehta, arXiv, 2026)
- **Agents overtrust tools** — LLMs report a tool's answer even when they'd have been right without it; they don't detect silent tool errors. ("Tools Fail," CMU, EMNLP 2024)

**NOT HYPOTHETICAL (slide explicitly called this out):**
> 2025: an AI coding agent deleted a live production database during an explicit code freeze — then fabricated data and reported the run as a success. Fabricated results + false success reporting. (publicly documented, 2025)

**Orellana's framing:** "All of it shares one signature: the call says success. The bytes don't."

---

## Why this matters now — Local ↔ Remote portability gotchas

| On your laptop | In a sandbox |
|---|---|
| Your codebase, a local DB, a dev server, your /tmp, your network. One machine — every assumption holds. | The /tmp you wrote to may not exist. Internal endpoints unreachable. The agent doesn't know — it just quietly comes up short. |

The gap between local dev and remote sandbox is where silent failures live.

---

## Evidence — capability lives in the environment

**SWE-agent (same model, GPT-4 Turbo, change only the interface):**
- Basic shell/editor: 11.0% on SWE-bench Lite
- Agent-built interface: 18.0% → **+64% more tasks solved**
- Source: Yang et al., "SWE-agent," NeurIPS 2024

**Corrupted tool environment (same agent, same prompt):**
- F1 on HotpotQA: 44.8 (good env) → 4.7 (corrupted tool environment)
- Source: "Don't Blindly Trust It," arXiv 2026

**Harness, not model:**
- Independent monitoring, same model → up to 15 pts benchmark swing from scaffold alone
- Source: "Stop Comparing LLM Agents Without Disclosing the Harness," arXiv 2026

**Key quote from slide:**
> "The unit of performance is no longer just the model." — UC Berkeley, "Software Harness Design," 2026

**Orellana's framing:** "The same agent is brilliant or crippled depending on the sandbox around it — and it can't see the difference. Neither can you, unless you test for it."

---

## Takeaway / closing slide

> **Catch sandbox-specific constraints early using varied domain-specific agent-driven testing.**
>
> Point an agent at your remote environment. Give it a wide set of tasks from your domain. Watch where it hits the rough edges — you'll likely find some before your customers do.
