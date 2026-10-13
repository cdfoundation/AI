# AI Glossary

A comprehensive reference for terms used across the CDFoundation AI SIG scope:
AI-assisted development, context engineering, MLOps, inference serving, CI/CD
integration for AI workloads, and platform engineering.

## Core Concepts

### Agent

An AI system that can dynamically choose and use tools to accomplish tasks with
varying degrees of autonomy. The definition is contested—some define agents as
systems with autonomous tool choice, while others use it more broadly. Anthropic
distinguishes between "workflows" (structured, predictable sequences) and
"agents" (systems that make autonomous decisions about tool usage).

### Agent Harness

The collection of tools, capabilities, and environment that an agent can access.
This includes whether the agent can spawn sub-agents, write/execute code, access
file systems, use search tools, call APIs, etc. The harness significantly
affects what an agent can accomplish.

### Agent Skills / Skills

Structured files (typically Markdown) that provide agents with specific
capabilities or knowledge. Skills use progressive disclosure—the agent sees
descriptions of all available skills, then loads full details only when needed.
Skills can include instructions, example code, scripts, and references to other
tools. The format and portability of skill files varies by platform and is not
standardized across AI coding tools.

### Agentic Search

Search performed by AI agents rather than humans. Unlike traditional search (one
query, ten blue links), agentic search involves multiple iterative queries,
multi-hop reasoning, and canvassing hundreds or thousands of pages to assemble
information. The agent authors queries, executes searches, and digests results
autonomously.

### AGENTS.md

A markdown file in your project repository that serves as persistent context and
documentation for AI agents working on that project. Contains project goals,
rules, conventions, library choices, file locations, and other essential
context. Updated continuously as the project evolves—a form of "living
documentation."

### Async Agents / Background Agents

AI agents that run without real-time human supervision, typically on schedules
(cron jobs) or triggered by events (GitHub Actions, CI/CD pipelines). Examples
include agents that work through GitHub issues, run code quality checks, or
handle routine maintenance tasks while you focus on other work.

### Autonomous AI Coding

The progression beyond collaborative coding where AI agents work independently
on tasks with human review rather than real-time collaboration. Involves async
agents, automated workflows, and pull request-based collaboration patterns.

### Foundation Model / Base Model

A large model trained on broad, general-purpose data that serves as the starting
point for downstream use. Foundation models can be used directly via prompting
or adapted through fine-tuning for specific tasks or domains. Examples include
general-purpose LLMs (Claude, GPT, Gemini) and multimodal models.

### Guardrails

Input and output validation mechanisms that enforce policies on what an AI
system can receive and produce. Guardrails can be implemented at the prompt
level, as separate classifier models, or as post-processing layers. They are
used to prevent harmful outputs, enforce business rules, and maintain safety
constraints in production systems.

### Hallucination

When a language model generates output that is confident but factually
incorrect, inconsistent with the provided context, or entirely fabricated.
Hallucinations are a fundamental failure mode of LLMs, not bugs that can be
fully eliminated. Mitigations include grounding, RAG, tool use with verified
sources, and output verification steps.

### Human-in-the-Loop (HITL)

A design pattern requiring explicit human review or approval at defined points
in an automated or agentic workflow. HITL gates are used where the consequences
of an incorrect automated decision are significant—such as model promotion,
deployment to production, or irreversible pipeline actions.

### LLM (Large Language Model)

A neural network model trained on large volumes of text to predict and generate
language. LLMs underpin most modern AI coding assistants, agents, and natural
language interfaces. They are characterized by having billions of parameters and
the ability to follow instructions, answer questions, generate code, and reason
across a wide range of tasks.

### System Prompt

Instructions provided to a model at the start of a session, before any user
input, that define its persona, constraints, capabilities, and behavior. System
prompts are a primary mechanism for customizing model behavior in applications
and agents. They form a trust boundary and should be treated as a security
surface—prompt injection attacks target system prompt override.

## Context & Engineering

### Context Engineering

The practice of carefully curating what information goes into an AI agent's
context window to maximize accuracy, minimize distraction, and manage costs. The
modern evolution of prompt engineering, focused on the entire information
environment rather than just the prompt. Considered "the job" for building
production AI systems.

### Context Rot / Context Window Degradation

The phenomenon where AI model performance decreases as the number of tokens in
the context window increases. Even state-of-the-art models show measurable
accuracy drops on tasks requiring precise attention as context length scales
into the tens or hundreds of thousands of tokens. This makes simply "stuffing"
large context windows impractical for production systems.

