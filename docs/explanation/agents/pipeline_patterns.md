# Agentic Pipeline Patterns

## Overview

Integrating an AI agent into a CI/CD pipeline requires the same discipline as
integrating any other non-deterministic, long-running, or externally-dependent
step: the pipeline must define how the agent is triggered, how long it is
allowed to run, what constitutes success or failure, and what happens when it
goes wrong.

The patterns in this document describe how agentic steps compose with
traditional pipeline stages, how to structure human oversight, and how to handle
the failure modes specific to agent workloads.

## The Agent Step in a Pipeline

An agent occupies a **step** in the pipeline, just like a build, test, or
deployment step. From the pipeline's perspective, the agent step has inputs,
outputs, a timeout, and an exit condition. What happens inside the step — the
Thought-Action-Observation loop — is the agent's concern, not the pipeline's.

```text
+------------------+     +------------------+     +------------------+
|    Build Step    |     |   Agent Step     |     |   Deploy Step    |
|                  |     |                  |     |                  |
|  - Compile code  | --> |  - Read PR diff  | --> |  - Push artifact |
|  - Run linters   |     |  - Analyze impact|     |  - Update config |
|  - Unit tests    |     |  - Write report  |     |                  |
|                  |     |  - Exit 0/1      |     |                  |
+------------------+     +------------------+     +------------------+
     Deterministic            Non-deterministic         Deterministic
```

The agent step must produce a clear exit signal (pass or fail) that the pipeline
can act on. An agent that runs indefinitely, produces ambiguous output, or
silently swallows errors is not a well-integrated pipeline step.

## Synchronous vs Asynchronous Execution

### Synchronous Agents

A synchronous agent runs inline as a blocking pipeline step. The pipeline waits
for the agent to complete before proceeding. This is the simplest integration
pattern and appropriate for short-duration tasks (seconds to a few minutes)
where the agent's output is required before downstream steps can run.

```text
Pipeline:  Build --> [Agent: code review] --> Tests --> Deploy
                             |
                        Blocks pipeline
                        until complete
```

Synchronous agents are subject to the same timeout constraints as any other
pipeline step. A well-designed synchronous agent step has a hard wall-clock
timeout that triggers a clean failure, not an indefinite hang.

### Asynchronous (Background) Agents

An asynchronous agent is triggered by the pipeline but runs independently,
reporting results through a separate channel (a pull request comment, a webhook,
a status check). The triggering pipeline step exits immediately after launching
the agent; downstream steps do not wait for it.

```text
Pipeline:  Build --> Tests --> Deploy
              |
              +--> [Trigger agent job] --> (runs in background)
                                                |
                                          PR comment / webhook
                                          when complete
```

Asynchronous agents are appropriate for tasks that run longer than a pipeline
step timeout, tasks whose output is informational rather than gate-blocking, or
tasks that should not hold up the delivery pipeline on their own.

The trade-off is observability: an asynchronous agent that fails silently is
harder to detect than a synchronous step that returns a non-zero exit code. The
agent must emit structured logs and a clear completion signal regardless of
whether it succeeded.

## Event-Driven Triggers

Agents in a CI/CD context are typically triggered by events rather than on a
fixed schedule. Common trigger patterns:

```text
Event Source          Trigger Condition           Agent Task
-----------           -----------------           ----------
Pull request opened   New diff available          Code review, impact analysis
Test suite failed     Exit code != 0              Root cause analysis, fix proposal
Deployment completed  Health check passed         Smoke test, regression summary
Data drift detected   Metric threshold exceeded   Retraining pipeline initiation
New issue opened      Label matches criteria      Triage, duplicate detection
Scheduled cron        Time-based                  Dependency audit, doc freshness
```

Event-driven triggers should be idempotent: if the same event fires twice (which
happens in at-least-once delivery systems), running the agent twice should not
produce harmful side effects. For agents that write to shared state (comments,
files, registries), this means checking whether the action has already been
taken before taking it.

