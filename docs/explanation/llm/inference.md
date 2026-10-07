# LLM Inference

## What Inference Means for LLMs

In machine learning, inference is the process of running a trained model on new
input to produce output. For LLMs, inference is the act of generating tokens:
the model processes an input sequence and repeatedly samples the next token
until it produces a stop condition or reaches the output length limit.

Inference is fundamentally different from training. Training updates the model's
weights and runs over the entire dataset many times; it is a batch, offline
process. Inference reads the weights (which do not change) and responds to
individual requests in real time. The two phases have completely different
resource profiles, latency requirements, and operational concerns, and they are
almost always handled by separate infrastructure.

Understanding inference is a CI/CD concern because the serving infrastructure is
the deployment target for every model promotion. The performance characteristics
of inference — latency, throughput, memory footprint — determine whether a model
is fit for production and define the acceptance criteria that evaluation gates
must verify.

## Key Metrics

### Time to First Token (TTFT)

The time from when a request is received to when the first output token is
returned. TTFT matters for interactive applications where users perceive any
delay before the response begins. It is primarily affected by the input sequence
length (longer inputs take more time to process before generation begins) and
the degree of parallelism the serving infrastructure provides.

### Time Between Tokens (TBT) / Generation Speed

Once generation begins, TBT measures how quickly subsequent tokens are produced.
For a user reading a streamed response, TBT determines whether the output feels
smooth or choppy. Generation speed is primarily a function of model size, batch
size, and hardware.

### Throughput

The number of tokens the serving infrastructure can generate per second across
all concurrent requests. Throughput determines the maximum load a deployment can
handle and is the primary metric for capacity planning. There is a fundamental
tension between throughput and latency: techniques that improve throughput
(larger batches, more concurrent requests) tend to increase per-request latency.

### Concurrency

The number of simultaneous requests the serving infrastructure can handle. LLM
inference holds significant GPU memory per active request because of the KV
cache (described below). Concurrency is therefore bounded by available GPU
memory, not just compute.

## The KV Cache

The KV cache (key-value cache) is the most important infrastructure concept in
LLM serving. Understanding it explains why LLM inference has such different
memory and concurrency characteristics from traditional API serving.

The Transformer attention mechanism computes key and value matrices for every
token in the sequence. When generating token N+1, the model must attend to all
previous tokens — but it has already computed their keys and values when
generating tokens 1 through N. The KV cache stores these intermediate
computations so they do not need to be recomputed on every generation step.

Without the KV cache, generating a 1,000-token response would require the model
to process the entire context sequence 1,000 times. With it, each new token
requires only one forward pass over the new token, consulting the cached keys
and values for the context. This is the difference between unusable and
practical inference latency.

The cost is memory. The KV cache for a single long-context request can consume
gigabytes of GPU memory. The size of the KV cache scales with sequence length,
batch size, the number of attention heads, and the number of layers. Managing KV
cache memory is the primary constraint on how many concurrent requests an
inference server can handle.

## Batching Strategies

Batching — processing multiple requests together in a single forward pass — is
the primary lever for improving GPU utilization and throughput in inference
serving.

### Static Batching

The simplest approach: collect a fixed number of requests, process them as a
batch, return results. Simple to implement but inefficient in practice. Requests
in a batch finish at different times; a batch cannot be dispatched until the
longest sequence completes, wasting GPU cycles waiting for a single slow
request.

### Continuous Batching (Iteration-Level Scheduling)

Continuous batching processes requests at the token level rather than the
request level. As soon as a sequence in the batch produces its final token, a
new waiting request is inserted into the batch in its place. The GPU is never
idle waiting for a slow request to finish. This technique, pioneered by vLLM, is
now the standard in production LLM serving and delivers significantly higher
throughput than static batching at comparable latency.

## Quantization

The weights of a full-precision LLM are stored as 32-bit or 16-bit floating
point numbers. Quantization reduces this precision — typically to 8-bit integers
(INT8) or 4-bit integers (INT4) — to reduce the model's memory footprint and
increase inference speed.

