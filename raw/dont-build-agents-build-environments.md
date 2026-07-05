# Don't build agents, build environments

**Speaker:** Adam Azzam  
**Role:** Member of Product Staff, Modal (formerly VP of Product at Prefect; maintainer of Prefect and FastMCP)  
**Event:** AI Engineer World Fair, San Francisco  
**Day:** Day 3 — Session Day 2  
**Time:** 10:45am–11:05am  
**Room:** Track 1  
**Track:** Sandbox & Platform Engineering  
**LinkedIn:** https://linkedin.com/in/adam-azzam  
**Twitter:** https://x.com/aaazzam  
**Website:** https://adamazzam.com  
**Slides:** [slides/dont-build-agents-build-environments/](slides/dont-build-agents-build-environments/)

---

## Slide 01 — Speaker intro

![slide-01](slides/dont-build-agents-build-environments/slide-01.HEIC)

👋, I'm Adam  
Maintainer  
Prefect, FastMCP

---

## Slide 02 — This talk in a nutshell

![slide-02](slides/dont-build-agents-build-environments/slide-02.HEIC)

- Background agents are converging on a universal system architecture.
- The hardest / most reinvented parts:
  - devboxes
  - orchestration
- knowledge-cutoff: July 2026

---

## Slide 03 — Devboxes: designing a system that generates devboxes on demand

![slide-03](slides/dont-build-agents-build-environments/slide-03.HEIC)

- Not new, just different constraints.
- CI favors fidelity over TTI.
- Trust boundary totally different.

---

## Slide 04 — The trust problem: "the call is coming from inside the house!"

![slide-04](slides/dont-build-agents-build-environments/slide-04.HEIC)

- Totally different threat model
- CI executes a fixed, reviewed script
- Feels more like running CircleCI than CI
  - Don't own the control flow
  - Big blast-radius problem

---

## Slide 05 — Case study: What did Ramp do?

![slide-05](slides/dont-build-agents-build-environments/slide-05.HEIC)

- Defined per-repo images in code on Modal.
- Rebaked each image on a 30-min cron.
- Each new devbox mounted a warm image (+ git pull / uv sync).
- Secrets brokered through proxies/sidecars.

---

## Slide 06 — Results: much better user experience!

![slide-06](slides/dont-build-agents-build-environments/slide-06.HEIC)

- Each image builds asynchronously.
- Nearly instant boot / TTI.
- Could tune perms/ports per repo.
- Setup versioned along env.
- Secrets isolated from agents.

---

## Slide 07 — Architecture diagram: Control Plane / Data Plane

![slide-07](slides/dont-build-agents-build-environments/slide-07.HEIC)

[Hand-drawn diagram]

- Left bubble (Control Plane): Agent + Tool
- Right bubble (Data Plane): Tool + Tool
- Connected via: Network
- Key insight: Agent lives in the Control Plane; workloads/tools run in the Data Plane, separated by a network boundary

---

## Slide 08 — Orchestration 101

![slide-08](slides/dont-build-agents-build-environments/slide-08.HEIC)

- Keep an agent in your control plane.
- Let it provision/manage devboxes.
- Let it read/write/bash into those devboxes.

---

## Slide 09 — If you're building your own

![slide-09](slides/dont-build-agents-build-environments/slide-09.HEIC)

- Invest in your devbox supply chain.
- Best UI / durable execution / integration is the one you didn't have to build.
- Build a devbox MCP as a "headless background agent".
