---
type: concept
tags: [devboxes, control-plane, data-plane, orchestration, trust-boundary, agent-environments, sandbox-platform-engineering, minimal-sandbox, codemode]
updated: 2026-07-05
---

# Agent Environment Architecture

## Definition

The structural separation of an agent system into a **Control Plane** (where the agent and its orchestration logic live) and a **Data Plane** (where workloads execute — devboxes, shells, file systems). The two planes communicate over a network boundary. This split is the foundational answer to the trust and blast-radius problems that arise when agents generate and execute their own code.

## The Problem It Solves

Agent devboxes are not CI environments with a different name. The constraint set is different enough that the design has to be rethought:

| Dimension | CI | Agent devbox |
|-----------|----|--------------|
| Control flow | Fixed, reviewed script | Agent-generated, dynamic |
| Trust model | Internal — operator owns the flow | "Call from inside the house" — agent owns the flow |
| Priority | Fidelity (reproducibility) | TTI (time-to-interactive) |
| Blast radius | Script-bounded | Potentially unbounded if misscoped |

The CI mental model is dangerous for agents because CI executes what a human reviewed. An agent executing its own generated code in a loop is a categorically different threat model — you don't own the control flow, and a bad action has a "big blast-radius problem" (Azzam).

## The Architecture

```
┌─────────────────────────────────────────────────────┐
│  Control Plane                                       │
│  ┌────────┐    ┌──────┐                             │
│  │ Agent  │    │ Tool │                             │
│  └────────┘    └──────┘                             │
└──────────────────────────────────────────────────────┘
                       │ Network
┌──────────────────────────────────────────────────────┐
│  Data Plane                                          │
│                   ┌──────┐   ┌──────┐               │
│                   │ Tool │   │ Tool │               │
│                   └──────┘   └──────┘               │
└──────────────────────────────────────────────────────┘
```

**Control Plane:** the agent process and its direct orchestration tools. Governs provisioning, lifecycle management, and access policy. Relatively small blast radius — what lives here should be limited in scope.

**Data Plane:** the actual workload environments — devboxes, shells, file systems, APIs. Separated from the agent by a network boundary. The agent reaches in via well-defined interfaces (read/write/bash) rather than running directly on the compute it manipulates.

**The network boundary is the trust enforcement point.** An agent that can't directly access its own secrets file (because those secrets live in a sidecar/proxy on the other side of a controlled interface) can't exfiltrate them even if compromised.

## Orchestration Pattern

1. Keep the agent in the control plane
2. Let it provision/manage devboxes (in the data plane) — via an MCP server or equivalent API
3. Let it read/write/bash into those devboxes

The agent treats devboxes as remote resources it requests and manages, not as local state it owns.

## How Different Teams Implement This

### Ramp (on Modal)
- Per-repo environment images defined as code on Modal
- Images rebaked on a 30-min cron (asynchronous — startup never pays the build cost)
- New devbox mounts the warm image, then `git pull` + `uv sync`
- Secrets brokered through proxies/sidecars — never exposed to the agent process

### Uber (DevPods on GKE)
- Balloon pods (pre-provisioned idle capacity) + warm pool + snapshot store
- Pre-indexed code — clone and index happen before the agent asks
- Secrets and MCP tools managed at the platform layer, not inside the agent
- 14K+ DevPods at peak across 5 GKE regions, ~seconds startup

### OpenRouter (one VM per intern)
- One dedicated VM per agent ("intern") — durable state, hard blast-radius isolation
- Secrets masked at the OS level (`systemd InaccessiblePaths`) — intern process cannot open its own credentials file
- Secrets flow only via a privileged sibling process triggered by human approval, not by any chat message to the agent

Despite very different stack choices (Modal managed images, Kubernetes balloon pods, per-agent VMs), all three converge on the same principles:
- Pre-bake/pre-provision; never pay startup cost at task time
- Isolate secrets from the agent process
- The agent manages environments via APIs, not by being co-located with them

## Relationship to Other Concepts

- [DevPods](devpods.md) — Uber's specific implementation of the data-plane workspace concept
- [Guardrails](guardrails.md) — blast-radius limiting at the policy layer; agent environment architecture limits it at the infrastructure layer
- [Agent Skills](agent-skills.md) — skills are what runs inside environments; the environment is the execution substrate

