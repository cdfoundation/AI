# Context Pollution

## Overview

Context pollution is the silent killer of AI agent performance. It refers to the
degradation of a Large Language Model's reasoning ability due to the presence of
irrelevant, redundant, or conflicting information within its working context
window.

Many developers assume that as long as they are under the model's maximum token
limit, the AI will function perfectly. The reality is that excessive, noisy data
pollutes the prompt. It causes the model to lose focus, fail to identify
relevant instructions, and produce confused or hallucinated outputs. In agentic
workflows, context pollution is the primary cause of infinite loops and
illogical actions.

## The Anatomy of Context Pollution

Context pollution manifests in several distinct ways during an AI session:

- Signal to noise ratio: As the sheer volume of information in the context
  increases, the model's ability to filter out noise decreases. Even if the data
  fits within the token limit, a massive block of irrelevant text can obscure
  the actual task.
- Context distraction: Irrelevant information explicitly disrupts the model's
  pattern matching. It makes it difficult for the AI to distinguish between
  important system instructions, user data, and structural markers.
- Context poisoning: A dangerous form of pollution that takes two forms. The
  first is accidental: a hallucination or error is introduced into the
  conversation history and the model treats its own past output as ground truth,
  compounding the error. The second is deliberate: an attacker crafts malicious
  input — via prompt injection, a poisoned document, or a compromised tool
  output — to override system instructions or manipulate the agent's reasoning.
  Both forms share the same root cause: the model cannot distinguish trusted
  instructions from untrusted content once both are in the same context window.
- Memory staleness: In long-running sessions, older and outdated information
  remains in the context. The model may anchor onto this obsolete data instead
  of using the current, relevant information provided later in the prompt.
- Context clashes: When the context contains conflicting instructions, the model
  struggles. LLMs are notoriously bad at subtractive reasoning, which is the
  ability to actively ignore a previous rule, and will instead try to blend the
  conflicting instructions together.

## The Ghost of Failing Tests

Consider a standard agentic workflow where an AI writes a feature and runs the
test suite. The tests fail, dumping five hundred lines of stack traces into the
terminal. The agent reads the error, writes a fix, and runs the tests again.
They fail again with a different error.

By the third attempt, the agent's context window is stuffed with thousands of
tokens of failed test output, broken code snippets, and incorrect assumptions.
Even if the agent eventually fixes the bug, all of that failure data remains in
the context window.

When you ask the agent to build the next feature, it has to read through its own
history of failures before it can process your new request. This creates a
massive cognitive drag. The model becomes confused by the ghosts of its past
mistakes, often reintroducing the exact bugs it just fixed because the broken
code is still sitting in its working memory.

## The Hallucination Echo Chamber

This problem compounds when the model makes logical errors. If an agent
hallucinates a non-existent API endpoint and writes code to call it, that
hallucination is now permanently etched into the chat history.

When you correct the agent, it apologizes and writes the correct code. But the
hallucinated API endpoint is still in the context window. Because LLMs use
attention mechanisms to predict the next token based on all previous tokens, the
presence of the bad code actively pulls the model's reasoning toward the wrong
answer.

The more mistakes the model makes, the more polluted the context becomes. It
creates an echo chamber of bad logic. The agent starts second-guessing its own
correct code, blending the hallucinated API with the real one, and spiraling
into a state of complete context rot.

## Mitigation Strategies

To combat context pollution, developers must practice rigorous context
engineering.

- Fresh starts: The most effective strategy. Wipe the conversation history
  completely and start a new, clean context for a new task. This prevents the
  leakage of irrelevant, outdated, or conflicting information.
- Context summarization: Use a faster, cheaper LLM to summarize past
  interactions and tool outputs, replacing thousands of tokens of raw logs with
  a concise paragraph.
- Context compaction: Actively strip out redundant, outdated, or irrelevant
  information from the history before sending the next prompt.
- Targeted retrieval: Instead of stuffing as much information as possible into
  the context window, use highly specific retrieval to ensure only high-signal
  information is included.

## Related

- [context_window.md](context_window.md) — The mechanics of the context window
  and why size limits exist
- [tokens.md](tokens.md) — How tokens are counted and why token waste is costly
- [AI Glossary: Context Rot](../../reference/ai_glossary.md) — Concise
  definition of the broader degradation phenomenon
- [AI Glossary: Context Stuffing](../../reference/ai_glossary.md) — The
  anti-pattern this document argues against
- [AI Glossary: Prompt Injection](../../reference/ai_glossary.md) — The
  deliberate form of context poisoning
- [AI Glossary: Hallucination](../../reference/ai_glossary.md) — Root cause of
  accidental context poisoning

## Links

- [OWASP GenAI Security Project: Prompt Injection](https://genai.owasp.org/)
- [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)

## Practical Example: The Grep Trap

One of the most common ways developers accidentally pollute an agent's context
window is through careless file searching.

When an agent needs to find a function in a repository, it might try to use a
standard grep command.

```bash
# The forbidden pattern
grep -r "def authenticate" .
```

Running a bare recursive grep on the repository root is a disaster for the
context window. It will recurse into target directories, compiled binaries,
virtual environments, and massive dependency folders. The agent's context window
will instantly fill with thousands of lines of minified JavaScript and compiled
bytecode. The signal to noise ratio drops to zero, and the agent suffers
immediate context rot.

The fix is to scope searches precisely: limit by file type, restrict to relevant
directory subtrees, and return only the signal the agent needs. Tools that
auto-respect ignore files (like `ripgrep`) and search interfaces that return
structured, filtered results are preferable to raw recursive searches over the
entire repository. The specific commands and project rules for enforcing this
belong in a how-to guide or in the project's `AGENTS.md`.
