# Large Language Models

Explanation documents covering Large Language Models (LLMs) in the context of
the CDFoundation CI/CD AI SIG. These documents are understanding-oriented: they
discuss background, architecture, trade-offs, and rationale. For step-by-step
guidance see the [how-to guides](../../how-to/). For concise definitions see the
[AI Glossary](../../reference/ai_glossary.md).

## Contents

| Document                                     | Description                                                                                       |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [overview.md](overview.md)                   | What LLMs are, how they are trained, and why they present distinct challenges for CI/CD pipelines |
| [fine_tuning.md](fine_tuning.md)             | Base models, fine-tuning, PEFT/LoRA, and when each approach fits a delivery context               |
| [inference.md](inference.md)                 | Inference concepts: latency, throughput, batching, quantization, and serving trade-offs           |
| [evaluation.md](evaluation.md)               | How LLM output quality is measured and what an evaluation gate looks like in a delivery pipeline  |
| [context_window.md](context_window.md)       | The context window: mechanics, components, size categories, and the challenges of long contexts   |
| [tokens.md](tokens.md)                       | Tokenization, Byte Pair Encoding, and OpenAI encoding schemes                                     |
| [context_pollution.md](context_pollution.md) | How noisy or conflicting context degrades model reasoning and what causes it                      |

## Scope

These documents cover LLM concepts as they relate to CI/CD delivery pipelines
and platform engineering. They are not a general introduction to machine
learning or deep learning. The focus is on what makes LLMs difficult to manage
as software artifacts, and how standard CI/CD concepts such as evaluation gates,
artifact promotion, supply chain integrity, and reproducibility apply to models.
