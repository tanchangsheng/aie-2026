---
type: speaker
tags: [aws, amazon-agentcore, sandbox, code-execution, browser-automation]
updated: 2026-07-05
---

# Kevin Orellana

| Field | Value |
|-------|-------|
| **Role** | Software Engineer |
| **Affiliation** | Amazon Web Services |
| **Twitter** | [@KevssOrellana](https://x.com/KevssOrellana) |
| **LinkedIn** | [linkedin.com/in/kevinorellana](https://www.linkedin.com/in/kevinorellana/) |

## Background

Kevin Orellana is a software engineer at AWS, where he builds the sandboxed code-execution and browser-automation infrastructure that lets AI agents — including coding agents — run code and drive websites safely at scale. He previously worked on Amazon Bedrock's model-serving platform, where he tech-led the launch of Anthropic's Claude Sonnet on Bedrock and helped design the infrastructure behind several frontier-model launches.

## Talks at AIE World Fair 2026

- [1,000 Agent Tasks in a Sandbox: What Breaks When LLMs Write and Run Code](../talks/day3-1425-1000-agent-tasks-sandbox.md) — Day 3 · 2:25pm · Track 1

## Recurring Themes

- **Empirical sandbox testing:** testing infrastructure the way agents actually use it, not with unit tests
- **Silent failure modes:** the class of problems where the call says success but the bytes are wrong — invisible to standard monitoring
- **Environment as a capability constraint:** the sandbox controls that provide security are the same controls that can silently break domain-specific agent tasks

## Notes

The only Day 3 speaker presenting live operational data from a production cloud sandbox rather than an architectural framework or product pitch. His work sits at the intersection of the Bedrock serving infrastructure background (model launches) and the AgentCore sandbox infrastructure he now builds.
