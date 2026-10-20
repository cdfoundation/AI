# Tokens

Large Language Models do not read text directly. They process sequences of
integers. A token is the unit of text a model works with, and a tokenizer
converts between text and token IDs.

## Byte Pair Encoding

Most modern models use a tokenizer based on Byte Pair Encoding (BPE). BPE starts
from individual bytes or characters and iteratively merges the most frequently
occurring adjacent pairs into single tokens, based on statistics from the
tokenizer's training data.

Common words such as "apple" typically become a single token. Rare or complex
words are split into smaller known pieces; "unbreakable" might become "un",
"break", and "able".

Because any string can fall back to individual bytes, BPE can represent any
text, including typos, slang, and base64 encoded strings, while keeping the
vocabulary to roughly 50,000 to 200,000 unique tokens.

## The Mathematics of Language

Tokens exist for two reasons: they keep the vocabulary compact, and neural
networks operate on numbers rather than text.

Every unique token is assigned an integer ID. The word "Hello" might be ID
15496, while " world" with a leading space and a lowercase w might be 995 (IDs
from `r50k_base`; other encodings assign different IDs). When a prompt is
submitted, the tokenizer translates the text into an array of these integers.
The model maps the integers into high-dimensional vectors, computes the
probabilities of which token should come next, and outputs a new integer. The
tokenizer translates that final integer back into text.

## Sizing Up the Context Window

Tokens determine how much a model can process in one request and how much usage
costs, since providers typically bill per token.

As a general rule for English text, one token is roughly four characters or
three-quarters of a word: 100 tokens is about 75 words and 1,000 tokens is about
750 words. Non-English languages and heavily formatted code usually need more
tokens for the same content because they compress less efficiently under Byte
Pair Encoding.

Every model also has a context window: the limit on how many tokens it can
process in one request, covering both the input prompt and the generated output.
If the input and the reserved output together would exceed the limit, the API
typically rejects the request. Applications stay within the limit by choosing
what to send on each turn, commonly by dropping the oldest messages first or by
summarizing them (compaction). The model does not forget on its own; it only
sees what the application sends.

## Character-Level Tasks

Because LLMs operate on token IDs instead of individual letters, they can
struggle with tasks that require character-level awareness.

Asking an older LLM to count the number of "r"s in "strawberry" often fails.
Newer models with stronger reasoning capabilities handle this better, but the
underlying tokenization constraint remains: the model sees token IDs, not the
letters inside them.

Number handling shows the same effect. Older tokenizers split numbers
inconsistently; the number 12345 might become "12" and "345", which makes
digit-level arithmetic harder. Newer tokenizers split numbers into consistent
blocks or single digits.

## The Cost of Doing Business

Tokens are the unit for context limits and billing. In agentic workflows, track
token usage per request, stay within the context window, and use the provider's
tokenizer when counting precisely.

## Encoding Methods