### Context Window

The amount of text (measured in tokens) that an AI model can "see" and process
at once. Includes the system prompt, conversation history, retrieved documents,
tool outputs, and any other information provided to the model.

### Golden Data Set / Eval Set

A curated collection of inputs (and ideally expected outputs) used to
consistently evaluate AI system performance over time. Essential for measuring
the impact of changes to prompts, context, models, or tools. Can be expanded as
new edge cases are discovered.

### Harness Engineering

The practice of designing and optimizing the tools and environment (harness) in
which an agent operates. Related to but distinct from context
engineering—focuses on what capabilities the agent has access to rather than
what information is in context.

### Progressive Disclosure

A technique where an agent initially sees only high-level descriptions of
available resources (like skills or documents), then loads full details only
when relevant. Helps manage context window size while maintaining access to
broad capabilities.

## Evaluation & Quality

### Evals / Evaluations

Systematic testing of AI system outputs against expected results. Can range from
informal "vibe checks" (human reviewing outputs) to formal automated scoring.
The rigor should match the consequences of failure—low stakes warrant simple
evals, high stakes demand comprehensive evaluation.

### LLM as Judge

Using a language model to evaluate the outputs of another language model. Useful
but not a complete solution—provides scalable evaluation but has its own biases
and limitations.

### Minimum Viable Evals (MVE)

The simplest evaluation approach that provides useful signal, often starting
with vibe-based human review of consistent inputs before building more
sophisticated evaluation harnesses.

### Traces / LLM Traces

Detailed logs of what an AI agent did during execution, including prompts sent,
responses received, tools called, and decisions made. Essential for debugging,
evaluation, and continuous improvement. Should be logged and analyzed
systematically.

## Search & Retrieval

### BM25

A lexical (keyword-based) search algorithm that ranks documents based on term
frequency and document length. Often used alongside vector search in hybrid
search approaches. Good at exact matches and specific terminology.

### Embedding

A numerical vector representation of text (or other data) that captures semantic
meaning. Texts with similar meanings have similar embeddings. Used for semantic
search and finding conceptually related information.

### Hybrid Search

Combining lexical search (like BM25) with semantic search (vector embeddings).
Leverages the complementary strengths of both approaches—lexical for precise
terminology, semantic for conceptual similarity. Considered a best practice for
most use cases.

### RAG (Retrieval Augmented Generation)

A pattern where relevant information is retrieved from a knowledge base and
included in the prompt to ground the AI's response in specific facts or
documents. Contrasts with relying solely on the model's training data.
Retrieval-Augmented Generation reduces hallucinations and allows models to
access up-to-date or proprietary information without retraining.

### Vector Database / Vector Store

A database optimized for storing embeddings and performing similarity search.
Examples include Chroma, Pinecone, Weaviate, and Qdrant. Essential
infrastructure for semantic search and RAG systems.

## Development Practices

### Compaction

The automatic process some AI coding tools use to compress or summarize
conversation history when it gets too long, freeing up context window space.
Often opaque and can lose important information—one reason explicit living
documentation is valuable.

### Fine-tuning

Training a pre-trained language model further on specific data to adapt it to
particular tasks or domains. Generally attempted after simpler techniques (RAG,
prompt engineering, few-shot examples) have been exhausted, because it is more
expensive and complex to maintain. Common use cases include cost and latency
reduction at scale, strict data residency or regulatory requirements that
prevent use of third-party API providers, and performance on highly specialized
domains not well represented in the base model's training data.

### Inner Loop vs Outer Loop

**Inner loop**: Figuring out what information to put into context right now for
the current task. **Outer loop**: Building systems that improve at context
selection over time through feedback and learning.

### Living Documentation

Documentation that evolves continuously with a project rather than becoming
stale. In AI-assisted coding, this often means having agents automatically
update documentation files like AGENTS.md as they learn new patterns or
preferences.

### Non-LLM Filter

The practice of asking "do I actually need an LLM for this?" before reaching for
AI solutions. Hierarchy: 1) Simple Python/traditional code, 2) Simple ML
model, 3) LLM workflow, 4) LLM agent. Simpler solutions are more reliable and
easier to debug.

### Spec-First Planning / Specification-First

Starting development with clear specifications that define what you're building,
constraints, success criteria, and architecture before writing code. Prevents
"vibing" with an agent and losing control of the direction.

