# Agents Overview

## Overview

An **AI agent** is a system that uses a Large Language Model to reason about a
goal and take a sequence of actions to accomplish it. Unlike a standard LLM
interaction — where a model receives a prompt and generates a single response —
an agent runs in a loop: it observes its environment, decides what to do next,
executes an action, observes the result, and repeats until the task is complete
or a limit is reached.

The definition of "agent" is contested in the field. Some reserve the term for
fully autonomous systems with minimal human oversight. Others use it for any
LLM-powered system that calls tools. For the purposes of this SIG, an agent is
any LLM-based system that:

1. Has access to one or more **tools** it can invoke autonomously.
2. Operates in a **loop**, using tool outputs to inform subsequent decisions.
3. Pursues a **goal** across multiple steps rather than responding once.

## The Agent Spectrum

Agents exist on a spectrum from simple to fully autonomous. Most production
systems sit somewhere in the middle.

```text
Simple                                                         Autonomous
  |                                                                 |
  v                                                                 v
+-------------+  +--------------+  +----------------+  +----------+
| Single-turn |  | Tool-calling |  |  ReAct / Plan  |  |  Multi-  |
| Completion  |  |   (1 step)   |  |  and Execute   |  |  Agent   |
|             |  |              |  |  (N steps)     |  |  System  |
| "Summarize  |  | "Search for  |  | "Implement     |  | Network  |
|  this doc"  |  |  X and reply"|  |  this feature" |  | of agents|
+-------------+  +--------------+  +----------------+  +----------+
     No tools       One tool call    Many tool calls    Many agents
     No loop        No loop          Looping             Looping +
                                                         delegation
```

**Single-turn completion** is not an agent. It is a prompt and a response.

**Tool-calling (single step)** is the simplest agent pattern: the model receives
a query, decides to call a tool once, and returns the result. Most LLM APIs
support this natively via function calling.

**ReAct / Plan and Execute** is the pattern most practitioners mean when they
say "agent." The model reasons about a goal, executes a sequence of tool calls,
observes results, adjusts its plan, and continues until it determines the task
is done. This is the pattern implemented by the harness described in
[agent_harness.md](agent_harness.md).

**Multi-agent systems** involve multiple LLM-driven agents working together,
with one agent (the orchestrator) delegating subtasks to others (subagents).
Covered in [multi_agent.md](multi_agent.md).

## The Thought-Action-Observation Loop

The core execution pattern of a ReAct agent is the **Thought-Action-Observation
loop**. At each iteration the agent:

```text
+-------------------+
|      Thinking     |  <-- LLM reasons about the current state and decides
|  "I need to read  |      what action to take next
|   the config file"|
+--------+----------+
         |
         v
+--------+----------+
|      Action       |  <-- The harness executes the chosen tool call
|  read_file(       |
|   "config.yaml")  |
+--------+----------+
         |
         v
+--------+----------+
|    Observation    |  <-- The tool result is fed back into context
|  "db_host: ..."   |      The agent sees what happened and thinks again
+--------+----------+
         |
         +---> repeat until task complete or limit reached
```

This loop is what distinguishes an agent from a single LLM call. The agent
adapts its plan based on what it observes, allowing it to handle tasks that
cannot be fully specified in advance.

## How Agents Differ from Traditional Pipeline Steps

Traditional CI/CD pipeline steps are deterministic and explicit. A step runs a
defined command, produces a known artifact type, and either passes or fails
based on a clear exit code. The behavior of a step is fully specified before it
runs.

An agent step is different in every one of these properties:

| Property        | Traditional Step                | Agent Step                                                    |
| --------------- | ------------------------------- | ------------------------------------------------------------- |
| Behavior        | Fully specified in advance      | Determined at runtime by the LLM                              |
| Output          | Predictable artifact type       | Variable, depends on task and context                         |
| Error handling  | Defined exit codes              | LLM decides whether to retry or fail                          |
| Auditability    | Command and output logged       | Reasoning trace must be explicitly captured                   |
| Reproducibility | Same input produces same output | Non-deterministic; same input may produce different actions   |
| Resource usage  | Known at pipeline design time   | Unpredictable; depends on how many iterations the agent takes |

These differences do not make agents worse than traditional pipeline steps —
they make agents suited to different kinds of tasks. An agent is appropriate
when the steps needed to accomplish a goal cannot be fully enumerated in
advance. A traditional step is appropriate when they can.

## What Agents Enable in CI/CD

Agents extend what a CI/CD pipeline can do beyond deterministic automation:

- **Adaptive code review**: An agent can read a pull request, understand its
  context, identify issues that simple linters cannot catch, and generate
  structured feedback — adjusting its analysis based on what it finds.
- **Automated remediation**: When a pipeline step fails, an agent can read the
  error, identify the cause, propose a fix, apply it, and re-run the step —
  rather than simply reporting failure.
- **Documentation generation**: An agent can traverse a codebase, understand the
  intent of components, and produce documentation that reflects what the code
  actually does.
- **Dependency analysis**: An agent can reason about the implications of a
  dependency update across a codebase, identifying breaking changes that static
  analysis tools miss.

## What Agents Do Not Replace

Agents are not a replacement for deterministic pipeline steps. A step that can
be expressed as a command should be expressed as a command. Agents introduce
non-determinism, latency, token cost, and security surface area that
deterministic steps do not. The right question before adding an agent to a
pipeline is: "Can this be done without an LLM?" If the answer is yes, it should
be.

## Related

- [agent_harness.md](agent_harness.md)
- [pipeline_patterns.md](pipeline_patterns.md)
- [multi_agent.md](multi_agent.md)
- [trust_and_security.md](trust_and_security.md)
- [AI Glossary: Agent](../../reference/ai_glossary.md)
- [AI Glossary: Tool Use](../../reference/ai_glossary.md)
- [AI Glossary: Workflow vs Agent](../../reference/ai_glossary.md)
- [AI Glossary: Human-in-the-Loop](../../reference/ai_glossary.md)

## Links

- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
- [Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761)
