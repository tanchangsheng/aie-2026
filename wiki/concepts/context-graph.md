---
type: concept
tags: [agentic-sdlc, context-graph, knowledge-graph, org-knowledge, tool-call-efficiency]
updated: 2026-06-30
---

# Context Graph

## Definition

A Context Graph is an organization-wide knowledge graph (Uber's: 40M+ nodes & edges, 150+ node/edge types) that encodes relationships between engineering entities — services, teams, repositories, incidents, feature flags, datasets, and more — so agents can traverse connections directly instead of discovering them through repeated tool calls.

## What It Solves

- **Agents guess without relationships** — without a graph, agents can't traverse connections they can't see; they have to infer or search for relationships that already exist in scattered systems.
- **Context is scattered across systems** — 30+ sources including incidents, ownership records, code changes, and feature flags, each requiring separate integration.
- **Fan-out burns tokens and time** — assembling connected context across these sources takes many sequential tool calls, and the unpredictability compounds with each hop.

## Architecture

- **Scale:** 40M+ nodes & edges, 150+ node/edge types
- **Sample node types:** Engineer, Team, Service (the apparent center/hub type), Repository, Code Change, Feature, Feature Flag, Incident, On-Call, Alert, Dataset, Database, Pipeline, API Endpoint, Metric, Design Doc, Design Review, Work Item, Epic, Experiment, Documentation, Event Stream, Chat Channel, AI Tool, Mobile App, Screen
- **Data sources:** Asset Registry, Code Review, Issue Tracker, Project Planning, Incident Mgmt, Feature Flags, CI/CD, and 12+ more
- **Use cases:** Planning, Ownership, Oncall & RCA, Data Analysis, Security

## The Headline Result: Graph vs. No Graph

Same question, both paths reach the same correct answer — only the path differs:

> "What % of Mobility trips in India use Cash?"

| Metric | With Graph | Without Graph |
|---|---|---|
| Time | 44s | 627s (~14x slower) |
| Cost | $0.38 | $2.75 (~7x more expensive) |
| Tool calls | 8 | 94 |

_(Corrected 2026-07-05: a higher-resolution re-upload of the source slide photo confirmed the cost was $0.38, not $0.36 as originally transcribed from a lower-resolution photo. See [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) for detail.)_

Without the graph, the agent burns dozens of tool calls just exploring schema and discovering which table/system has the answer. With the graph, that discovery step collapses to a direct traversal.

## Why This Matters Beyond One Demo

The gap (14x time, 7x cost, ~12x tool calls) for an *identical correct answer* makes the case that org-knowledge structure is not a nice-to-have — it's a direct multiplier on agent efficiency for any question that spans more than one internal system. This generalizes the "tool-call fan-out is expensive" problem named in [MCP Gateway](mcp-gateway.md) one level up: even with cheap tool-calling mechanics, an agent without a map still has to search blindly.

## Relationship to Other Concepts

- Directly complements [MCP Gateway](mcp-gateway.md): the gateway makes individual tool calls cheap, but the graph reduces *how many* calls are needed in the first place by making relationships directly traversable.
- Conceptually adjacent to [Context Engineering](context-engineering.md)'s offload/retrieval pattern (durable external store + small live window) — but here the "external store" is a structured graph of org relationships rather than a file tree or RAG corpus.

## Open Questions

- How is the graph kept fresh as the 30+ source systems change (real-time sync vs. batch indexing)?
- Does graph traversal degrade gracefully when a node type has incomplete or stale data, or does it fail silently?
- How does the agent decide when to query the graph vs. fall back to direct tool calls — is this a router decision, a skill, or built into the harness?

## Sources

- [Agentic SDLC at Uber](../talks/day2-1140-agentic-sdlc-at-uber.md) — Uday Kiran Medisetty, Adam Huda