### Sub-agents

Specialized agents spawned by a primary agent to handle specific subtasks. Helps
manage context rot by giving each sub-agent focused context for its particular
job rather than stuffing everything into one massive context window.

### Tool Use / Tool Calling

The ability of an AI agent to call external functions or APIs to accomplish
tasks beyond text generation. Tools might include search, code execution, file
operations, API calls, database queries, etc.

## Workflow Patterns

### Collaborative Coding

Real-time interaction with AI coding assistants, where human and AI work
together iteratively on problems. The middle ground between basic code
completion and fully autonomous agents.

### Multi-hop Reasoning / Search

The ability to connect information across multiple steps or queries. Instead of
answering from a single source, the agent searches iteratively, using
information from one query to inform the next, building up a comprehensive
answer.

### Workflow (vs Agent)

A structured, predictable sequence of steps to accomplish a task. More
deterministic than agents. Anthropic's distinction: workflows have predefined
paths, agents make autonomous decisions about how to proceed.

## MLOps and Continuous Training

### Continuous Training (CT)

The practice of automatically retraining a model in response to triggers such as
data drift detection, scheduled intervals, or accumulated new labeled data. CT
pipelines treat model training as a recurring, automated stage in the delivery
pipeline rather than a one-time event. Essential for maintaining model accuracy
as production data evolves over time.

### Data Drift / Model Drift

Data drift occurs when the statistical distribution of production input data
diverges from the distribution the model was trained on. Model drift (or concept
drift) occurs when the relationship between inputs and correct outputs changes
over time, degrading accuracy. Both are monitored as signals to trigger
retraining or model review in a CT pipeline.

### Evaluation Gate

An automated quality checkpoint that a trained model must pass before being
promoted to the next environment. Evaluation gates run a defined set of metrics
against a held-out evaluation dataset and block promotion if thresholds are not
met. They are the ML equivalent of CI test gates in a software delivery
pipeline.

### Experiment Tracking

The systematic logging of ML training runs, including hyperparameters, datasets,
metrics, and output artifacts, to enable comparison and reproducibility. Tools
such as MLflow, Weights and Biases, and DVC provide experiment tracking
infrastructure. Tracked experiments form the basis for model selection decisions
and provide an audit trail for governance.

### Feature Store

A centralized repository for computed ML features—numerical representations
derived from raw data—that are shared across model training and inference. A
feature store ensures consistency between the features used during training and
those served at inference time, preventing training-serving skew, and enables
feature reuse across teams and models.

### Model Artifact

The set of files produced by a training run that fully define a trained model:
weights, configuration, tokenizer files, preprocessing logic, and metadata.
Model artifacts are versioned, stored in a model registry, and promoted through
environments like any other software artifact in a delivery pipeline.

### Model Card

A short standardized document that accompanies a trained model, describing its
intended use cases, training data, evaluation results, known limitations, and
ethical considerations. Model cards are the primary documentation standard for
published models and are required by many model registries and AI governance
frameworks.

### Model Promotion

The process of moving a trained and validated model artifact from one
environment to the next (e.g., development to staging to production). Model
promotion is gated by evaluation gates and approval workflows, directly
analogous to release promotion in a software CI/CD pipeline.

### Model Registry

A versioned artifact store specifically designed for trained models. A model
registry tracks model versions, associated metadata (training run, dataset
version, evaluation results), lifecycle stage (staging, production, archived),
and lineage. Examples include MLflow Model Registry, Hugging Face Hub, and
cloud-provider-native registries.

## Inference Serving

### Canary Deployment

A deployment strategy that routes a small, controlled percentage of live traffic
to a new model version while the majority continues to use the current version.
Canary deployments allow validation of a new model's behavior under real
production conditions before full rollout, limiting the blast radius of any
regression.

### Inference Server

Software that loads a trained model and serves it as an API, handling request
batching, hardware acceleration, and concurrent requests. Common LLM inference
servers include vLLM, NVIDIA Triton Inference Server, ONNX Runtime Server, and
llama.cpp. The inference server is the primary deployment unit for models in
production and the main integration point with CD pipelines.

### Latency SLO (Service Level Objective)

A target threshold for inference response time that the serving infrastructure
is expected to meet under defined load conditions. Latency SLOs are used to
define promotion criteria for model deployments, trigger autoscaling, and
evaluate infrastructure changes. Commonly expressed as a percentile target
(e.g., p99 latency under 500ms).

