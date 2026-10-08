# Multi-Agent Coordination

## Overview

A **multi-agent system** is a network of AI agents working together to
accomplish tasks that would be impractical for a single agent. One agent — the
**orchestrator** — decomposes a high-level goal into subtasks and delegates them
to specialized **subagents**. Each subagent operates with its own focused
context and reports results back to the orchestrator, which synthesizes them
into a coherent outcome.

Multi-agent systems extend the capabilities of a single agent but introduce new
failure modes, coordination overhead, and trust complexity. They are appropriate
when a task genuinely benefits from parallelism or specialization — not as a
default architecture.

## The Orchestrator / Subagent Pattern

The orchestrator is responsible for planning and synthesis. Subagents are
responsible for execution. Neither role is fixed to a particular model; the same
model can act as orchestrator for one task and subagent for another.

```text
+------------------------------------------+
|              Orchestrator                 |
|                                           |
|  1. Receive high-level goal               |
|  2. Decompose into subtasks               |
|  3. Delegate to subagents                 |
|  4. Collect and synthesize results        |
|  5. Return final output or escalate       |
+------+----------+----------+--------------+
       |          |          |
       v          v          v
  +----+----+ +----+----+ +----+----+
  |Subagent | |Subagent | |Subagent |
  |    A    | |    B    | |    C    |
  |         | |         | |         |
  | Focused | | Focused | | Focused |
  | context | | context | | context |
  +---------+ +---------+ +---------+
  Code review  Test gen    Doc update
```

### Why Decompose?

The primary motivation for multi-agent decomposition is **context management**.
A single agent working on a large task accumulates tool outputs, error messages,
and intermediate results in its context window. As the context grows, reasoning
quality degrades (see
[../llm/context_pollution.md](../llm/context_pollution.md)). Subagents each
receive only the context relevant to their specific subtask, keeping their
reasoning sharp.

A secondary motivation is **parallelism**. Independent subtasks can be delegated
to subagents that run concurrently, reducing the wall-clock time of the overall
task.

## Context Propagation

A critical design decision in any multi-agent system is what context each agent
receives and what it can pass back.

### Isolated Context (Preferred)

Each subagent receives only what it needs: the specific subtask description,
relevant files or data, and any constraints. It returns only its result. The
orchestrator's full conversation history is not shared with subagents.

```text
Orchestrator context:
  [Goal] [Plan] [Subagent A result] [Subagent B result] ...

Subagent A context:
  [Subtask A description] [Relevant files] -> [Result A]

Subagent B context:
  [Subtask B description] [Relevant files] -> [Result B]
```

Isolated context prevents errors and hallucinations in one subagent from
contaminating the reasoning of another. It also keeps each subagent's context
small and focused.

### Shared Context (Use With Caution)

In some patterns, subagents share a common context store — a file, a database,
or a message queue — that multiple agents read from and write to. This enables
tighter coordination but creates race conditions, write conflicts, and the risk
of one agent's output polluting another's reasoning.

Shared context should be append-only where possible, and writes should be
structured (JSON, YAML) rather than free text to reduce the risk of a malformed
entry corrupting the shared state.

## Coordination Strategies

### Sequential Delegation

The orchestrator delegates subtasks one at a time, waiting for each result
before proceeding. Simple and predictable; the orchestrator can use the result
of each subtask to inform the next delegation decision.

```text
Orchestrator --> Subagent A --> [result A]
                     |
                Orchestrator --> Subagent B (informed by result A) --> [result B]
```

Use when subtasks have dependencies: the output of one informs the input of the
next.

### Parallel Delegation

The orchestrator delegates multiple subtasks simultaneously and collects results
as they complete. Faster for independent subtasks; requires the orchestrator to
handle partial results and out-of-order completion.

```text
Orchestrator --> Subagent A ---\
             --> Subagent B ---+--> [collect all] --> synthesize
             --> Subagent C ---/
```

Use when subtasks are genuinely independent: they do not read from or write to
shared state that the other subtasks depend on.

### Map-Reduce

A large input is split into chunks; each chunk is processed by a separate
subagent (map); the orchestrator combines the results (reduce). Common for tasks
like analyzing a large codebase, reviewing a long document, or processing a
dataset in parallel.

```text
Large input
    |
    +---> Chunk 1 --> Subagent --> Result 1 ---\
    +---> Chunk 2 --> Subagent --> Result 2 ---+--> Orchestrator --> Final result
    +---> Chunk 3 --> Subagent --> Result 3 ---/
```

## Failure Modes in Multi-Agent Systems

Multi-agent systems have failure modes that do not exist in single-agent
systems.

### Cascade Failure

A subagent produces an incorrect result. The orchestrator, trusting the result,
uses it as the basis for the next delegation. The error propagates and compounds
across subsequent subagents until the final output is significantly wrong in a
way that is difficult to trace back to the original source.

Mitigation: validate subagent outputs against defined schemas or quality checks
before the orchestrator consumes them. Do not propagate unvalidated subagent
output directly into other subagents' context.

### Coordination Deadlock

Two or more agents are each waiting for the other to complete before proceeding.
This is most common in shared-context architectures where agents read state
written by other agents.

Mitigation: avoid circular dependencies between agents. If Agent A's output
feeds Agent B, Agent B's output must not feed back into Agent A in the same
execution.

### Conflicting Actions

Two subagents operating in parallel both attempt to modify the same resource (a
file, a registry entry, a branch) with incompatible changes. The later write
silently overwrites the earlier one, or both fail with a conflict error.

Mitigation: assign each subagent a disjoint write scope before execution begins.
The orchestrator is responsible for partitioning the work such that parallel
subagents never write to the same resource.

### Orchestrator Context Overload

The orchestrator accumulates the results of many subagents in its context
window. As the number of subagents grows, the orchestrator's context fills with
results it must synthesize, eventually degrading its ability to reason about the
overall goal.

Mitigation: subagents should return concise, structured summaries rather than
raw output. The orchestrator's context should contain results, not logs.

## Trust Between Agents

Agents in a multi-agent system should not automatically trust each other. A
subagent that has been compromised through prompt injection (see
[trust_and_security.md](trust_and_security.md)) could return a malicious result
that causes the orchestrator to take harmful actions.

The orchestrator should treat subagent results with the same skepticism it
applies to any external tool output: validate structure, check for anomalies,
and do not pass subagent output directly into another agent's system prompt
without sanitization.

## Summary

Multi-agent systems are a powerful tool for tasks that benefit from
decomposition, parallelism, or specialized focus. They are not a default
architecture. The coordination overhead, failure modes, and trust complexity
they introduce are real costs that must be weighed against the benefits for each
use case. When in doubt, start with a single well-harnessed agent before adding
the complexity of orchestration.

## Related

- [overview.md](overview.md)
- [agent_harness.md](agent_harness.md)
- [pipeline_patterns.md](pipeline_patterns.md)
- [trust_and_security.md](trust_and_security.md)
- [../llm/context_pollution.md](../llm/context_pollution.md)
- [AI Glossary: Sub-agents](../../reference/ai_glossary.md)
- [AI Glossary: Agent](../../reference/ai_glossary.md)

## Links

- [Mixture-of-Agents Enhances Large Language Model Capabilities](https://arxiv.org/abs/2406.04692)
- [AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation](https://arxiv.org/abs/2308.08155)
