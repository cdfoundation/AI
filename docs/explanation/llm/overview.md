# LLM Overview

## What an LLM Is

A Large Language Model is a neural network trained to predict the next token in
a sequence of text. That deceptively simple objective, applied at massive scale
across hundreds of billions of tokens of text, produces a model that can
summarize documents, write code, answer questions, reason through problems, and
follow complex instructions.

The key word is "predict." An LLM does not retrieve facts from a database or
execute a search. It generates output by repeatedly sampling the most probable
next token given everything that came before it. This probabilistic, generative
nature is the source of both the capability and the fundamental operational
challenges LLMs introduce for CI/CD teams.

## How LLMs Are Trained

LLM training happens in two major phases.

### Pre-training

The model is trained on a large corpus of text — web pages, books, code,
scientific papers — with a single objective: predict the next token. No labels,
no human feedback, no task specification. The model learns grammar, facts,
reasoning patterns, and coding conventions as an emergent side effect of being
forced to predict what comes next across trillions of examples.

Pre-training is extraordinarily expensive. Training a frontier model costs tens
to hundreds of millions of dollars in compute. It requires thousands of GPUs
running for weeks or months, careful data curation pipelines, and significant
engineering investment in distributed training infrastructure. Pre-training is
not something most organizations do; they consume pre-trained base models as a
starting point.

### Alignment and Post-training

A raw pre-trained model produces fluent text but is not reliably useful or safe.
Post-training shapes the model's behavior through techniques such as:

- **Instruction fine-tuning**: Training on examples of instruction-response
  pairs so the model learns to follow directions rather than just complete text.
- **Reinforcement Learning from Human Feedback (RLHF)**: Using human preference
  judgments to train a reward model, then optimizing the LLM against that reward
  model to produce outputs humans prefer.
- **Constitutional AI / RLAIF**: Variants that use AI-generated feedback rather
  than or in addition to human feedback, reducing the annotation bottleneck.

The result is a model that is both capable and aligned to behave helpfully and
avoid producing harmful outputs. This post-training is what distinguishes a
deployable chat or coding model from a raw pre-trained base.

## What Makes LLMs Different from Traditional Software

LLMs are software artifacts, but they behave differently from any software
artifact CI/CD pipelines were designed to handle. Understanding these
differences is prerequisite to designing pipelines that manage them well.

### Non-determinism

Given the same input, an LLM will not always produce the same output. The
sampling process is stochastic by design. A traditional unit test that asserts
`function(input) == expected_output` does not apply directly. Evaluation
requires statistical approaches: running the model across a representative
dataset and measuring aggregate quality metrics rather than asserting exact
outputs.

### Emergent and Opaque Behavior

LLM capabilities are not explicitly programmed. They emerge from training and
cannot be fully enumerated. A model that can write Python may also, without
explicit design, be able to write Rust, reason about legal documents, or explain
a physics concept. This makes it impossible to produce a complete specification
of what a model can and cannot do. Testing must be empirical and ongoing, not
exhaustive and final.

### Scale of Artifacts

A typical software build artifact might be tens or hundreds of megabytes. A
medium-sized LLM is tens of gigabytes. A large frontier model can exceed a
terabyte in full precision. This changes the economics and mechanics of artifact
storage, transfer, caching, and promotion. Model registries, chunked transfers,
and content-addressed storage are necessary infrastructure, not optional
optimizations.

### Reproducibility Is Not Free

Compiling the same source code twice produces the same binary. Training the same
model twice does not produce the same weights. Floating-point non-determinism in
GPU operations, data ordering, and subtle differences in environment mean that
rerunning a training job produces a functionally similar but not bit-identical
model. Reproducibility in the LLM context means preserving the exact artifact
rather than rebuilding it, and tracking the full provenance of that artifact:
training data, code, hyperparameters, and infrastructure.

### Behavior Degrades Silently Over Time

Traditional software does not spontaneously start returning wrong answers
because the world changed. LLM behavior degrades when the distribution of
production inputs shifts away from the distribution the model was trained on
(data drift), or when the correct answer to a question changes over time while
the model's weights do not (concept drift). A model that was accurate at
deployment can become inaccurate months later without a single line of code
changing.

## Why This Matters for CI/CD

Each of these properties translates directly into CI/CD pipeline requirements.

| LLM Property                        | CI/CD Implication                                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------------------------- |
| Non-determinism                     | Evaluation gates must use statistical metrics over datasets, not exact-match assertions         |
| Opaque behavior                     | Comprehensive evaluation suites are required before every promotion decision                    |
| Large artifact size                 | Model registries, content-addressed storage, and efficient transfer are required infrastructure |
| Reproducibility requires provenance | Training runs must log all inputs; artifacts must be signed and attested                        |
| Silent degradation over time        | Continuous monitoring and triggered retraining pipelines are necessary in production            |
| Pre-training cost                   | Organizations consume base models; CI/CD pipelines manage fine-tuning, evaluation, and serving  |

The core challenge is that the software delivery practices developed over
decades for deterministic, human-authored artifacts must be extended — not
replaced — to handle stochastic, learned artifacts. The pipelines, gates,
registries, and security controls all have LLM equivalents, but they require
adaptation.

## The LLM Delivery Lifecycle

For most organizations, the LLM delivery lifecycle looks like this:

1. Select a base model from a provider or open-source registry.
2. Adapt the model to the task through prompting, RAG, or fine-tuning.
3. Evaluate the adapted model against a defined quality bar.
4. Deploy the model to a serving infrastructure behind an inference API.
5. Monitor production behavior and collect feedback.
6. Trigger retraining or adaptation when drift or regression is detected.
7. Promote the updated model through the same evaluation and deployment
   pipeline.

Each of these steps is a CI/CD concern. Steps 1 and 2 are covered in
[fine_tuning.md](fine_tuning.md). Step 3 is covered in
[evaluation.md](evaluation.md). Steps 4 and 5 are covered in
[inference.md](inference.md).

## Related

- [fine_tuning.md](fine_tuning.md)
- [inference.md](inference.md)
- [evaluation.md](evaluation.md)
- [AI Glossary: LLM](../../reference/ai_glossary.md)
- [AI Glossary: Foundation Model](../../reference/ai_glossary.md)
- [AI Glossary: Hallucination](../../reference/ai_glossary.md)
- [AI Glossary: Model Drift](../../reference/ai_glossary.md)

## Links

- [Attention Is All You Need (Transformer architecture)](https://arxiv.org/abs/1706.03762)
- [Training language models to follow instructions with human feedback (InstructGPT / RLHF)](https://arxiv.org/abs/2203.02155)
- [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073)
