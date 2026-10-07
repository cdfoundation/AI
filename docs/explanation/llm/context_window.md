# Context Window

## Overview

A model context window is the maximum amount of information—measured in
tokens—that a Large Language Model (LLM) can "see," process, and remember during
a single interaction. It functions as the AI’s working memory. This window
encompasses everything in the current session: the system instructions, the
user's prompt, the conversation history, and any attached documents or data.

When this limit is reached, the model must "forget" earlier parts of the
conversation to accommodate new input, typically operating on a "first-in,
first-out" basis. Alternatively, the interaction may simply fail if the input
exceeds the model's absolute limit.

## What are Tokens?

LLMs do not read text as words or characters like humans do. Instead, they
process text as **tokens**.

- **Tokenization**: This is the process of converting text into numerical IDs.
- **Size**: A token can represent a single character, a partial word, or a whole
  word.
  - **Approximation**: 1 token is roughly 0.75 words (100 tokens is roughly 75
    words).
  - **Approximation**: 1 token is roughly 4 characters.
- **Variability**: Complex words or different languages (like Telugu) may
  require more tokens to represent the same amount of information compared to
  simple English text.

## Components of the Context Window

The context window is a shared resource consumed by several elements:

1. **System Prompt**: The hidden instructions that define the AI's persona and
   rules.
2. **Conversation History**: The previous questions and answers in the chat
   session.
3. **New Input**: The user's current query and any uploaded files or RAG
   (Retrieval-Augmented Generation) data.
4. **Output Reservation**: The model needs space to generate its response. If
   the input fills 99% of the window, the model can only generate a very short
   reply.

The total context usage is the sum of all four components: system prompt,
conversation history, new input, and the space reserved for the generated
output. When any one component grows, the others must shrink or the interaction
will be truncated or fail entirely.

## The Mechanics: Why Limits Exist

Most modern LLMs utilize the **Transformer architecture**. This relies on a
mechanism called **self-attention**, which calculates the relationships between
every token and every other token in the sequence.

- **Quadratic Cost**: As the number of tokens doubles, the computational power
  required increases roughly fourfold. This makes extremely large context
  windows computationally expensive and slower (higher latency).
- **Architecture**: The specific model architecture defines the hard limit
  (e.g., 8,192 tokens for GPT-4 original, 128k for GPT-4o).

## Context Window Sizes

Context windows generally fall into two categories:

- **Standard Context (<32k tokens)**: Sufficient for chat, short summaries, and
  coding tasks. These models are typically faster and cheaper.
- **Long Context (128k - 2M+ tokens)**: Capable of ingesting entire books,
  codebases, or complex legal documents.

| Category         | Approximate Range | Characteristics                                                                     |
| ---------------- | ----------------- | ----------------------------------------------------------------------------------- |
| Standard context | Up to 32k tokens  | Fast and cost-effective; suited for chat, short documents, and focused coding tasks |
| Long context     | 128k - 1M tokens  | Capable of ingesting entire codebases or long documents; higher latency and cost    |
| Extended context | 1M+ tokens        | Emerging capability; suited for very large document collections and deep analysis   |

Context window sizes vary by model and change rapidly as architectures improve.
For current limits, consult each provider's official model documentation.

## Challenges with Large Contexts

While larger windows are powerful, they introduce specific challenges:

### 1. "Lost in the Middle" Phenomenon

Models tend to be best at recalling information at the very beginning (system
prompt) and the very end (latest user question) of the context. Information
buried in the middle of a 100k token prompt is more likely to be overlooked or
hallucinated.

### 2. Information Density and Noise

Feeding a model too much irrelevant data ("noise") can decrease its reasoning
accuracy. More context is not always better if the context is low-quality.

### 3. Security

Longer contexts provide a larger attack surface for "jailbreaking" attempts,
where adversarial prompts try to override safety guardrails.

## Related

- [tokens.md](tokens.md) — Deep dive into how text is converted into tokens and
  how encoding schemes affect token counts
- [context_pollution.md](context_pollution.md) — How a full or noisy context
  window degrades model reasoning
- [AI Glossary: Context Window](../../reference/ai_glossary.md) — Concise
  definition
- [AI Glossary: Context Rot](../../reference/ai_glossary.md) — Related failure
  mode
- [AI Glossary: Tokens](../../reference/ai_glossary.md) — Concise definition

## Links

- [IBM: What is a context window?](https://www.ibm.com/think/topics/context-window)
- [Transformer architecture (Attention Is All You Need)](https://arxiv.org/abs/1706.03762)
