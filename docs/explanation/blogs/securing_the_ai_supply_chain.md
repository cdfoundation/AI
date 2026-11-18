# Securing the AI Supply Chain

| **status**   | Draft                                      |
| ------------ | ------------------------------------------ |
| **category** | explanation                                |
| **tags**     | ai-security, supply-chain, model-artifacts |

A pipeline builds an inference image, signs it, and verifies the signature at
admission. The image starts, downloads its weights from a URL that resolves to
whatever `latest` points at today, and begins serving predictions. Every control
in the chain passed. The artifact that actually determines the model's behavior
was never the one that was signed.

That gap is the subject of this post. A delivery pipeline is only as trustworthy
as the artifacts it promotes. Most organizations have spent years building
controls around container images and source code: pinned digests, signatures,
provenance attestations, and admission checks. AI systems add new artifact types
to that list, and the controls do not transfer without some thought.

## An AI system is more than code

When people say "the model", they usually mean one file. A working AI system
includes model weights or adapters, tokenizers and configuration files, training
and evaluation datasets, prompts and agent instructions, tool definitions,
inference container images and runtime libraries, and evaluation suites with
their expected results. Each one can change the behavior of the system, and each
one can be replaced independently. A signed image that loads unsigned weights
from a mutable URL is not a secure delivery chain, because the image is not the
thing that decides what the system does.

## Why model artifacts are different

Traditional build artifacts are usually reproducible from source. That is the
ideal, and reproducible builds are still an open problem in many ecosystems, but
the gap is much wider for model weights. Weights are large binary files produced
by a training process that is expensive to repeat, often with data that cannot
be redistributed. You cannot rebuild them to verify a hash, and reading them
tells you very little about what they do. A hash can tell you that the file has
not changed. It cannot tell you what the file learned.

The practical consequence is that provenance and testing carry more weight than
inspection. You need to know where an artifact came from, who produced it, and
what it was evaluated against. You also need to treat model behavior as part of
the artifact, not as something that will be caught later by a code review.

## Threats worth naming

### Model substitution

A pipeline references a model by a mutable tag or URL, and the contents change
between approval and deployment. This is the scenario from the opening, and it
is common because "download the latest model" is the easiest thing to write. Pin
model files by content hash, store them in a registry you control, and make the
reference that reaches production a digest rather than a name.

### Unsafe deserialization

Some model formats can execute code when loaded. Python pickle is the obvious
example: unpickling a file can run arbitrary code, and a surprising number of
model formats are built on it or on a similar mechanism. Safetensors exists
because of this problem. It is a data-only format, so loading a file cannot
execute code. When you cannot avoid a format that can execute on load, load
untrusted artifacts in a sandbox with no credentials and no network.

