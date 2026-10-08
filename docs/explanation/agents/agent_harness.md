# Agent Harness Architecture

## Overview

An **Agent Harness** is the runtime environment that wraps a Large Language
Model (LLM) to transform it from a text predictor into an autonomous agent.
While the LLM provides the "brain" (reasoning and planning), the harness
provides the "body" (execution, senses, and constraints).

Without a harness, an LLM is a passive entity that waits for input and generates
text. With a harness, an LLM can:

1. **Retain Memory**: Maintain state and context over long-running tasks.
2. **Interact with the World**: Execute shell commands, read/write files, and
   make network requests.
3. **Plan and Iterate**: Run in a loop, observing the output of its actions and
   correcting its course.
4. **Operate Safely**: Run within a sandboxed environment with strict access
   controls.

## High-Level Architecture

The harness sits between the user and the raw LLM API. It manages the flow of
information, injects system prompts, parses tool calls, and executes actions.

```text
+----------------------------------------------------------------------+
|                           CLI / User Interface                       |
|   Commands: chat | run | auth | models | skills                     |
+-------------------------------+--------------------------------------+
                                |
                     +----------v-----------+
                     |    Configuration      |
                     |   (YAML + env vars)   |
                     +----------+-----------+
                                |
          +---------------------+---------------------+
          |                                           |
+---------v----------+                     +----------v---------+
|   Chat Mode (REPL) |                     |   Run Mode (Auto)  |
|  Interactive Loop  |                     |  Plan Execution    |
+---------+----------+                     +----------+---------+
          |                                           |
          +---------------------+---------------------+
                                |
                     +----------v-----------+
                     |    Execution Engine   |
                     |   - Conversation mgmt |
                     |   - Iteration control |
                     |   - State machine     |
                     +----------+-----------+
                                |
               +----------------+----------------+
               |                                 |
    +----------v-----------+          +----------v-----------+
    |   Provider Layer      |          |     Tool Registry    |
    |   (LLM Abstraction)   |          |   - Tool dispatch    |
    |                       |          |   - Schema export    |
    |   +---------------+  |          |   +---------------+  |
    |   | OpenAI / GPT  |  |          |   | File System   |  |
    |   +---------------+  |          |   | Terminal      |  |
    |   | Anthropic     |  |          |   | Network/Http  |  |
    |   +---------------+  |          |   | Sub-agents    |  |
    |                       |          |   +---------------+  |
    +-----------------------+          +-----------------------+
               |                                 |
    +----------v-----------+          +----------v-----------+
    |   Security Layer      |          |   Skills System      |
    |   - Command Validator |          |   - Discovery        |
    |   - Path Validator    |          |   - Registry         |
    |   - Permission Mgr    |          |   - Prompt Injection |
    |   - Tool Approval     |          |                      |
    +-----------------------+          +-----------------------+
```

## The Execution Loop

The core of an agent harness is the **Execution Loop**. Unlike a standard
chatbot that responds once, an agent harness runs a "Thought-Action-Observation"
loop.

### State Machine

To prevent the agent from getting stuck or running indefinitely, the harness
implements a strict state machine:

```text
State Flow:
  Idle --> WaitingForUser --> Thinking --> ExecutingTool --> Completed
                                  |                            ^
                                  +----> Failed                |
                                  +----------------------------+
```

1. **Idle**: System is initialized.
2. **Thinking**: The harness packages the conversation history and available
   tools, sending them to the LLM.
3. **ExecutingTool**: The LLM has requested an action (e.g., "read file"). The
   harness intercepts this request, validates security, executes the code, and
   captures the output.
4. **Observation**: The output (stdout/stderr) is fed back into the context
   window as a new message.
5. **Loop**: The cycle repeats until the LLM determines the task is complete or
   a limit (iterations/tokens) is reached.

### Workflow Diagram

```text
User Input
    |
    v
+-------------------+     +-----------------------+
| Mention Parser    |---->| Context Augmentation  |
| (expand references)|    | Inject @file content  |
+-------------------+     +-----------+-----------+
                                      |
                                      v
                          +-----------+-----------+
                          |   Engine Start        |
                          |   Reset Iterations    |
                          +-----------+-----------+
                                      |
                          +-----------v-----------+
                          |      Thinking         |
                          | Send history + tools  |
                          | to LLM Provider       |
                          +-----------+-----------+
                                      |
                          +-----------v-----------+
                          | Parse LLM Response    |
                          +----+--------+---------+
                               |        |
                  +------------+        +-------------+
                  | has_tool_calls?     | text only?   |
                  v                     v              |
     +------------+-------+  +---------+---------+    |
     | State = Executing  |  | State = Completed |    |
     | For each ToolCall: |  | Return text to    |    |
     |   1. Security Check|  | user / caller     |    |
     |   2. Execute Tool  |  +-------------------+    |
     |   3. Capture Output|                           |
     +------------+-------+                           |
                  |                                    |
                  v                                    |
     +------------+-------+                           |
     | Append Tool Result |                           |
     | to Conversation    |                           |
     +------------+-------+                           |
                  |                                    |
                  +---> Loop back to Thinking ->-------+
                        (Increment Iteration Count)
```

