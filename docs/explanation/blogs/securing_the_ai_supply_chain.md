# Securing the AI Supply Chain

| **status**   | Draft                                      |
| ------------ | ------------------------------------------ |
| **category** | explanation                                |
| **tags**     | ai-security, supply-chain, model-artifacts |

A delivery pipeline is only as trustworthy as the artifacts it promotes. Most
organizations have spent years building controls around container images and
source code: pinned digests, signatures, provenance attestations, and admission
checks. AI systems add new artifact types to that list, and the controls do not
transfer without some thought.

## An AI system is more than code

When people say "the model", they usually mean one file. A working AI system
includes at least the following:

- model weights or adapters
- tokenizers and configuration files
- training and evaluation datasets
- prompts, agent instructions, and tool definitions
- inference container images and runtime libraries
- evaluation suites and their expected results

Each of these can change the behavior of the system, and each one can be
replaced independently. A signed container image that loads unsigned weights
from a mutable URL is not a secure delivery chain.

## What changes when the artifact is a model

Traditional build artifacts are usually reproducible from source. Model weights
are not. They are large binary files produced by a training process that is
expensive to repeat, often with data that cannot be redistributed. You cannot
rebuild them to verify a hash, and reading them tells you very little about what
they do.

The practical consequence is that provenance and testing carry more weight than
inspection. You need to know where an artifact came from, who produced it, and
what it was evaluated against. You also need to treat model behavior as part of
the artifact, not as something that will be caught later by a code review.

## Threat vectors worth naming

- **Model substitution.** A pipeline references a model by a mutable tag or URL,
  and the contents change between approval and deployment.
- **Unsafe deserialization.** Some model formats can execute code when loaded.
  Prefer formats that are data-only, or load untrusted artifacts in a sandbox.
- **Dataset poisoning.** Training or fine-tuning data is modified to introduce
  behavior that normal evaluation does not detect.
- **Dependency confusion in ML libraries.** Model loaders and tokenizer packages
  pull in transitively resolved dependencies that are not reviewed.
- **Configuration tampering.** The weights are unchanged, but the prompt, tool
  definition, or sampling configuration is edited to change behavior.
- **Tool and plugin supply chain.** An agent's tools are software too, and often
  run with more access than the agent itself.

## Controls that map to the pipeline

The good news is that the shape of the solution is familiar, even if the details
differ:

- **Reference artifacts by digest.** Pin model files and inference images by
  content hash, not by tag. Store them in a registry you control.
- **Sign and verify.** Sign the artifacts you produce, and verify signatures
  before promotion and again at admission time. Signing proves origin, not
  intent, so pair it with evaluation.
- **Generate an AI bill of materials.** Record models, datasets, libraries, and
  external services in a machine-readable inventory. Treat it as a starting
  point for review, not a guarantee of safety.
- **Track dataset lineage.** Record which dataset versions produced which model
  versions, and keep the evaluation results attached to the pair.
- **Make evaluation a promotion gate.** A model that has not passed its
  evaluation suite should not advance, regardless of who signed it.
- **Version prompts and agent configuration like code.** Review changes, keep
  history, and roll back deliberately.
- **Rebuild environments from locked dependencies.** Floating ML libraries are a
  common source of unreviewed code in inference paths.

## A promotion flow

A minimal flow that applies these controls looks like this:

```text
train or import
    -> record dataset and configuration
    -> scan artifact and dependencies
    -> run evaluation suite
    -> sign artifact with pipeline identity
    -> publish to registry with digest
    -> verify signature and evaluation at admission
    -> deploy canary with monitoring
```

Each arrow is a control point. The important property is that the decision to
promote is based on evidence produced by the pipeline, not on the reputation of
the person or system that uploaded the artifact.

## Claims to avoid

An AI bill of materials does not make a model safe. A signature does not prove
that a model behaves correctly. An evaluation suite does not cover inputs that
nobody thought to include. These controls reduce specific risks, and describing
them as more than that erodes trust in the controls themselves.

What they do provide is a chain of evidence. When something goes wrong, you can
answer which artifact ran, where it came from, what it was tested against, and
who approved it. That is the same standard we already expect from the rest of
the delivery pipeline.

## Related documentation

- [Agent Trust and Security](../agents/trust_and_security.md)
- [AI Security FAQ](../faq/security.md)
- [Prompt Injection in CI/CD](prompt_injection_in_cicd.md)
- [AI Glossary](../../reference/ai_glossary.md)