## Open Questions

- At what scale does the "one VM per agent" model (OpenRouter) become more expensive than a shared warm pool (Uber)? OpenRouter has 73 interns; Uber runs 14K+ pods.
- How does the Control Plane agent handle partial failures in the Data Plane — if a devbox becomes unreachable mid-task, does the agent retry, rebuild, or fail?
- The devbox MCP as "headless background agent" pattern (Azzam): what does the ownership model look like — who restarts it if it crashes?

## The Minimal Sandbox Counter-Argument (Colvin / Pydantic)

The talks above focus on how to build a fast, secure full sandbox. Samuel Colvin's talk asks a prior question: **does an agent actually need a full sandbox?**

The "desert" vs. "sandbox" framing: giving an agent a raw Linux shell exposes dozens of binaries (`awk`, `gcc`, `bash`, `ssh`, `apt`, etc.) that a typical embedded agent workflow never needs. The cost of this is not just security surface area — it is also cold-start latency. Full sandbox environments (VM, Container, gVisor, Firecracker) all share a **1–3s boot time**, regardless of whether the task takes minutes (justified) or milliseconds (a 1,000–3,000× overhead ratio).

Colvin's alternative: a **curated tool set** — `sql_query`, `create_chart`, `create_table`, `load_skill` — served by **Pydantic Monty**, a minimal Python interpreter written in Rust, safe by default (no filesystem, env, or network unless explicitly granted), type-checked before execution, and snapshottable.

This does not refute the full-sandbox architecture for heavy tasks (clone a repo, run a coding agent, heavy compilation). It argues that a large class of light/embedded agent tasks doesn't need it — and that paying for a full sandbox for light tasks is a mismatch of problem to solution.

**Synthesis:** the two positions complement rather than contradict. The open question is where the line falls in practice — which real-world workflows are light enough for a Monty-style minimal sandbox vs. heavy enough to warrant a full Linux VM.

## What Actually Isolates the Agent — MicroVM Primitives (Orellana)

Orellana's talk provides the most detailed breakdown of the underlying OS primitives that sandbox controls map onto — framed explicitly as generic ("this is how sandboxes work, not any one product's config"):

**Host layer:** KVM hardware virtualization + microVM manager (VMM) = the hypervisor boundary.

**Guest kernel:** own address space, own syscall surface. The isolation primitives inside it are the actual dials:

| Primitive | What it controls |
|-----------|----------------|
| **Namespaces** (net, mount, PID, user, IPC, UTS) | Network egress + filesystem visibility + process isolation |
| **cgroups v2** (memory.max, cpu.max, pids.max) | Session resources — CPU, memory, process count |
| **seccomp-bpf** | Execution permissions — syscall filter |
| **capabilities** (dropped CAP_* set) | Execution permissions — privilege surface |

**The MCP / transport layer is explicitly NOT a security control** — it's the boundary the agent's calls cross, but it is not an isolation primitive. Yet it is the most prolific source of silent failures (binary data coerced to base64, null bytes truncating downloads, stdout → stderr, exit codes dropped), because teams treat encoding and channel routing as transparent when they're not.

This maps onto a domain-cost table: each control that secures the environment can also silently break domain-specific agent tasks (ML: Numba JIT killed by seccomp; web scraping: bot detection from environment fingerprint; enterprise: pip blocked by network isolation with no error message). See the full table in [1,000 Agent Tasks in a Sandbox](../talks/day3-1425-1000-agent-tasks-sandbox.md).

## Sources

- [Don't Build Agents, Build Environments](../talks/day3-1045-dont-build-agents-build-environments.md) — Adam Azzam (Modal)
- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda (DevPods section)
- [Letting the Interns Loose](../talks/day3-1110-letting-the-interns-loose.md) — Shashank Goyal (one-VM-per-intern isolation, OS-level secrets)
- [Your Agent Needs a Sandbox, Not a Desert](../talks/day3-1205-agent-needs-sandbox-not-desert.md) — Samuel Colvin (Pydantic): the minimal sandbox counter-argument
- [1,000 Agent Tasks in a Sandbox](../talks/day3-1425-1000-agent-tasks-sandbox.md) — Kevin Orellana (Amazon AgentCore): microVM anatomy, primitive-to-control mapping, domain-cost table, transport-layer failure taxonomy
