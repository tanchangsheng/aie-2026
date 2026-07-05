---
type: concept
tags: [testing, agent-testing, harness, mcp, sandbox, quality-assurance, sqlite]
updated: 2026-07-05
---

# Agent-Driven Testing

## Definition

A testing methodology in which an LLM agent authors, organizes, and executes its own test suite against the service or environment it will eventually use in production. The agent uses the same interface (e.g. MCP) that real production agents use — so the test is a realistic simulation of live usage, not a synthetic unit test.

The key distinction from conventional testing: the agent writes the tests. This enables test generation that covers domain-specific task shapes (ML training, file I/O, web scraping) that human engineers would be unlikely to enumerate systematically.

## Orellana's Implementation (Amazon AgentCore)

**Why agent-authored tests?** A coding agent can enumerate domain-specific failure scenarios more exhaustively than a human writing unit tests, and it will naturally write the kinds of tasks it actually performs — rather than idealized scenarios.

**Architecture:**

1. A coding agent writes ~1,000 tests, organized by domain, stored in a SQLite test plan
2. A hardened runner executes each test via MCP — the same MCP wire real customer agents use
3. The runner handles credential refresh, pass/fail recording, and triage (service-bug ≠ input-bug)
4. Failures → findings → fixes → re-run (closed loop)

**Why MCP as the test wire?** It ensures the test exercises the full stack the agent actually uses, not a mocked or bypassed path. Interface bugs and service bugs are separated by design: a small validation harness (seed tasks: run a script, JSON→CSV, scrape a page) is run first to confirm the interface is clean before scaling to 1,000 tests.

**Result (Amazon AgentCore):** 88% of 1,000 tasks passed; 12% revealed 18 distinct failure modes that no unit test would catch.

## What It Catches That Unit Tests Miss

Conventional unit tests mock infrastructure. Agent-driven tests run against real infrastructure over the real interface. The failure modes this exposes are precisely the ones the slides documented:

- Binary encoding corruption in transport (base64 coercion of binary data)
- Null bytes silently truncating file downloads
- Network isolation blocking pip with no useful error message
- Path traversal inputs accepted without validation
- State not persisting across calls (sandbox lifecycle)
- Environment fingerprinting causing silent bot-detection blocks

These are integration failures between the agent's expectations and the sandbox's actual behavior. They are invisible to unit tests because unit tests don't run the full stack.

## The Triage Step Is Critical

Not every failure is a service bug. The hardened runner's triage logic distinguishes:

- **Service bug:** the sandbox misbehaved — fix the service
- **Input bug:** the agent's test generated a bad input — the test is wrong

Without this distinction, a high failure rate is uninterpretable — you don't know whether the service is broken or the agent is writing bad tests. Separating the two is what makes the closed loop actionable.

## Generalization

The method isn't specific to sandboxes. Any service that agents consume via a well-defined interface (MCP, REST, tool calls) can be tested this way:

1. Wrap the service in the same interface agents use
2. Have an agent author a large task set, organized by domain
3. Validate with seed tasks to separate interface noise from service bugs
4. Run at scale, triage results, close the loop

The SQLite test plan format makes the test suite inspectable, filterable by domain, and re-runnable incrementally as bugs are fixed.

## Relationship to Other Concepts

- [Silent Semantic Failure](silent-semantic-failure.md) — agent-driven testing is specifically designed to surface silent failures; completion-based monitoring alone would miss them
- [Agent Environment Architecture](agent-environment-architecture.md) — the harness architecture (coding agent + runner + cloud sandbox) is itself an application of the control/data plane split
- [Agent Skills](agent-skills.md) — skills are tested by running them; agent-driven testing could apply to skill validation as well as infrastructure validation

## Open Questions

- What's the right test size for coverage vs. cost? Orellana chose ~1,000. At what N does marginal coverage flatten?
- How do you manage test debt — if the sandbox evolves, which tests become invalid vs. which tests are now legitimately failing?
- The agent-authored test suite is presumably not reviewed by a human line by line. What's the quality bar for the agent's test writing? Is there a meta-test that validates the tests themselves?
- Does this methodology generalize to testing agent *behavior* (not just infrastructure) — e.g., having one agent write tasks and another agent's responses graded for correctness?

## Sources

- [1,000 Agent Tasks in a Sandbox](../talks/day3-1425-1000-agent-tasks-sandbox.md) — Kevin Orellana (Amazon AgentCore): primary and only source
