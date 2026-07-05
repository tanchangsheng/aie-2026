---
type: concept
tags: [agentic-sdlc, devpods, agent-workspaces, kubernetes, developer-environments, modal, devboxes]
updated: 2026-06-30
---

# DevPods

## Definition

DevPods are ephemeral, isolated cloud development environments purpose-built for agents (and engineers) to run code in, distinct from a laptop or a shared CI runner. Uber runs 14K+ DevPods at peak with ~seconds startup time, across 5 GKE regions.

## What It Solves

- **Agents can't run on laptops** — autonomous agents need isolated, ephemeral environments with no risk of clobbering a developer's local machine state.
- **Environment setup kills agent speed** — cloning a monorepo, installing dependencies, and indexing code normally takes minutes; an agent doing this on every task is a major latency tax.
- **Language silos fragment execution** — separate Go/Java/Web/iOS environments mean an agent can't work across repos spanning multiple languages in one session.

## Architecture

```
Users: Engineers | Interactive Agents | Autonomous Agents
        ↓
DevPod Platform
  Mobile simulation:  iOS Simulator | Android Emulator | Visual Validation
  AI DevPod:           AI-optimized image | one-click setup | non-engineer access
  Mega DevPod:          Go, Java, Web, Data, iOS, Android → unified environment
  Instant start:        Balloon pods | warm pool | snapshot store | pre-indexed code
  Core runtime:          K8s Pod | DinD sidecar | SSD volume | agent runtime
        ↓
GKE regions: Oregon, Virginia, Netherlands, São Paulo, Mumbai
```

## How ~Seconds Startup Is Achieved

The "Instant Start" layer is the key to the speed claim — it avoids cold-starting an environment from scratch:
- **Balloon pods** — pre-provisioned, idle capacity ready to be claimed
- **Warm pool** — a standing pool of ready-to-use pods
- **Snapshot store** — pre-built filesystem/dependency state rather than installing on demand
- **Pre-indexed code** — the monorepo is already cloned and indexed before the agent asks for it

This directly targets the "env setup kills agent speed" problem: the expensive parts (clone, install, index) are done ahead of time and amortized across many DevPod claims.

## Mega DevPod: Solving the Language-Silo Problem

Rather than maintaining separate Go, Java, Web, iOS, Android, and Data environments, Uber unifies them into one environment an agent can work across — directly addressing the cross-repo, cross-language fragmentation problem named in the talk.

## Relationship to Other Concepts

- DevPods are the execution substrate that [Agent Skills](agent-skills.md) and the [MCP Gateway](mcp-gateway.md)'s tool calls actually run inside.
- Conceptually parallel to [Self-Hosted Inference](self-hosted-inference.md)'s "owning the infrastructure" framing, but for compute/dev environments rather than model serving — both are about an org choosing to operate its own substrate at scale rather than relying on ad hoc/local setups.

## Open Questions

- What's the cost model for keeping a warm pool + balloon pods idle at 14K+ peak scale?
- How is state isolated between an autonomous agent's DevPod and the production systems it might call out to?
- "Non-engineer access" is listed under AI DevPod — what does a non-engineer actually do with one?

## Comparison: Ramp's Approach (on Modal)

Adam Azzam's talk describes how Ramp solved the same problem with a different stack:

- Per-repo images defined as code on Modal, rebaked on a 30-min cron
- New devboxes mount the latest warm image, then `git pull` + `uv sync`
- Secrets brokered through proxies/sidecars — never exposed to the agent

The outcome is the same (near-instant TTI, secrets isolation), but the mechanism is managed image snapshots on Modal rather than K8s balloon pods + snapshot store. Both converge on the principle: pre-bake; never pay startup cost at task time. See also [Agent Environment Architecture](agent-environment-architecture.md) for a cross-team comparison.

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda
- [Don't Build Agents, Build Environments](../talks/day3-1045-dont-build-agents-build-environments.md) — Adam Azzam (Modal/Ramp case study)
