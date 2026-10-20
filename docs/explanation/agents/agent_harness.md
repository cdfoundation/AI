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
information, composes system prompts, parses tool calls, and executes actions.

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
    |   - Permission Mgr    |          |   - Prompt Append    |
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
                                          text-only response
  Idle --> WaitingForUser --> Thinking -----------------------> Completed
                               |    ^
                  tool calls   |    |   tool result appended
                               v    |
                            ExecutingTool

  Thinking, ExecutingTool --> Failed  (error, or iteration/token limit)
```

1. **Idle**: System is initialized.
2. **WaitingForUser**: The harness is waiting for user input, such as the
   initial request, a clarification answer, or approval of a dangerous tool
   call.
3. **Thinking**: The harness packages the conversation history and available
   tools, sending them to the LLM.
4. **ExecutingTool**: The LLM has requested an action (e.g., "read file"). The
   harness intercepts this request, validates security, executes the code, and
   captures the output. The output (stdout/stderr) is appended to the context
   window as a new message, the observation, and the harness returns to
   Thinking.
5. **Completed**: The LLM returned a text-only response with no tool calls, and
   the result is returned to the user or caller.
6. **Failed**: An unrecoverable error occurred or a limit (iterations or tokens)
   was reached, and the loop stops.

The Thought-Action-Observation loop is the Thinking to ExecutingTool to Thinking
cycle. It repeats until the state is Completed or Failed.

### Workflow Diagram

```text
User Input
    |
    v
+-------------------+     +-----------------------+
| Mention Parser    |---->| Context Augmentation  |
| (expand references)|    | Insert @file content  |
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
                  +------------+        +--------------+
                  | has_tool_calls?     | text only?   |
                  v                     v              |
     +------------+---------+ +---------+---------+    |
     | State = ExecutingTool| | State = Completed |    |
     | For each ToolCall:   | | Return text to    |    |
     |   1. Security Check  | | user / caller     |    |
     |   2. Execute Tool    | +-------------------+    |
     |   3. Capture Output  |                          |
     +------------+---------+                          |
                  |                                    |
                  v                                    |
     +------------+---------+                          |
     | Append Tool Result   |                          |
     | to Conversation      |                          |
     +------------+---------+                          |
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

Allow and deny lists on command strings are defence in depth, not a security
boundary. String matching is easy to get around: `bash -c "..."`, shell
expansion and substitution (`$(...)`, backticks, variables), and command
chaining can all change what actually runs without matching a denied string. The
actual boundary is OS-level isolation: run commands in a container or equivalent
sandbox, as an unprivileged user, with no ambient credentials such as cloud
tokens, SSH keys, or secrets in the environment. Treat the validator as a way to
catch mistakes and obvious misuse, and assume an injected or determined agent
can get around it. As with other controls, prefer a structural limit over a
policy that can be bypassed; see
[Structural Boundaries vs Policy](trust_and_security.md#structural-boundaries-vs-policy).

### 2. Path Validation (The Sandbox)

File system access must be restricted to a specific working directory ("The
Sandbox"). The harness must prevent **Directory Traversal Attacks** where the
LLM tries to access `../../etc/passwd`.

- **Canonicalization**: Resolve all paths to their absolute form.
- **Prefix Check**: Ensure the resolved path starts with the allowed sandbox
  root.
- **Symlink Protection**: Detect if a symlink inside the sandbox points to a
  target outside the sandbox.

Like command validation, these checks are defence in depth; the boundary is
OS-level isolation such as read-only mounts and a container that only sees the
working directory.

### 3. Human-in-the-Loop (Tool Approval)

For sensitive operations, the harness should pause execution and require
explicit user approval.

- **Interactive Mode**: The user is prompted [y/N] for every tool call. If the
  user just presses Enter, the call is denied.
- **Autonomous Mode**: Safe tools (read) are auto-approved; dangerous tools
  (write/execute) require confirmation.
- **Full Auto**: All tools are auto-approved (Use with extreme caution).

### 4. Network Safety (SSRF Protection)

If the agent has a `fetch` tool, it must be protected against **Server-Side
Request Forgery (SSRF)**. The harness enforces this in its execution engine, so
individual tools cannot skip it. Required controls include blocking loopback
addresses (IPv4 and IPv6), private ranges, and cloud metadata endpoints;
restricting protocols to HTTP and HTTPS; and validating the resolved IP address
after DNS resolution on every request, including redirects. On CI runners the
metadata endpoint is the main route to credentials. See
[Network Safety](trust_and_security.md#network-safety) for the full list and the
DNS rebinding rationale.

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

An Agent Harness makes a stochastic model bounded and auditable. The model's
individual steps remain non-deterministic, but the harness constrains what they
can do through validation, approval gates, and resource limits, and it is the
point where every tool call can be logged. It provides the **Safety**,
**Memory**, and **Tools** necessary for an LLM to perform useful work in the
real world. See [trust_and_security.md](trust_and_security.md) for audit
logging.

The most critical component is the **Security Layer**. An agent harness without
security is just a vulnerability waiting to be exploited. By implementing strict
path validation, command filtering, and human-in-the-loop confirmation, we can
safely leverage the power of autonomous agents.