## Security: Keeping the Agent in Check

An autonomous agent is effectively remote code execution (RCE) as a feature.
Without a robust security layer, an LLM can delete files, exfiltrate data, or
crash the host system.

A production-grade harness must implement the following security gates:

### 1. Command Validation (The "sudo" Gate)

The agent should never have unfettered access to the shell. The harness must
inspect every command before execution.

```text
+---------------------+
| Command Validator   |
+---------------------+
| Inputs:             |
|   - Command String  |
|   - User Config     |
+---------------------+
| Logic:              |
| 1. Check Deny List  | (e.g., rm -rf /, shutdown, mkfs)
| 2. Check Allow List | (e.g., ls, cat, grep)
| 3. Check Patterns   | (Regex matching for dangerous flags)
+---------------------+
| Output:             |
| -> Allowed          |
| -> Requires Confirm | (Ask human permission)
| -> Blocked          | (Throw error to agent)
+---------------------+
```

### 2. Path Validation (The Sandbox)

File system access must be restricted to a specific working directory ("The
Sandbox"). The harness must prevent **Directory Traversal Attacks** where the
LLM tries to access `../../etc/passwd`.

- **Canonicalization**: Resolve all paths to their absolute form.
- **Prefix Check**: Ensure the resolved path starts with the allowed sandbox
  root.
- **Symlink Protection**: Detect if a symlink inside the sandbox points to a
  target outside the sandbox.

### 3. Human-in-the-Loop (Tool Approval)

For sensitive operations, the harness should pause execution and require
explicit user approval.

- **Interactive Mode**: The user is prompted [Y/n] for every tool call.
- **Autonomous Mode**: Safe tools (read) are auto-approved; dangerous tools
  (write/execute) require confirmation.
- **Full Auto**: All tools are auto-approved (Use with extreme caution).

### 4. Network Safety (SSRF Protection)

If the agent has a `fetch` tool, it must be protected against **Server-Side
Request Forgery (SSRF)**.

- **Block Localhost**: Prevent the agent from scanning internal ports
  (`127.0.0.1`, `localhost`).
- **Block Private Ranges**: Deny access to `192.168.x.x`, `10.x.x.x`, etc.
- **Protocol Restriction**: Allow only `http` and `https` (no `file://` or
  `gopher://`).

### 5. Resource Limits

To prevent "runaway" agents that consume infinite resources:

- **Iteration Limit**: Hard cap on the number of thought loops (e.g., max 50
  steps).
- **Context Limit**: FIFO pruning of conversation history to stay within token
  windows.
- **Output Limit**: Truncate tool outputs (e.g., max 1MB) to prevent flooding
  the context window with massive log files.
- **Execution Timeout**: Kill long-running shell commands after $N$ seconds.

## Tooling and Capabilities

The harness exposes capabilities to the LLM via a **Tool Registry**. Tools are
defined by a JSON schema that describes the function signature, arguments, and
return types.

Common Standard Library Tools:

**File System**:

- `read_file(path)`: Read content with line numbers.
- `write_file(path, content)`: Create or overwrite files.
- `list_directory(path)`: Explore project structure.
- `grep(pattern, path)`: Search codebases efficiently.

**Terminal**:

- `execute_command(cmd)`: Run shell commands (sandboxed).

**Network**:

- `fetch_url(url)`: Scrape documentation or API results.

**Meta**:

- `task_complete(result)`: Signal that the user's request is finished.
- `ask_user(question)`: Request clarification from the human.

## Summary

An Agent Harness transforms a stochastic model into a deterministic application.
It provides the **Safety**, **Memory**, and **Tools** necessary for an LLM to
perform useful work in the real world.

The most critical component is the **Security Layer**. An agent harness without
security is just a vulnerability waiting to be exploited. By implementing strict
path validation, command filtering, and human-in-the-loop confirmation, we can
safely leverage the power of autonomous agents.
