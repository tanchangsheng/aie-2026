---
type: talk
tags: [devboxes, agent-environments, control-plane, data-plane, orchestration, trust-boundary, modal, ramp, fastmcp, prefect, sandbox-platform-engineering]
updated: 2026-07-05
---

# Don't Build Agents, Build Environments

| Field | Value |
|-------|-------|
| Speaker | [Adam Azzam](../speakers/adam-azzam.md) |
| Affiliation | Modal (formerly Prefect / FastMCP maintainer) |
| Date | 2026-07-05 |
| Time | 10:45–11:05am |
| Track | Sandbox & Platform Engineering |
| Room | Track 1 |
| Day | Day 3 — Session Day 2 |
| Slides coverage | 9 of 9 slides captured (full deck) |

---

## Summary

Adam Azzam argues that the hard part of building coding agents has moved: the agent loop is solved; the environment around it is not. Two problems account for most of the reinvented wheels he sees: devboxes (how do you give an agent a fast, reproducible, correctly-scoped workspace?) and orchestration (how does an agent provision and work inside those workspaces?). He uses Ramp's Modal-based devbox system as a detailed case study, then closes with a Control Plane / Data Plane architecture diagram and actionable advice for teams building their own.

---

## Key Claims

- **The agent loop is a solved problem; the environment is not.** Background agents are converging on a universal system architecture — the differentiation has moved to the infrastructure around the agent.
- **Devboxes are not a new idea, just a new constraint set.** Traditional CI environments exist for the same reason, but the priorities are inverted: CI favors fidelity over TTI (time-to-interactive); agent devboxes must favor TTI. The trust boundary is also fundamentally different.
- **The trust boundary is the critical new problem.** CI executes a fixed, reviewed script — you own the control flow. An agent generates its own code and executes it in a loop. This is "the call is coming from inside the house" — a totally different threat model with a big blast-radius problem.
- **Ramp's solution on Modal: warm image caching + secrets via sidecars.** Defined per-repo images in code on Modal, rebaked every 30 minutes on a cron. New devboxes mount the warm image (then git pull + uv sync). Secrets are never exposed to the agent — brokered through proxies/sidecars.
- **The payoff: nearly instant TTI + isolated secrets.** Image builds happen asynchronously; boot is nearly instant. Permissions and ports are tunable per repo. Environment setup is versioned alongside code. Secrets are isolated from agents.
- **The universal architecture: Control Plane / Data Plane.** The agent lives in the Control Plane alongside an orchestration tool. The actual workload tools (devboxes, shells) run in the Data Plane, separated by a network boundary. The agent never needs to be on the same machine as what it's manipulating.
- **Orchestration 101:** keep an agent in your control plane; let it provision/manage devboxes; let it read/write/bash into those devboxes.
- **If building your own:** invest in your devbox supply chain. The best UI / durable execution / integration is the one you didn't have to build. Build a devbox MCP as a "headless background agent."

---

## Architecture

### Devbox design: different constraints than CI

| Dimension | CI environments | Agent devboxes |
|-----------|----------------|----------------|
| Priority | Fidelity (reproducibility) | TTI (speed to interactive) |
| Control flow | Fixed, reviewed script | Agent-generated, dynamic |
| Trust model | Internal — you own the script | "Call from inside the house" — agent owns the flow |
| Blast radius | Limited (script-bounded) | Large (agent can do anything it's allowed to) |

### Ramp's approach (on Modal)

1. Per-repo environment images defined as code on Modal
2. Each image rebaked on a 30-minute cron (asynchronous — never blocks agent startup)
3. Each new devbox mounts the latest warm image, then runs `git pull` + `uv sync`
4. Secrets brokered through proxies/sidecars — never exposed to the agent process directly

**Outcome:**
- Nearly instant boot / TTI (image build cost is amortized, not paid at startup)
- Per-repo tunable permissions and ports
- Environment setup versioned alongside the codebase
- Secrets isolated from agents

### The universal architecture

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

- **Control Plane:** where the agent and its direct orchestration tools live. Governs what gets provisioned and how.
- **Data Plane:** where the actual workloads execute — devboxes, file systems, shells. Separated by a network boundary.
- This split is the answer to the trust problem: the agent operates in a controlled environment and reaches into the data plane via well-defined APIs rather than running directly on the compute it manipulates.

### Orchestration 101

1. Keep an agent in your control plane
2. Let it provision and manage devboxes (in the data plane)
3. Let it read/write/bash into those devboxes

### If you're building your own

- Invest in your devbox supply chain (image baking, warm pools, per-repo configuration)
- The best UI / durable execution / integration is the one you didn't have to build — prefer managed solutions where possible
- Build a devbox MCP as a "headless background agent" — expose devbox management as an MCP server so the orchestration agent can provision/destroy environments programmatically

---

## My Reactions

- "The call is coming from inside the house" is the clearest articulation of the agent trust problem I've heard at this conference. The CI vs. agent-devbox comparison made the threat model concrete immediately.
- The Ramp case study is valuable because it's specific: Modal, 30-min cron, uv sync, sidecar secrets. This is the kind of production detail that's usually absent from conference talks.
- The Control Plane / Data Plane split is a simple but load-bearing idea. It resolves the question of where the agent process should live relative to what it's manipulating — and why you don't want them co-located.
- The "headless background agent" framing for a devbox MCP is interesting — it reframes infrastructure management (provision, run, destroy) as an agentic task rather than a static config problem.
- This talk sits in productive tension with the Uber DevPods talk: Uber solved the same problem with K8s warm pools + balloon pods at 14K+ pod scale; Ramp solved it with Modal-managed warm images at a smaller scale. Both converge on the same principle (pre-bake, mount warm, don't pay startup cost at task time), validating the approach across very different infra stacks.

---

## Links

- [Adam Azzam](../speakers/adam-azzam.md)
- Related concepts: [DevPods](../concepts/devpods.md) · [Agent Environment Architecture](../concepts/agent-environment-architecture.md) · [Guardrails](../concepts/guardrails.md)
- Related talks: [Agentic SDLC at Uber](day2-1140-agentic-sdlc-at-uber.md) (DevPods — same problem, K8s-based solution) · [Letting the Interns Loose](day3-1110-letting-the-interns-loose.md) (one-VM-per-intern isolation, OS-level secrets enforcement)
- Source: [raw/dont-build-agents-build-environments.md](../../raw/dont-build-agents-build-environments.md)