OpenAI has released four main encoding schemes through the
[tiktoken](https://github.com/openai/tiktoken) library. Each encodes the same
text differently, and knowing which one a model uses matters when you are
counting tokens precisely or building tooling around a specific API. Other
providers use different tokenizers; see Other Tokenizers below.

### r50k_base

The oldest encoding, derived directly from GPT-2. It has a vocabulary of 50,257
tokens and uses a simple regex pattern that splits on whitespace, contractions,
letters, numbers, and punctuation. Numbers are not forced to tokenize
consistently. Used by the original GPT-2 and legacy Davinci, Curie, Babbage, and
Ada completion models. Largely obsolete for new work.

### p50k_base

A modest update to `r50k_base` that expanded the vocabulary to 50,281 tokens by
adding 24 tokens that represent runs of 2 to 25 spaces, which encode code
indentation more compactly. It shares the same regex splitting pattern as
`r50k_base` but produces slightly different token boundaries for some inputs.
The `p50k_edit` variant adds three Fill-in-the-Middle special tokens
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
efficiently than its predecessors.

Special tokens include `<|endoftext|>`, FIM tokens for fill-in-the-middle, and
`<|endofprompt|>`.

### o200k_base

The current encoding, used by GPT-4o, the o1/o3/o4 reasoning models, and GPT-5.
Its vocabulary expands to 200,019 tokens including special tokens (about 199,998
mergeable tokens), roughly double `cl100k_base` and about four times
`r50k_base`. The larger vocabulary means longer common words and phrases can be
represented as a single token, reducing overall token counts for dense prose and
compressing code even further.

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

When selecting an encoding for token counting in your own tooling with OpenAI
models, use `cl100k_base` for any GPT-3.5 or GPT-4 series model, and
`o200k_base` for GPT-4o and newer. Using the wrong encoding will produce
incorrect counts and can cause silent context-window overflows in production
pipelines.

## Other Tokenizers

OpenAI's encodings are one family among many. Other providers train their own
tokenizers, so the same text produces different token IDs and different token
counts depending on the model. Use the provider's own tokenizer or token
counting tool when counting precisely.

### Tokenization Algorithms

- **Byte-level BPE**: BPE with the 256 byte values as the base vocabulary, so
  any text can be encoded without an unknown token. Used by the OpenAI encodings
  above and many open-weight models.
- **WordPiece**: Like BPE, it merges pairs from the bottom up, but it picks the
  pair that maximizes the likelihood of the training data rather than the most
  frequent pair. Used by BERT-family models.
- **Unigram**: Starts from a large candidate vocabulary and prunes the tokens
  that contribute least to the training-data likelihood. A word can be tokenized
  several ways, and the most probable one is chosen. Used by T5-family models.
- **SentencePiece**: A library that applies BPE or Unigram directly to raw text
  and treats whitespace as a symbol (written as "▁"). It does not assume spaces
  separate words, so it suits languages such as Chinese and Japanese.

### Examples from Open-Weight Families

| Family  | Tokenizer                                                          | Approximate vocabulary |
| ------- | ------------------------------------------------------------------ | ---------------------- |
| Llama   | SentencePiece BPE in earlier generations; tiktoken-based BPE later | 32,000, then 128,256   |
| Mistral | SentencePiece in earlier models; Tekken (tiktoken-based) in newer  | 131,072 for Tekken     |
| Gemma   | SentencePiece                                                      | about 256,000          |
| Qwen    | Byte-level BPE                                                     | about 151,000          |

Vocabularies have grown from tens of thousands of tokens to well over 100,000.
Larger vocabularies generally compress non-English text and code better, which
reduces the non-English token tax. Mistral reports that Tekken compresses source
code and many languages more efficiently than its earlier SentencePiece
tokenizer. Tokenizers change between model generations, so confirm the tokenizer
on the model card.

### Hosted Models

Not every provider publishes its tokenizer. Some offer a token counting API
instead; Anthropic, for example, provides a `count_tokens` endpoint for Claude
models. Counts from these services are estimates and can change when the
provider updates its tokenizer.

## Related

- [context_window.md](context_window.md) — How token limits define the model's
  working memory and what happens when those limits are reached
- [context_pollution.md](context_pollution.md) — How token waste from noise and
  redundancy degrades model reasoning
- [AI Glossary: Tokens](../../reference/ai_glossary.md#tokens) — Concise
  definition
- [AI Glossary: BPE](../../reference/ai_glossary.md#bpe-byte-pair-encoding) —
  Concise definition
- [AI Glossary: Non-English Token Tax](../../reference/ai_glossary.md#non-english-token-tax)
  — Related concept

## Links

- [tiktoken (OpenAI tokenizer library)](https://github.com/openai/tiktoken)
- [OpenAI tokenizer playground](https://platform.openai.com/tokenizer)
- [BPE original paper: Neural Machine Translation of Rare Words with Subword Units](https://arxiv.org/abs/1508.07909)
