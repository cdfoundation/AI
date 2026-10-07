# Base Models and Fine-tuning

## The Adaptation Spectrum

When an organization wants to use an LLM for a specific task, it faces a
spectrum of adaptation options ranging from no model modification at all to
training a model from scratch. The choice is not primarily a technical one — it
is an economic one. The further along the spectrum, the more expensive, complex,
and time-consuming the path becomes, and the more operational burden the
organization takes on.

```text
Prompting --> RAG --> Fine-tuning --> Pre-training
  (lowest cost)                       (highest cost)
  (least control)                     (most control)
```

The right starting point is always as far left as possible. Most use cases can
be addressed through prompting alone. Many of the remainder can be addressed
through RAG. Fine-tuning should be considered only when the simpler approaches
have been genuinely exhausted and characterized, not as a first reflex when a
prompt does not immediately produce the desired behavior.

## Base Models

A base model is a large model trained on broad, general-purpose data that serves
as the starting point for downstream use. Base models are not optimized for any
specific task — a raw base model trained on next-token prediction will complete
text rather than follow instructions. The post-training alignment process
described in [overview.md](overview.md) — instruction fine-tuning and RLHF —
produces the instruction-following models that most practitioners encounter as
the starting point for their work.

Important: most commercial base models (such as Claude Opus, GPT-4o, or Gemini
Ultra) are not available for fine-tuning by external parties. When this document
refers to fine-tuning a base model, it refers to open-weight models (such as
Llama, Mistral, or Falcon) or models explicitly offered with fine-tuning APIs by
their providers. Before planning a fine-tuning project, verify whether the
specific model you intend to use supports it.

From a delivery perspective, a base model is a large binary artifact obtained
from an external source. It must be versioned, stored, and treated as a
dependency with the same rigor applied to any other upstream dependency: source
verification, integrity checks, provenance attestation, and a defined update
policy.

## Prompting

The lightest form of adaptation is prompting: crafting the input to the model to
elicit the desired output. A well-designed system prompt can significantly
change how a model behaves — its tone, format, scope, and even its apparent
knowledge — without modifying a single model weight.

Prompting is cheap to iterate, requires no training infrastructure, and produces
changes that are immediately testable. The trade-off is that prompts are
fragile: they are plain text, not code, and subtle wording changes can produce
dramatically different outputs. Prompts are also part of the model's context
window, reducing the space available for user input and retrieved documents.

In a CI/CD context, prompts should be version-controlled, tested against a
representative evaluation set, and promoted through environments like any other
configuration artifact.

## Retrieval-Augmented Generation (RAG)

RAG augments a prompt with dynamically retrieved information: relevant documents
are fetched from a knowledge base and injected into the context alongside the
user's query, grounding the model's response in specific, up-to-date, or
proprietary content.

RAG addresses two specific weaknesses of prompting-only approaches: the model's
training data has a knowledge cutoff and does not include proprietary
information. By retrieving relevant content at inference time, RAG allows the
model to respond based on current or internal knowledge without any change to
model weights.

The operational complexity of RAG lies in the retrieval infrastructure: a vector
store or hybrid search index must be maintained, kept current, and evaluated
separately from the model itself. A RAG pipeline has two quality dimensions —
retrieval quality and generation quality — and failures in either will degrade
the end-to-end result.

## Fine-tuning

Fine-tuning is the process of continuing training on a pre-trained model using a
smaller, task-specific dataset to adapt its behavior to a particular domain or
task. Unlike pre-training, which shapes broad capabilities, fine-tuning shapes
specific behavior — the model's tone, its handling of domain vocabulary, its
output format, or its adherence to particular policies.

Fine-tuning is appropriate when:

- Prompting and RAG have been tried and characterized as insufficient for the
  specific task.
- The task requires consistent output format or style that is difficult to
  enforce through prompting alone.
- Latency or cost constraints require a smaller, specialized model rather than a
  large general-purpose one.
- Data residency or security requirements prevent use of external API providers.

Fine-tuning is a significant operational commitment. It requires curated
training data, a training pipeline, evaluation infrastructure to verify that the
fine-tuned model actually improved on the target task without regressing on
adjacent capabilities, and a model registry to manage the resulting artifacts.
The fine-tuned model is now an internally owned artifact with an ongoing
maintenance burden.

### Full Fine-tuning

