---
type: talk
tags: [sandbox, codemode, codeact, agent-environments, pydantic-monty, tool-design, python, rust, security]
updated: 2026-07-05
---

# Your Agent Needs a Sandbox, Not a Desert

**Speaker:** [Samuel Colvin](../speakers/samuel-colvin.md)  
**Affiliation:** Pydantic (Founder & CEO)  
**Day:** Day 3 — Session Day 2  
**Time:** 12:05pm–12:25pm  
**Room:** Track 1  
**Track:** Sandbox & Platform Engineering  

**Official description:** Everyone agrees agents need code execution. That agreement lasts right up until you ask how to do it. The default answer is usually something like "My agent needs a full Linux VM to succeed". That's a very convenient answer for sandbox providers, but I think it's often incorrect. In many real-world agent workflows, the model does not need a whole computer. It does not need arbitrary packages, shell access, CPython, node, let alone `awk` `sed` and `gcc`. It needs a small amount of safe, expressive compute: enough to write code, call tools, and keep intermediate state out of the context window. That is the idea behind Monty: a minimal Python interpreter, written in Rust, designed specifically for running code written by agents.

---

## Key Claims

- Most embedded agent workflows do not need a full Linux sandbox — they need a small, curated set of safe compute.
- Giving agents a full shell is the "desert": overwhelming surface area, most of it irrelevant and dangerous.
- The right answer for many agents is a "sandbox": a minimal set of purpose-built, high-level tools.
- Full sandboxes impose a 1–3s boot penalty even for tasks that take milliseconds — a 100,000× latency mismatch in the worst case.
- Codemode (aka CodeAct / RLM / "Programmatic tool calling") lets the agent write Python that orchestrates tools in code rather than via sequential LLM round-trips — the promise is "1000 tool calls with 1 LLM call".
- Constraints are a feature, not a bug: a minimal runtime is simpler, cheaper, faster, and safer.

---

## Slide Notes

### Codemode (slide 5/18)

Codemode = CodeAct = RLM (Recursive Language Models) = "Programmatic tool calling":

- The LLM writes Python (or JS); the Python calls tools.
- Tool composition, branching, looping all happen in code — not round-trips to the LLM.
- Intermediate state stays in the runtime, not in the context window.
- The agent only ever sees **two** tools: `run_code` and `find_tools`.
- "1000 tools in 1000 tokens" is the promise; better yet: "1000 tool calls with 1 LLM call".

Architecture: `AGENT → LLM (writes Python) → CODEMODE (find_tools, run_code) → YOUR TOOLS (get_weather, sql_query, read_file, load_skill, convert_temp, send_email, write_file, create_chart)`

### But how do we run it? (slide 6/18)

> The people who benefit most from codemode are those selling sandboxes.

The implicit critique: codemode is often used as a justification for selling full Linux VM sandboxes. Colvin's talk argues that's the wrong default for many use cases.

### Two Paradigms (slides 7 and 9/18)

**Light tasks** (milliseconds): pass tool outputs to next tool, calculations, run queries, render a chart.  
**Heavy tasks** (minutes): clone a repo, run a coding agent, heavy computation, compilation.

The problem: both task types share the same execution environment options (VM, Container, gVisor, Firecracker) — and the same **1–3s typical boot time**. A light task pays 1–3s of overhead for milliseconds of actual work. This is the core latency argument.

### The sandbox vs. the desert (slide 10/18)

**The desert** — a raw shell gives the agent everything, most of which it doesn't need:
`awk sed gcc g++ clang less more grep egrep fgrep make cmake bash zsh sh fish curl wget tar gzip bzip2 xz zip unzip find xargs locate vim nano emacs ed python3 pip pipx node npm pnpm yarn git svn hg ssh scp rsync cron systemd apt apt-get dpkg yum dnf brew jq yq tr cut sort uniq head tail`

**The sandbox** — exposes only what the agent needs:
- `sql_query`
- `create_chart`
- `create_table`
- `load_skill`

### Introducing Pydantic Monty (slide 12/18)

A minimal Python interpreter, **written in Rust**, for code written by AI.

- Safe by default: no filesystem, env, or network unless you grant it.
- Pass in your own functions — the model calls them as ordinary Python.
- Type-checks code (via `ty`) before it runs.
- Snapshottable — pause at an external call, resume later, anywhere.
- Supports a growing subsection of the standard library: `re`, `json`, `datetime`, `Pathlib`, etc.
- Monty code runs in a pool of subprocesses; wire protocol communicates with the main process.

```
uv add pydantic-monty
npm i @pydantic/monty
cargo add monty
```

---

## Reactions

- The title's framing is sharp and the desert/sandbox contrast is memorable — "too much sand" is a useful meme.
- This is the clearest counter-argument to the talks earlier in Track 1 (Azzam, Goyal) that are primarily about *how* to build fast/secure full sandboxes. Colvin asks *whether* you need a full sandbox at all.
- The boot-time argument is concrete: paying 1–3s for a millisecond task is a 1,000–3,000× overhead ratio. Hard to argue with on latency grounds.
- Monty is a genuine product announcement. Written in Rust, snapshottable, type-checked before execution — this sounds like a real alternative to subprocess-based Python execution for agent tool use.
- Missing: the Intro, Security, and Demo sections (slides 1–4, 8, 11, 13–18). Would particularly like to see the Security section — how does Monty's no-filesystem default compare to gVisor-level isolation?
- Interesting that the slide deck is 18 slides for a 20-minute talk — suggests a fast-paced, demo-heavy delivery.

---

## Links

- [Samuel Colvin](../speakers/samuel-colvin.md)
- [Agent Environment Architecture](../concepts/agent-environment-architecture.md) — prior sessions on sandbox/devbox design; this talk is the counter-argument
- [Don't Build Agents, Build Environments](day3-1045-dont-build-agents-build-environments.md) — Adam Azzam (Modal), same track, earlier that morning
- [Letting the Interns Loose](day3-1110-letting-the-interns-loose.md) — Shashank Goyal (OpenRouter), same track
