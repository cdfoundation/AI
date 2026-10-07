# Agents

Explanation documents covering AI agents and agentic systems in the context of
the CDFoundation CI/CD AI SIG. These documents are understanding-oriented: they
discuss background, architecture, trade-offs, and rationale. For step-by-step
guidance see the [how-to guides](../../how-to/). For concise definitions see the
[AI Glossary](../../reference/ai_glossary.md).

## Contents

| Document                                       | Description                                                                                      |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [overview.md](overview.md)                     | What agents are, the agent spectrum, and how they differ from traditional pipeline steps         |
| [agent_harness.md](agent_harness.md)           | Harness architecture, the execution loop, security gates, and the tool registry                  |
| [pipeline_patterns.md](pipeline_patterns.md)   | Patterns for integrating agents into CI/CD pipelines: async execution, HITL gates, and rollback  |
| [multi_agent.md](multi_agent.md)               | Orchestrator/subagent patterns, context propagation, coordination, and multi-agent failure modes |
| [trust_and_security.md](trust_and_security.md) | Trust boundaries, prompt injection, privilege scoping, and audit logging for agentic systems     |

## Scope

These documents cover AI agents as they relate to CI/CD delivery pipelines and
platform engineering. The focus is on how agents are architected, how they
integrate with delivery workflows, and what security and operational properties
they require. General AI agent frameworks and coding assistant workflows are
discussed only where they inform CI/CD-specific design decisions.
