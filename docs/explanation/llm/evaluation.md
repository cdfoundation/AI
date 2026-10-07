# LLM Evaluation

## Why LLM Evaluation Is Fundamentally Different

Traditional software testing rests on a simple principle: given a known input,
assert a known output. A function either returns the expected value or it does
not. This determinism makes testing tractable — a suite of unit tests produces a
binary pass or fail result that is reliable, reproducible, and automatable.

LLM evaluation cannot rely on this principle. The same input produces different
outputs across runs. The definition of a "correct" response to most natural
language tasks is inherently subjective or context-dependent. A response can be
factually accurate but poorly formatted, well-formatted but incomplete, complete
but subtly wrong in ways that only a domain expert would notice. The space of
possible failures is not enumerable in advance.

This does not mean LLM evaluation is impossible — it means it requires different
tools and a different mindset. The goal is not to prove the model is correct; it
is to characterize its behavior across a representative sample of inputs and
establish whether that behavior meets a defined quality bar.

## Types of Evaluation

### Benchmark Evaluation

Benchmarks are standardized evaluation sets with defined metrics, widely used to
compare models before selection or deployment. Examples include MMLU (reasoning
across domains), HumanEval (code generation), and TruthfulQA (factual accuracy).

Benchmarks are useful for initial model selection and for tracking broad
capability changes across versions. They are not sufficient for production
evaluation because they measure general capability, not task-specific fitness. A
model that scores highly on MMLU may still perform poorly on the specific domain
or format a production application requires.

The benchmark overfitting problem is well documented: models that are explicitly
trained or prompted to perform well on a specific benchmark often degrade on
related tasks that the benchmark was intended to represent. High benchmark
scores should be treated as necessary but not sufficient evidence of production
readiness.

### Human Evaluation

Direct human judgment over model outputs is the most reliable evaluation method
for tasks where quality is subjective — creative writing, nuanced summarization,
sensitive content handling. Humans can identify problems that automated metrics
miss: factual errors that sound plausible, responses that technically answer the
question while avoiding its intent, or subtle tone mismatches.

Human evaluation is expensive, slow, and difficult to scale. It is most valuable
for establishing ground truth for a new task (building the evaluation set
described below), for auditing automated evaluation results periodically, and
for edge cases that automated metrics consistently fail to catch.

### Automated Metrics

For tasks with more structured outputs, automated metrics provide scalable,
reproducible quality signals.

- **Exact match and ROUGE/BLEU**: Appropriate for tasks where the correct output
  is well-defined — classification, extraction, structured data generation.
  Inappropriate for open-ended generation where multiple valid outputs exist.
- **Code execution**: For code generation tasks, running the generated code
  against a test suite is the most reliable signal. A generated function either
  passes the tests or it does not.
- **Factual grounding checks**: For RAG systems, measuring whether the model's
  response is supported by the retrieved documents provides a measurable proxy
  for hallucination risk.

### LLM-as-Judge

Using a second LLM to evaluate the output of the primary model is a scalable
approach for tasks where human evaluation is the gold standard but cannot be
done at volume. The evaluator model is prompted with a rubric — criteria such as
accuracy, completeness, tone, and format — and asked to score or rank responses.

LLM-as-judge can produce useful signal but has known failure modes. Evaluator
models are biased toward outputs that resemble their own generation style,
prefer longer responses, and can be influenced by the order in which options are
presented. Results from LLM-as-judge should be calibrated against human
judgments on a representative sample before being used as a primary quality
gate.

## The Evaluation Dataset

The foundation of reproducible LLM evaluation is a curated evaluation dataset: a
collection of inputs (and, where possible, expected outputs or quality criteria)
that is held out from training and used consistently to measure model behavior
over time.

A good evaluation dataset has the following properties:

- **Representative**: It covers the distribution of real production inputs,
  including edge cases and challenging examples, not just easy or typical ones.
- **Stable**: It does not change between evaluation runs. Stability is what
  makes results comparable across model versions.