### ONNX (Open Neural Network Exchange)

An open format for representing machine learning models, enabling models trained
in one framework (e.g., PyTorch, TensorFlow) to be deployed in another runtime
(e.g., ONNX Runtime, TensorRT). ONNX is a key interoperability standard in ML
deployment pipelines, allowing training and inference infrastructure to be
decoupled.

### Quantization

The process of reducing the numerical precision of a model's weights (e.g., from
32-bit float to 8-bit or 4-bit integer) to reduce memory footprint and increase
inference throughput, at the cost of some accuracy. Quantization is one of the
most common techniques for deploying large models on constrained hardware or
reducing inference serving costs at scale.

### Shadow Deployment

A deployment strategy where a new model version receives copies of live
production traffic and generates responses, but those responses are not served
to users. Shadow deployment allows side-by-side comparison of the new model's
outputs against the current model without any user-facing risk.

## Models & Infrastructure

### Chain of Thought (CoT)

A prompting technique that instructs a model to reason through a problem
step-by-step before producing a final answer. CoT significantly improves
accuracy on complex reasoning, math, and multi-step tasks. It can be elicited
with simple instructions such as "think step by step" or structured into
explicit reasoning traces in agentic workflows.

### BPE (Byte Pair Encoding)

A subword tokenization algorithm used by modern AI language models. Originally
derived from data compression, BPE iteratively merges the most frequently
occurring pairs of characters or subword units into single tokens based on
training corpus statistics. This allows the model to handle any text string
(including typos, code, and rare words) while keeping its vocabulary size
manageable.

### Frontier Models

The most capable, cutting-edge AI models available at any given time. The
specific models that qualify as frontier change rapidly; current examples
include the latest releases from Anthropic (Claude), OpenAI (GPT and o-series),
and Google (Gemini). Frontier models typically lead on benchmark performance but
also carry the highest inference costs.

### Grounding

Connecting model outputs to verified, factual, or authoritative information
sources to reduce hallucination and increase trustworthiness. Grounding
techniques include RAG (providing retrieved documents as context), tool use
(querying live or authoritative data sources), and citation requirements (asking
the model to reference sources in its response).

### Model Routing

Automatically selecting which model to use for different tasks based on
requirements. Some tools route complex reasoning to powerful models and simpler
tasks to faster/cheaper models.

### LoRA / PEFT (Low-Rank Adaptation / Parameter-Efficient Fine-Tuning)

A family of techniques for fine-tuning large models by training only a small
subset of parameters rather than updating all model weights. LoRA inserts small
trainable matrices into the model's attention layers, reducing GPU memory and
compute requirements by orders of magnitude compared to full fine-tuning. PEFT
is the umbrella term; LoRA is the most widely adopted method.

### Non-English Token Tax

The phenomenon where non-English languages (and heavily formatted code) require
significantly more tokens to represent the same amount of meaning compared to
English. This occurs because tokenizers are typically trained on English-heavy
datasets, meaning non-English text doesn't compress as efficiently during Byte
Pair Encoding. This leads to higher API costs and reduced effective context
windows.

### Serverless Infrastructure

Cloud computing platforms that handle infrastructure management automatically,
charging only for actual usage. Examples include Modal, AWS Lambda, Google Cloud
Functions. Enables rapid deployment without traditional DevOps overhead.

### Temperature

A parameter controlling randomness in AI model outputs. Lower temperatures
(e.g., 0.2) produce more consistent, deterministic outputs. Higher temperatures
(e.g., 0.8) produce more creative, varied outputs.

### Tokens

The fundamental unit of data processed by Large Language Models. LLMs do not
read text or characters; they process sequences of integer IDs representing
tokens. A token can be a whole word, a subword, or a single character. As a
general rule for English, one token is roughly four characters or three-quarters
of a word.

## Security & Compliance

### Data Security / Permission Mapping

In regulated environments, ensuring that AI systems respect the same access
controls as source data. Major challenge: if an LLM is trained on documents with
different permission levels, who should have access to that LLM? Similar
challenges for vector stores, training logs, and traces.

### GxP Validation

An umbrella term for "Good Practice" quality regulations in regulated industries
(GMP - Good Manufacturing Practice, GLP - Good Laboratory Practice, GCP - Good
Clinical Practice, etc.). GxP-validated systems have strict requirements for
documentation, testing, auditability, and change control that directly affect
how AI systems can be developed, validated, and deployed in pharmaceutical,
medical device, and life sciences contexts.