This is not theoretical. In February 2025, ReversingLabs found two models on
Hugging Face that ran a reverse shell from a pickle payload, using an unusual
compression format to slip past the platform's scanner. The models were removed
within a day, but the scan had missed them. See
[nullifAI](https://www.reversinglabs.com/blog/rl-identifies-malware-ml-model-hosted-on-hugging-face)
for the details.

### Dataset poisoning

Training or fine-tuning data is modified to introduce behavior that normal
evaluation does not detect. This is the hardest of these threats to control, and
the reason is structural. The data is often collected from sources outside the
organization, its provenance is recorded unevenly, and the behavior it produces
may only appear on inputs that the evaluation suite does not cover. You cannot
scan a dataset for a payload in the way you can scan a serialized object.
Controls are indirect: dataset lineage, source review, evaluation with held-out
and adversarial inputs, and the ability to retrain from a known-good snapshot.

### Unreviewed transitive dependencies

Model loaders and tokenizer packages pull in transitively resolved dependencies
that are not reviewed. This is not dependency confusion, which has a specific
meaning: a public package that shadows an internal one. It is the quieter
problem of code that arrives through a dependency you did not choose. Lock
files, dependency review, and a clean build environment from pinned requirements
help. Floating version ranges in the inference path are the common way this
happens.

### Configuration tampering

The weights are unchanged, but the prompt, tool definition, or sampling
configuration is edited to change behavior. This one is easy to miss because the
artifact you signed still verifies. Treat prompts and agent configuration as
code: review changes, keep history, and roll back deliberately.

### Tools and agent plugins

An agent's tools are software too, and often run with more access than the agent
itself. MCP servers and agent plugins are the supply chain risk most readers
will care about, because they are installed from third-party sources and then
given credentials. Apply the same controls you would apply to any dependency:
pin versions, review the source, restrict what the tool can reach, and verify
definitions at session start. The
[trust and security explanation](../agents/trust_and_security.md#tool-definition-changes-between-sessions)
covers tool definition changes and the rug pull pattern.

The postmark-mcp incident is the clearest example so far. In September 2025, an
npm package that claimed to be an MCP server for Postmark shipped fifteen normal
versions before version 1.0.16 added a single line that blind-copied every
outgoing email to an attacker-controlled address. It was the first publicly
documented malicious MCP server in the wild. It built trust over fifteen
versions that did what they promised, which is why a one-line change was easy to
miss. See
[Snyk's analysis](https://snyk.io/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/)
for the timeline.

## Provenance: what it can and cannot tell you

Provenance is the strongest available signal for model artifacts, but it is
evidence, not a verdict. A signature tells you which pipeline produced an
artifact and that the artifact has not changed since. It does not tell you that
the model behaves correctly, that the training data was clean, or that the
person who approved the run understood what it does.

Record models, datasets, libraries, and external services in a machine-readable
inventory. Two formats are worth knowing. CycloneDX publishes an
[ML-BOM capability](https://cyclonedx.org/capabilities/mlbom/) that extends its
bill of materials model to models, datasets, and ML services. SPDX 3.0 adds an
[AI profile](https://spdx.dev/implementing-an-ai-bom/) for AI systems and model
artifacts. Either is a reasonable starting point. Treat the result as a prompt
for review, not a guarantee of safety.

For imported models, the problem is harder. Most teams do not train the model
they deploy. They pull it from a public hub, and the provenance they have is
whatever the publisher chose to share. Record the digest of the exact file you
imported, the publisher identity, the model card and license, and the evaluation
results you ran yourself. Do not inherit trust from a download count. The model
card describes intent. Your evaluation is the only evidence you have about
behavior.

PoisonGPT made the point in 2023. Mithril Security modified GPT-J-6B, uploaded
it to Hugging Face under a name close to the legitimate publisher, and showed
that standard benchmarks did not catch the altered behavior. It was downloaded
more than 40 times before it was removed. See
[PoisonGPT](https://blog.mithrilsecurity.io/poisongpt-how-we-hid-a-lobotomized-llm-on-hugging-face-to-spread-fake-news)
for the experiment.

## What scanning can and cannot see

A scan of a model artifact can detect some concrete problems. It can find
serialization payloads in formats that allow code execution, match known-bad
hashes against a feed, and flag files that are not the format they claim to be.
Those are useful checks, and they should run automatically before promotion.

A scan cannot tell you what the model will do. It cannot detect a backdoor that
was trained into the weights, a subtle bias in the training data, or a prompt
that will mislead the model in a specific context. A clean scan means "no known
payload in this file", not "safe to deploy". Keep that distinction in the
promotion decision and in the way you describe the control to reviewers.

## A promotion flow

A minimal flow that applies these controls looks like this:

```text
train or import
    -> record dataset, source, and configuration
    -> scan artifact and dependencies
    -> run evaluation suite
    -> sign artifact with pipeline identity
    -> publish to registry with digest
    -> verify signature and evaluation at admission
    -> deploy canary with monitoring
```

Each arrow is a control point. The import path deserves the same steps as the
training path, with a different evidence trail: the source digest, the publisher
identity, and your own evaluation results replace the dataset lineage you would
have for a model you trained.

Admission is a Kubernetes concept, and this flow assumes a Kubernetes-style
deploy target with an admission controller. That is a fair assumption for many
organizations, but not all of them. If you run inference on virtual machines,
the equivalent control is a deployment gate or a registry policy that checks the
signature and the evaluation record before the artifact is copied to the host.
The shape is the same even when the mechanism is different.

## Claims to avoid

An AI bill of materials does not make a model safe. A signature does not prove
that a model behaves correctly. An evaluation suite does not cover inputs that
nobody thought to include. These controls reduce specific risks, and describing
them as more than that erodes trust in the controls themselves.

What they do provide is a chain of evidence. When something goes wrong, you can
answer which artifact ran, where it came from, what it was tested against, and
who approved it. That is the same standard we already expect from the rest of
the delivery pipeline.

## What to do next

Take the model reference that is currently in production and replace the tag
with a digest. Then check what else the inference path downloads at runtime,
because that is where the unsigned weights usually come from. Those two changes
remove the most common version of the opening scenario.

If you want to work on this with other teams, see the
[ways to get involved](../../../README.md#new-members) in the repository README,
or open an issue with the artifact types and registries you are trying to cover.

## Related documentation

- [Agent Trust and Security](../agents/trust_and_security.md)
- [AI Security FAQ](../faq/security.md)
- [Prompt Injection in CI/CD](prompt_injection_in_cicd.md)
- [Least Privilege for AI Agents](least_privilege_for_ai_agents.md)
- [AI Glossary](../../reference/ai_glossary.md)