Full fine-tuning updates all of the model's parameters during training. It
offers the greatest degree of adaptation but requires substantial GPU memory to
hold the full parameter set during the backward pass, and it risks catastrophic
forgetting: overwriting the model's general capabilities while specializing it
for the target task.

Full fine-tuning is rarely necessary. Parameter-efficient methods achieve
comparable results on most tasks at a fraction of the cost.

### Parameter-Efficient Fine-tuning (PEFT), LoRA, and QLoRA

Parameter-efficient fine-tuning (PEFT) is a family of techniques that adapt a
model by training only a small subset of parameters rather than the full model.
The pre-trained weights are frozen; only the adapter parameters are updated.
This dramatically reduces the compute and memory requirements for fine-tuning
and mitigates catastrophic forgetting, since the original weights are untouched.

Low-Rank Adaptation (LoRA) is the most widely used PEFT method. It inserts
small, trainable low-rank matrices alongside the frozen weight matrices in the
model's attention layers. At inference time, the LoRA weights can be merged back
into the base model weights or applied as a lightweight plugin, allowing
multiple adaptations of the same base model to be maintained and served
efficiently.

QLoRA (Quantized LoRA) extends LoRA by first quantizing the frozen base model
weights to 4-bit precision before adding the LoRA adapters on top. Only the LoRA
adapter weights are trained in full precision; the quantized base weights remain
frozen throughout. This combination reduces the GPU memory required for
fine-tuning by roughly 75% compared to LoRA alone, making it practical to
fine-tune models with tens of billions of parameters on a single consumer-grade
or mid-range GPU. The quality trade-off from 4-bit quantization of the base is
generally small for most downstream tasks, but should be verified through
evaluation on the specific task before committing to a QLoRA-based pipeline.

From a delivery perspective, both LoRA and QLoRA produce adapter artifacts that
are small (often tens to hundreds of megabytes) compared to the full model (tens
of gigabytes). This makes them practical to version, store, and promote
independently of the base model, and to swap at inference time to serve
different use cases from a single loaded base.

## The Decision Framework

The following trade-offs inform which approach is appropriate:

| Approach         | Data Required                     | Training Cost | Maintenance Burden           | Best For                                     |
| ---------------- | --------------------------------- | ------------- | ---------------------------- | -------------------------------------------- |
| Prompting        | None                              | None          | Low (prompt versioning)      | Format, tone, scope control                  |
| RAG              | Documents / knowledge base        | None (model)  | Medium (index maintenance)   | Current or proprietary knowledge             |
| QLoRA            | Hundreds to thousands of examples | Low           | Medium (adapter versioning)  | Resource-constrained fine-tuning, single GPU |
| PEFT / LoRA      | Hundreds to thousands of examples | Low to medium | Medium (adapter versioning)  | Consistent behavior, domain vocabulary       |
| Full fine-tuning | Thousands to millions of examples | High          | High (full model management) | Deep domain adaptation                       |
| Pre-training     | Billions of tokens                | Very high     | Very high                    | Proprietary model from scratch               |

## CI/CD Implications

Each adaptation approach produces a different type of artifact with different
delivery pipeline requirements.

A prompt is a text artifact. It should be stored in version control, deployed as
configuration, and tested against an evaluation set in the same pipeline that
tests code changes.

A RAG knowledge base is a data artifact. It has its own update cadence,
versioning, and quality gates independent of the model. Changes to the retrieval
index should trigger re-evaluation of the end-to-end system.

A LoRA or QLoRA adapter is a model artifact. It is versioned in a model
registry, linked to the base model version it was trained against, and promoted
through evaluation gates before reaching production. QLoRA adapters must also
record the quantization configuration used during training, as this affects
compatibility with the serving infrastructure.

A full fine-tuned model is a complete model artifact with the same storage and
promotion requirements as any other model, plus the added concern of regression
against the base model's general capabilities.

## Related

- [overview.md](overview.md)
- [evaluation.md](evaluation.md)
- [AI Glossary: Fine-tuning](../../reference/ai_glossary.md)
- [AI Glossary: Foundation Model / Base Model](../../reference/ai_glossary.md)
- [AI Glossary: LoRA / PEFT](../../reference/ai_glossary.md)
- [AI Glossary: RAG](../../reference/ai_glossary.md)
- [AI Glossary: Model Registry](../../reference/ai_glossary.md)

## Links

- [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685)
- [QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314)
- [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)
- [The Power of Scale for Parameter-Efficient Prompt Tuning](https://arxiv.org/abs/2104.08691)