- **Labeled**: For tasks where expected outputs can be defined, ground truth
  labels allow automated scoring. For tasks where they cannot, the dataset
  should include the rubrics that human or LLM evaluators will apply.
- **Growing**: As new failure modes are discovered in production, corresponding
  examples should be added to the evaluation set so that regressions on those
  failures will be caught in future evaluations.

The evaluation dataset is a long-term project artifact that accumulates value
over time. It should be version-controlled, treated with the same care as
training data, and maintained independently of any specific model version.

## Evaluation in a CI/CD Pipeline

In a traditional software pipeline, the test suite runs on every commit and
produces a pass/fail signal that gates promotion. The LLM equivalent is an
evaluation gate: a step in the model promotion pipeline that runs the candidate
model against the evaluation dataset and blocks promotion if quality thresholds
are not met.

A well-designed evaluation gate has the following properties:

- **Defined thresholds**: The metrics and pass/fail thresholds are specified
  before evaluation runs, not adjusted after seeing results.
- **Automated execution**: The evaluation runs without human intervention and
  produces a reproducible result given the same model and dataset.
- **Regression detection**: Results are compared against the current production
  model, not just against an absolute threshold. A model that performs worse
  than production on any significant metric should not be promoted, even if it
  exceeds the absolute threshold.
- **Scope-appropriate metrics**: The metrics measured correspond to what the
  model is actually used for in production, not just what is convenient to
  measure.

The evaluation gate is not a guarantee of production quality. It is a defense
against regressions and obvious failures. Production monitoring is a separate,
ongoing concern that catches issues the evaluation gate does not.

## What "Done" Looks Like for a Model

A model is ready for production promotion when:

1. It meets all defined quality thresholds on the held-out evaluation dataset.
2. It does not regress on any significant metric compared to the current
   production model.
3. It has passed inference performance testing: latency SLOs are met at expected
   concurrency under realistic load.
4. If quantized, the quantized artifact has been separately evaluated.
5. Provenance is documented: the training run, dataset version, and
   hyperparameters are recorded and the artifact is signed.
6. The relevant stakeholders have approved promotion, particularly for
   high-stakes or regulated applications.

This definition of "done" is necessarily task-specific and
organization-specific. What matters is that it is written down, agreed upon
before evaluation starts, and applied consistently across every model promotion
decision.

## Evaluation Debt

Evaluation debt is the accumulation of known limitations in a model or
evaluation setup that have been accepted or deferred rather than addressed. Like
technical debt in software, evaluation debt compounds over time. A model
promoted with known gaps in its evaluation coverage will eventually produce
failures in production that the pipeline was not equipped to detect.

Common sources of evaluation debt:

- Evaluation datasets that do not cover recent changes in production input
  distribution.
- Quality thresholds set low to allow a launch to proceed rather than based on
  actual production requirements.
- Metrics that are easy to compute rather than metrics that reflect what users
  care about.
- LLM-as-judge setups that have not been calibrated against human judgments.

Addressing evaluation debt requires investment in evaluation infrastructure that
is often less visible and less celebrated than model capability improvements. It
is nonetheless essential for sustainable model delivery.

## Related

- [overview.md](overview.md)
- [fine_tuning.md](fine_tuning.md)
- [inference.md](inference.md)
- [AI Glossary: Evaluation Gate](../../reference/ai_glossary.md)
- [AI Glossary: Golden Data Set](../../reference/ai_glossary.md)
- [AI Glossary: LLM as Judge](../../reference/ai_glossary.md)
- [AI Glossary: Evals](../../reference/ai_glossary.md)
- [AI Glossary: Model Card](../../reference/ai_glossary.md)

## Links

- [Holistic Evaluation of Language Models (HELM)](https://crfm.stanford.edu/helm/)
- [BIG-bench: Beyond the Imitation Game](https://arxiv.org/abs/2206.04615)
- [Chatbot Arena: An Open Platform for Evaluating LLMs by Human Preference](https://arxiv.org/abs/2403.04132)
- [G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment](https://arxiv.org/abs/2303.16634)
