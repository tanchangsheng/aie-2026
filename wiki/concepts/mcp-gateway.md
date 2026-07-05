---
type: concept
tags: [agentic-sdlc, mcp-gateway, mcp, tool-calling, context-cost, llm-infrastructure]
updated: 2026-06-30
---

# MCP Gateway

## Definition

An MCP Gateway is a centralized proxy that fronts an organization's MCP (Model Context Protocol) servers — both first-party (internal tools) and third-party (Glean, Slack, GitHub, etc.) — handling auth, discovery, schema, routing, execution, rate limiting, and observability in one place, rather than having every agent harness integrate with every MCP server directly.

## The Core Problem: Tool-Schema Context Cost

Loading N tool schemas directly into an agent's context window is expensive and scales badly as the number of available tools grows. Uber's talk ranks four caller patterns from most to least expensive:

| Pattern | Cost | Mechanism |
|---|---|---|
| Direct MCP | $$$$ | N tool schemas loaded into context |
| Omni MCP | $$$ | One MCP that discovers & calls any server; nothing pre-loaded into context |
| `aifx mcp call` | $$ | A CLI wrapper around any MCP — JSON in/out, zero context cost |
| Code Mode | $ | Skills call MCP CLIs and parse outputs automatically — cheapest |

The pattern is a clear progression: each tier moves more of the "how do I call this tool" decision out of the LLM's context and into deterministic code/CLI plumbing, paying for it only when a tool is actually invoked rather than up front for every available tool.

## Architecture (Uber's Implementation)

```
Agent harnesses (Claude Code, Codex, OpenCode, Cursor, Langchain)
        ↓ (via one of the 4 caller patterns above)
MCP Gateway
  Request flow:  Auth → Discovery → Schema → Route → Execute
  Management:    Auth & Identity | Observability | Rate Limits
  Infrastructure: Proxy MCP | Registry | Playground
        ↓
MCP Servers — 1P (Code, Phabricator, QueryRunner, Flipt) and 3P (Glean, Slack, Google Workspace, GitHub)
```

## Relationship to Other Concepts

- This is the same "minimize what's in context" problem that [Context Engineering](context-engineering.md) addresses for chat history — applied here specifically to tool/schema definitions rather than conversation turns.
- Pairs with [Agent Skills](agent-skills.md): "Code Mode" (the cheapest caller pattern) is explicitly skills that call MCP CLIs, meaning the skills layer is partly *how* the MCP Gateway gets used cheaply in practice.

## Open Questions

- What's the latency tradeoff of routing every tool call through a CLI wrapper (`aifx mcp call`) versus a direct MCP connection?
- How does the Registry decide what's discoverable to a given agent — is it scoped by identity, by team, or globally open?
- Does Code Mode's "parse outputs automatically" step introduce a new failure mode (brittle parsing) that Direct MCP's structured schemas avoid?

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda
