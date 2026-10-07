# Tokens

One would like to imagine a Large Language Model sitting in a digital library,
reading our prompts word by word like a scholar. Let's be honest. That is a
complete fabrication. LLMs do not read text. They process sequences of numbers.
To bridge the gap between the messy reality of human language and the rigid
requirements of mathematical computation, we use tokens.

## The Byte Pair Encoding Meat Grinder

Think of a tokenizer as a highly specific, statistically driven meat grinder for
language. Modern models use a technique called Byte Pair Encoding. It starts by
looking at individual characters and bytes, then iteratively merges the most
frequently occurring pairs into single tokens based on the training data.

If you feed it a common word like "apple", the tokenizer has seen that exact
sequence of characters so many times that it assigns it a single token ID. But
if you feed it a rare or complex word like "unbreakable", it falls back to the
subwords it knows, chopping it into "un", "break", and "able".

It is an elegant tokenization algorithm. It keeps the model's vocabulary size
manageable, usually around fifty thousand to two hundred thousand unique tokens,
while ensuring it can represent literally any string of text, including typos,
slang, and base64 encoded strings.

## The Mathematics of Language

The canned answer for why we use tokens is efficiency. The real answer is that
neural networks are fundamentally massive matrices of weights. They need numbers
to do their job.

Every unique token gets assigned a specific integer ID. The word "Hello" might
be ID 15496, while a trailing space and the word "World" might be 995. When you
submit a prompt, the system translates your text into an array of these
integers. The model then maps these integers into high-dimensional vector
spaces, crunches the probabilities of what token should come next, and spits out
a new integer. The tokenizer translates that final integer back into text. It is
a constant translation layer operating under the hood.

## Sizing Up the Context Window

You need to understand tokens because they are the fundamental currency of AI.
They dictate how much a model can remember and how much you are going to pay the
API provider.

As a general rule for English text, one token is roughly four characters or
three-quarters of a word. A hundred tokens will get you about seventy-five
words. If you are working in non-English languages, or writing heavily formatted
code, the token tax goes up significantly because those characters do not
compress as neatly in the Byte Pair Encoding process.

This matters because every LLM has a context window. This is the strict memory
limit of the model, encompassing both your input prompt and the generated
output. If you have a model with a 32,000 token limit, and your conversation
exceeds that boundary, the model simply drops the oldest tokens. It is a sliding
window of memory. Once you push new tokens in, the old ones fall out the back,
and the model loses the original context of your architecture decisions.

## Why Your AI Fails at Basic Spelling

Here is where the reality of tokens bites you in production. Because LLMs
operate on token IDs instead of individual letters, they are notoriously
terrible at tasks that require character-level awareness.

If you ask an older LLM to count the number of "r"s in "strawberry", it will
often fail. Newer models with stronger reasoning capabilities handle this
better, but the underlying tokenization constraint remains. It does not see the
letters in the word. It sees a single token ID representing the concept of the
fruit. Asking it to count the letters is like asking you to count the number of
threads in a sweater while you are wearing it.

The same applies to math. Historically, numbers were tokenized inconsistently.
The number 12345 might be chopped into "12" and "345". When the model tries to
perform arithmetic on these fragmented concepts, it confabulates the wrong
answer. Newer tokenizers have started forcing numbers to tokenize digit by digit
or in consistent blocks to fix this exact problem, but the underlying limitation
remains. The model is doing math on concepts of numbers, not the digits
themselves.

## The Cost of Doing Business

Tokens are the building blocks of the AI revolution. They are how we force human
language into a neural network. When you build agentic workflows, you have to
manage this currency carefully. Understand how Byte Pair Encoding works, respect
the limits of the context window, and remember that every time you hit enter,
you are paying for the translation.

## Encoding Methods

OpenAI has released four main encoding schemes through the
[tiktoken](https://github.com/openai/tiktoken) library. Each encodes the same
text differently, and knowing which one a model uses matters when you are
counting tokens precisely or building tooling around a specific API.

### r50k_base

The oldest encoding, derived directly from GPT-2. It has a vocabulary of 50,257
tokens and uses a simple regex pattern that splits on whitespace, contractions,
letters, numbers, and punctuation. Numbers are not forced to tokenize
consistently, which contributes to the arithmetic failures described above. Used
by the original GPT-2 and legacy Davinci, Curie, Babbage, and Ada completion
models. Largely obsolete for new work.

### p50k_base

A modest update to `r50k_base` that expanded the vocabulary to 50,281 tokens by
adding 25 additional merge operations. It shares the same regex splitting
pattern as `r50k_base` but produces slightly different token boundaries for some
inputs. The `p50k_edit` variant adds three Fill-in-the-Middle special tokens
(`<|fim_prefix|>`, `<|fim_middle|>`, `<|fim_suffix|>`) used by the Codex edit
models. `p50k_base` was used by `text-davinci-002`, `text-davinci-003`, and the
Codex code completion models. All are now deprecated.

### cl100k_base

The encoding used by GPT-3.5 Turbo, GPT-4, and the current text embedding models
(`text-embedding-ada-002`, `text-embedding-3-small`, `text-embedding-3-large`).
Its vocabulary nearly doubles the older encodings at 100,277 tokens. The key
improvement is a significantly more sophisticated regex pattern that:

- Handles contractions case-insensitively
- Caps number runs at three digits per token, making arithmetic more consistent
- Treats newlines and whitespace more carefully, producing cleaner splits for
  code and structured text

These changes mean `cl100k_base` tokenizes code and multi-lingual text more
efficiently than its predecessors, and it is why GPT-4 tends to perform better
on reasoning tasks that involve numbers.

Special tokens include `<|endoftext|>`, FIM tokens for fill-in-the-middle, and
`<|endofprompt|>`.

### o200k_base

The current encoding, used by GPT-4o, the o1/o3/o4 reasoning models, and GPT-5.
Its vocabulary expands to 200,019 mergeable tokens — a fourfold increase over
`cl100k_base`. The larger vocabulary means longer common words and phrases can
be represented as a single token, reducing overall token counts for dense prose
and compressing code even further.

The regex pattern was rewritten to handle Unicode letter categories explicitly
(`\p{Lu}`, `\p{Ll}`, `\p{Lo}`, etc.) rather than relying on the broad `\p{L}`
class. This improves tokenization quality across non-Latin scripts such as
Arabic, Chinese, Japanese, Korean, and Devanagari. Number runs remain capped at
three digits, preserving the arithmetic improvement introduced in `cl100k_base`.

Special tokens are sparser: only `<|endoftext|>` (ID 199,999) and
`<|endofprompt|>` (ID 200,018) are defined in the base encoding.

### Comparison at a glance

| Encoding      | Vocabulary size | Number handling    | Models                           | Status     |
| ------------- | --------------- | ------------------ | -------------------------------- | ---------- |
| `r50k_base`   | 50,257          | Inconsistent       | GPT-2, legacy completion models  | Deprecated |
| `p50k_base`   | 50,281          | Inconsistent       | text-davinci-002/003, Codex      | Deprecated |
| `cl100k_base` | 100,277         | Capped at 3 digits | GPT-3.5 Turbo, GPT-4, embeddings | Current    |
| `o200k_base`  | 200,019         | Capped at 3 digits | GPT-4o, o1, o3, o4-mini, GPT-5   | Current    |

When selecting an encoding for token counting in your own tooling, use
`cl100k_base` for any GPT-3.5 or GPT-4 series model, and `o200k_base` for GPT-4o
and newer. Using the wrong encoding will produce incorrect counts and can cause
silent context-window overflows in production pipelines.

## Summary

### Why Do LLMs Use Tokens?

- Efficiency: Using subwords, the model keeps a reasonable vocabulary size
  (e.g., 50,000–100,000 unique tokens) while representing any word in the
  language, including misspelled or new words.
- Mathematical Processing: Neural networks understand numbers, not text. Each
  token in a model’s vocabulary gets a unique numerical ID. The text "Hello"
  might be ID 15496, and "World" might be 995. The model computes based on these
  numbers.

### Key Rules of Thumb

English: Roughly 1 token equals:

- 4 characters or 0.75 words.
- 100 tokens: Roughly 75 words.
- 1,000 tokens: Roughly 750 words.

**Note:** Non-English languages often require more tokens per word than English.

## Related

- [context_window.md](context_window.md) — How token limits define the model's
  working memory and what happens when those limits are reached
- [context_pollution.md](context_pollution.md) — How token waste from noise and
  redundancy degrades model reasoning
- [AI Glossary: Tokens](../../reference/ai_glossary.md) — Concise definition
- [AI Glossary: BPE](../../reference/ai_glossary.md) — Concise definition
- [AI Glossary: Non-English Token Tax](../../reference/ai_glossary.md) — Related
  concept

## Links

- [tiktoken (OpenAI tokenizer library)](https://github.com/openai/tiktoken)
- [OpenAI tokenizer playground](https://platform.openai.com/tokenizer)
- [BPE original paper: Neural Machine Translation of Rare Words with Subword Units](https://arxiv.org/abs/1508.07909)