The trade-off is accuracy. Lower precision introduces rounding error. For most
practical purposes, INT8 quantization produces negligible quality degradation
while roughly halving memory requirements. INT4 quantization is more aggressive
and can produce measurable quality degradation on tasks requiring precise
reasoning; whether that degradation is acceptable depends on the use case and
must be verified through evaluation.

Quantization also has CI/CD implications. A quantized model is a different
artifact from the full-precision model it was derived from. It should be tracked
separately in the model registry, with its own evaluation results that confirm
the quality trade-off is acceptable for the target deployment.

## Serving Infrastructure

### Inference Servers

An inference server loads a model and exposes it as an API. It handles request
queuing, batching, KV cache management, and hardware acceleration. The inference
server is the primary deployment unit for models and the main integration point
for CD pipelines.

Common inference servers for LLMs include vLLM (optimized for continuous
batching and high throughput), NVIDIA Triton Inference Server (multi-framework,
enterprise-focused), ONNX Runtime (cross-framework, good for smaller models),
and llama.cpp (CPU-friendly, useful for edge and resource-constrained
deployments).

The choice of inference server affects performance, feature availability, and
operational complexity, but it should not affect model quality if the same
weights are served. Validating that a model produces equivalent quality results
across inference servers is a useful regression check during infrastructure
changes.

### Deployment Patterns

Model serving deployments use the same patterns as other stateful services, with
additional considerations for the cost and time required to load large model
weights.

**Canary deployment** routes a small percentage of traffic to a new model
version while the majority continues using the current version. This limits
exposure if the new model has unexpected behavior in production.

**Shadow deployment** sends copies of production traffic to a new model version
without serving its responses to users. The new model's outputs can be compared
offline against the current model, providing high-quality production data for
evaluation before any traffic is switched.

**Blue/green deployment** maintains two full serving environments and switches
traffic between them. For LLMs, the cost of maintaining two full GPU
environments simultaneously can be significant, making canary or shadow patterns
more economical for most organizations.

Model weight loading is slow — loading a large model from storage into GPU
memory takes seconds to minutes. Deployment strategies must account for this
warmup time to avoid serving errors during rollouts.

## Inference as a CI/CD Concern

The inference serving layer is where models meet users, and its properties
define the acceptance criteria for model promotion. Before a model is promoted
to production, the following questions must have documented answers:

- Does the model meet the latency SLO (e.g., p99 TTFT under a defined threshold)
  under the expected load profile?
- Does it fit within the available GPU memory budget, including KV cache at
  expected concurrency?
- If quantized, has the quantized artifact been separately evaluated to confirm
  acceptable quality?
- Does the inference server configuration (batch size, concurrency limits,
  timeouts) produce stable behavior under peak load?

These are not questions that can be answered by model quality evaluation alone.
They require load testing against a realistic serving configuration as part of
the promotion pipeline.

## Related

- [overview.md](overview.md)
- [evaluation.md](evaluation.md)
- [fine_tuning.md](fine_tuning.md)
- [AI Glossary: Inference Server](../../reference/ai_glossary.md)
- [AI Glossary: Quantization](../../reference/ai_glossary.md)
- [AI Glossary: Latency SLO](../../reference/ai_glossary.md)
- [AI Glossary: Canary Deployment](../../reference/ai_glossary.md)
- [AI Glossary: Shadow Deployment](../../reference/ai_glossary.md)
- [AI Glossary: ONNX](../../reference/ai_glossary.md)

## Links

- [Efficient Memory Management for Large Language Model Serving with PagedAttention (vLLM)](https://arxiv.org/abs/2309.06180)
- [Orca: A Distributed Serving System for Transformer-Based Generative Models (continuous batching)](https://www.usenix.org/conference/osdi22/presentation/yu)
- [NVIDIA Triton Inference Server](https://developer.nvidia.com/triton-inference-server)
- [llama.cpp](https://github.com/ggerganov/llama.cpp)