### Jailbreaking

Techniques to make AI systems bypass their safety guidelines or intended
constraints. A security concern when using system prompts to enforce data access
controls.

### Model Supply Chain

The full set of upstream dependencies that contribute to a trained model
artifact: base model weights, training datasets, data processing pipelines,
training code, and infrastructure. Securing the model supply chain—analogous to
software supply chain security—involves provenance tracking, artifact signing,
and integrity verification at each stage of the pipeline.

### Prompt Injection

An attack where malicious content in a model's input overrides or hijacks the
system prompt or agent instructions, causing the model to take unintended
actions. Prompt injection is a primary security concern for agentic systems that
process untrusted external content such as web pages, user-uploaded documents,
or tool outputs. Defenses include input sanitization, privilege separation, and
output validation.

### Provenance Attestation

A cryptographic record that establishes the origin of a model artifact: who
produced it, from what inputs, and through which pipeline. Provenance
attestations (using standards such as SLSA or in-toto) allow consumers to verify
that a model has not been tampered with and was produced by a trusted, auditable
process.

### SBOM (Software Bill of Materials) for AI

An inventory of the components, dependencies, and data sources that make up an
AI system, extending the software SBOM concept to include training datasets,
base model weights, and fine-tuning data. AI SBOMs support supply chain
transparency, license compliance, and vulnerability tracking for model artifacts
in a delivery pipeline.

## Productivity Concepts

### Intellectual Drudgery

Tasks that don't require deep thinking but consume significant time and human
attention. Prime candidates for AI automation. Examples: reformatting documents,
filling forms, writing routine code, generating commit messages.

### Quantify the Unquantified

Using AI to extract structured data or insights from unstructured sources like
PDFs, lab reports, or meeting transcripts. Common in scientific and enterprise
settings.

### Speed of Thought

Building tools and workflows that keep pace with human thinking rather than
introducing friction. Enabled by modern serverless infrastructure, reactive
interfaces, and AI assistance.

## Common Pitfalls & Anti-patterns

### Babysitting

Sitting and watching an agent work in real-time rather than using async
workflows. Wastes human time and attention—better to start agents on tasks and
review results later.

### Context Stuffing

Trying to solve problems by putting as much information as possible into the
context window. Leads to context rot, higher costs, slower responses, and often
worse results than carefully curated context.

### Vibing

Working with an agent without clear specifications or plans, just having
open-ended conversations and seeing where it goes. Often loses direction after a
few turns and produces poor results.

## Tools & Platforms

### Claude Code

Anthropic's AI coding tool, available as a CLI and integrated into editors and
desktop interfaces. Supports agentic workflows, tool use, and computer use.
Anthropic also provides claude.ai (web) and Claude Desktop for broader access to
the Claude model family.

### Cursor

AI-powered code editor built on VS Code with deep AI integration for code
completion, chat, and agent-like capabilities.

### MCP (Model Context Protocol)

A protocol for connecting AI models to various data sources and tools through
standardized servers. Enables agents to access external systems in a structured
way.

### Marimo

A reactive Python notebook that prevents variable redeclaration and can run
standalone with inline dependencies. Supports deployment as web apps and
deployment on serverless platforms.

### Modal

Serverless Python platform for deploying code, scheduled jobs, and web services
without managing infrastructure. Popular for AI/ML workloads.

---

## Quick Reference: Key Principles

1. **Context is king**: Engineering what goes into the context window is the
   primary lever for agent performance
2. **Start simple**: Apply the non-LLM filter—use traditional code before ML,
   simple ML before LLMs
3. **Match eval rigor to consequences**: Vibe checks for low stakes,
   comprehensive evals for high stakes
4. **Living documentation > static docs**: Keep AGENTS.md and related files
   updated as projects evolve
5. **Specification before implementation**: Clear specs prevent losing control
   to agent "vibing"
6. **Build the outer loop**: Focus on systems that improve over time, not just
   solving immediate problems
7. **Progressive disclosure**: Show agents what's available, load details only
   when needed
8. **Hybrid search as default**: Combine lexical and semantic search for best
   results
9. **Log everything**: Traces are essential for debugging, evaluation, and
   improvement
10. **Skepticism toward hype**: Follow academics over hypebeasts; focus on real
    capabilities not demos