## Human-in-the-Loop Gates

Not every agent action should proceed automatically. A **Human-in-the-Loop
(HITL) gate** pauses the pipeline at a defined point and requires explicit human
approval before the agent's actions take effect or before the pipeline proceeds.

```text
Agent proposes action
         |
         v
+--------+--------+
|   HITL Gate     |
|                 |
|  Show proposal  |
|  to human       |
+----+-------+----+
     |       |
  Approve  Reject
     |       |
     v       v
  Proceed  Discard /
           Request revision
```

HITL gates are appropriate when:

- The agent's action is **irreversible**: deploying to production, deleting
  artifacts, merging branches.
- The agent's action has **significant downstream impact**: changes to shared
  infrastructure, updates to security policy, modification of shared libraries.
- The **confidence in the agent's judgment** for the specific task is not yet
  established through operational experience.

HITL gates add latency to the pipeline. They should be designed to minimize the
time a human spends reviewing: the agent should surface a clear, concise summary
of what it proposes to do and why, not raw output that the human must interpret
themselves.

## Rollback and Failure Handling

Agent steps fail in ways that traditional steps do not. A traditional step
either completes successfully or exits with an error code. An agent can:

- Complete its iteration limit without finishing the task.
- Take a sequence of partially-correct actions that leave the system in an
  inconsistent state.
- Produce output that passes format validation but is semantically wrong.
- Hang on a tool call that does not return within its timeout.

### Iteration and Token Limits

Every agent step must have a hard limit on the number of reasoning iterations
and on total token consumption. When either limit is reached, the agent step
fails cleanly — it does not attempt to "finish" by producing a partial result.

```text
+----------------------+
|   Agent Execution    |
+----------------------+
| Max iterations: 50   |
| Max tokens: 100,000  |
| Timeout: 10 minutes  |
+----------------------+
| On limit reached:    |
| -> Log state         |
| -> Emit failure exit |
| -> Do not commit     |
|    partial changes   |
+----------------------+
```

### Partial Action Rollback

If an agent takes actions that modify shared state (writes files, pushes
commits, updates a registry) before failing, the pipeline must have a defined
strategy for cleaning up partial changes. Options include:

- **Transactional execution**: Stage all changes in an isolated workspace and
  commit only on confirmed success. If the agent fails, discard the workspace.
- **Compensation actions**: On failure, run a defined cleanup step that reverses
  known modifications.
- **Human review gate**: On failure, surface the partial state for human review
  before deciding whether to roll back or continue.

## Composing Agents with Traditional Steps

Agents integrate most cleanly into a pipeline when they are treated as
**black-box steps** with well-defined contracts: a set of inputs, a set of
possible outputs, and a clear pass/fail signal. The pipeline does not need to
understand the agent's internal reasoning; it only needs to act on the result.

```text
                    Pipeline Orchestrator
                           |
        +------------------+------------------+
        |                  |                  |
+-------+------+  +--------+------+  +--------+------+
| Traditional  |  |  Agent Step   |  | Traditional  |
|    Step      |  |               |  |    Step      |
|              |  | Input:  diff  |  |              |
| make build   |  | Output: report|  | make deploy  |
| exit: 0/1    |  | exit:   0/1   |  | exit: 0/1    |
+--------------+  +---------------+  +--------------+
```

This contract-based approach means that an agent step can be replaced with a
traditional step (or vice versa) without changing the pipeline definition, as
long as the input/output contract is preserved.

## Related

- [overview.md](overview.md)
- [agent_harness.md](agent_harness.md)
- [trust_and_security.md](trust_and_security.md)
- [AI Glossary: Async Agents](../../reference/ai_glossary.md)
- [AI Glossary: Human-in-the-Loop](../../reference/ai_glossary.md)
- [AI Glossary: Workflow vs Agent](../../reference/ai_glossary.md)

## Links

- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
