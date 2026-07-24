# CI/CD Artificial Intelligence (AI) Special Interest Group

[Website](#) <!-- Update with hosted site URL when available -->

Artificial Intelligence is rapidly reshaping every phase of the software
delivery lifecycle—from intelligent test generation and automated code review to
model-driven deployment decisions and self-healing pipelines. As AI workloads
become first-class citizens in modern software delivery, CI/CD teams are
increasingly responsible for building the pipelines, platforms, and operational
practices that carry them from experimentation through to production. This
Special Interest Group (SIG) is dedicated to defining how AI fits into
Continuous Integration and Continuous Delivery environments, with a focus on
practical implementation guidance, secure deployment patterns, platform
engineering best practices, and the full spectrum of MLOps disciplines—including
model training, fine-tuning, inference serving, and AI agent orchestration.

The CD Foundation's CI/CD AI Special Interest Group aims to close the gap
between AI innovation and production-grade delivery by developing open,
vendor-neutral guidance for integrating AI systems into CI/CD pipelines. The SIG
will identify emerging tools, frameworks, and patterns that enable teams to ship
AI workloads safely, repeatably, and at scale.

This SIG will produce a living guide that helps platform engineers, MLOps
practitioners, DevOps teams, and AI/ML engineers build AI-ready delivery
pipelines. It will map concrete implementation patterns to the distinct
challenges of AI workloads—covering data pipelines, model registries, inference
infrastructure, agent harnesses, and the security controls needed to govern them
responsibly. Where gaps exist in current tooling or community guidance, the SIG
will collaborate with the broader ecosystem to address them.

Where the existing [CDF MLOps SIG](https://github.com/cdfoundation/sig-mlops)
focuses on data and machine learning pipelines and model deployment, this SIG
focuses on the broader integration of AI systems into CI/CD environments,
including generative AI applications, agentic workflows, inference platforms,
secure delivery controls, and AI-ready internal developer platforms.

---

## Why This SIG Is Needed

The integration of AI into software delivery introduces challenges that existing
CI/CD guidance does not fully address:

- **AI Workloads Require New Pipeline Primitives:** Training runs, fine-tuning
  jobs, model evaluations, and inference deployments have fundamentally
  different resource profiles, artifact types, and promotion gates than
  traditional application builds. Teams are adapting general-purpose pipelines
  in ad hoc ways, leading to fragile, inconsistent practices.

- **MLOps Is Maturing Rapidly but Lacks CI/CD Grounding:** The MLOps discipline
  has produced valuable tooling for experiment tracking and model registries,
  but formal integration with CI/CD pipeline standards, event-driven
  architectures, and GitOps workflows remains underspecified.

- **AI Agents and Agentic Systems Demand New Harness Patterns:** The emergence
  of autonomous AI agents—including LLM-based coding assistants, planning
  agents, and multi-step tool-using systems—introduces new operational
  requirements for sandboxing, observability, rollback, and trust boundaries
  within pipelines.

- **Security and Compliance for AI Are Unsolved at Scale:** Model supply chain
  integrity, prompt injection, data poisoning, and inference API exposure are
  active threat vectors. The CI/CD pipeline is the right control point to
  address many of these risks, yet guidance specific to AI workloads is sparse.

- **Platform Engineering Teams Are Being Asked to Enable AI Without a Map:**
  Infrastructure and platform teams are fielding requests to support GPU-backed
  build environments, inference serving infrastructure, model artifact storage,
  and AI-powered developer tooling—often without established patterns or
  reference architectures to draw from.

The evolution of CI/CD practices to natively support AI workloads is no longer
optional. Organizations across industries are accelerating AI adoption while
their delivery platforms struggle to keep pace. This SIG provides a neutral,
community-driven forum to develop the guidance that teams need.

---

## SIG Goals and Objectives

The CI/CD AI Special Interest Group aims to:

- **Define AI Integration Patterns for CI/CD:** Document implementation patterns
  for incorporating AI and ML workloads into CI/CD pipelines, including data
  ingestion pipelines, model training jobs, evaluation gates, model promotion
  workflows, and inference deployment strategies.

- **Establish MLOps Best Practices in a Delivery Context:** Develop guidance
  that bridges the MLOps and CI/CD communities, covering model versioning,
  experiment reproducibility, artifact management, model registries, and
  continuous training pipelines.

- **Provide Guidance on AI Agent Orchestration and Harnesses:** Define patterns
  for integrating AI agents and agentic systems into delivery pipelines,
  including agent harness design, sandboxing, tool boundaries, observability,
  and human-in-the-loop approval gates.

- **Advance Security Practices for AI Workloads:** Identify and promote security
  controls specific to AI delivery, including model supply chain integrity,
  secure inference serving, secrets management for model APIs, input/output
  validation, and threat modeling for AI-augmented pipelines.

- **Support Platform Engineering for AI Readiness:** Develop reference
  architectures and platform patterns that enable internal developer platforms
  (IDPs) to support AI workloads—covering compute scheduling, GPU resource
  management, model serving infrastructure, and self-service AI tooling for
  development teams.

- **Track and Catalog Emerging Tooling:** Monitor and document emerging
  open-source and vendor tools across the AI delivery ecosystem—spanning
  inference servers, agent frameworks, MLflow-compatible registries, vector
  database integrations, evaluation frameworks, and CI/CD-native AI tooling.

- **Collaborate Across Foundations and Communities:** Partner with CNCF, LF AI &
  Data, OpenSSF, and other relevant communities to align guidance, avoid
  duplication, and foster cross-ecosystem adoption of standardized AI delivery
  practices.

---

## Scope of Work

The SIG will undertake the following key activities:

**CI/CD Pipeline Integration for AI Workloads**

- Define pipeline stages, event triggers, and promotion gates specific to ML
  model development lifecycles.
- Document patterns for integrating training, evaluation, and deployment jobs
  into CI/CD systems including Jenkins, Tekton, GitHub Actions, Argo Workflows,
  and others.
- Develop guidance on data versioning and dataset pipeline management as
  first-class CI/CD concerns.

**MLOps and Continuous Training**

- Establish best practices for continuous training (CT) pipelines, including
  triggered retraining, data drift detection gates, and automated evaluation.
- Provide guidance on model registry integration, artifact promotion policies,
  and reproducible training environments.
- Document fine-tuning workflows for foundation models and LLMs within CI/CD
  pipelines, including parameter-efficient fine-tuning (PEFT) techniques such as
  LoRA.

**AI Agent Harnesses and Agentic Pipelines**

- Define harness patterns for integrating LLM-based coding agents, planning
  agents, and multi-agent systems into CI/CD workflows.
- Develop guidelines for agent sandboxing, capability scoping, tool invocation
  auditing, and rollback mechanisms.
- Document patterns for human-in-the-loop (HITL) approval gates in agentic
  pipelines.

**Inference Serving and Deployment**

- Provide guidance on deploying and managing inference servers (e.g., vLLM,
  Triton Inference Server, ONNX Runtime, llama.cpp) within CD pipelines.
- Define canary, blue/green, and shadow deployment patterns adapted for model
  serving infrastructure.
- Cover model observability, latency SLOs, and feedback loop integration for
  production inference.

**Security for AI Workloads**

- Identify and map security controls to AI-specific threat vectors: model supply
  chain attacks, prompt injection, data poisoning, API key exposure, and
  inference endpoint abuse.
- Develop guidelines for model artifact signing, provenance attestation, and
  SBOM generation for AI workloads.
- Provide secure configuration guidance for inference APIs, model hosting
  platforms, and AI-assisted developer tooling within the pipeline.

**Platform Engineering Best Practices**

- Develop reference architectures for AI-ready internal developer platforms,
  including GPU compute provisioning, shared model serving infrastructure, and
  self-service AI tooling.
- Provide guidance on integrating AI coding assistants, AI-powered test
  generation, and AI-driven observability tools into platform offerings.
- Document patterns for cost governance, resource quotas, and scheduling for
  GPU-backed pipeline workloads.

**Community Collaboration**

- Review and align guidance with relevant CNCF cloud native AI work, LF AI &
  Data projects, and OpenSSF supply chain security work.
- Identify gaps in current open-source tooling and coordinate with relevant
  project communities to address them.
- Develop white papers, reference guides, and proof-of-concept implementations
  to validate and share SIG recommendations.

## Expected Deliverables

The SIG is expected to produce practical, implementation-oriented materials,
including:

- Reference architectures for AI-ready CI/CD platforms and internal developer
  platforms.
- Example pipelines for training, fine-tuning, evaluation, model promotion,
  inference deployment, and agentic workflows.
- Agent harness patterns covering tool boundaries, sandboxing, policy
  enforcement, audit logging, and human approval gates.
- Model and dataset promotion workflows that address reproducibility, lineage,
  versioning, rollback, and drift response.
- Threat-model templates and control mappings for AI-specific risks in CI/CD
  environments.
- Guidance for signing, provenance attestation, SBOM generation, and secure
  distribution of model artifacts.
- Implementation guides for common CI/CD systems such as Jenkins, Tekton, GitHub
  Actions, Argo Workflows, and related tools.
- Proof-of-concept implementations that validate recommendations against real
  delivery workflows.

## Non-Goals

To keep the SIG focused and avoid duplicating adjacent community work, the SIG
does not aim to:

- Select preferred model providers, commercial platforms, or foundation models.
- Replace the CDF MLOps, Software Supply Chain, Cybersecurity, or other related
  SIGs and guides.
- Write regulatory policy or provide legal compliance advice.
- Benchmark model quality except where evaluation gates, deployment safety, or
  CI/CD promotion criteria require it.
- Define general-purpose AI ethics guidance outside the context of software
  delivery, platform engineering, and operational risk controls.

## Reference Frameworks and Prior Work

Reference frameworks and prior work the SIG will consider include:

- [CDF MLOps SIG](https://github.com/cdfoundation/sig-mlops)
- [LF AI & Data projects](https://lfaidata.foundation/projects/)
- [NIST AI Risk Management Framework (AI RMF)](https://www.nist.gov/itl/ai-risk-management-framework)
- [Google Research: Securing the AI Software Supply Chain](https://research.google/pubs/securing-the-ai-software-supply-chain/)
- [OWASP GenAI Security Project](https://genai.owasp.org/)
- [CDF Interoperability SIG (archived)](https://github.com/cdfoundation/sig-interoperability)
- [CDF CI/CD Cybersecurity Guide](https://github.com/cdfoundation/CICD-Cybersecurity)
- [CNCF TAG Security](https://github.com/cncf/tag-security)

---

## Audience and Participants

This SIG is open to all practitioners, researchers, and technologists working at
the intersection of AI and software delivery. Within the CDF community and
beyond, we aim to engage:

- Platform engineering and DevOps teams integrating AI workloads into existing
  delivery infrastructure
- MLOps practitioners responsible for model training, evaluation, and deployment
  pipelines
- AI/ML engineers building or consuming CI/CD-integrated training and inference
  workflows
- Security engineers developing threat models and controls for AI-augmented
  pipelines
- Open-source project communities from the CDF, CNCF, and LF AI & Data
  foundations
- CDF and OpenSSF Ambassadors
- CDF Member companies building or adopting AI-powered delivery tooling
- CDF End User Council participants representing organizations shipping AI
  products at scale
- Researchers and standards bodies developing frameworks for safe and secure AI
  deployment

---

## Members

<!-- Founding members — submit a PR to add yourself -->

- Brett Smith <@xbcsmith>
- Animesh Pathak <@sonichigo>

---

## New Members

Membership in the CDF CI/CD AI Special Interest Group is open to the public and
self-declared.

New members are advised to:

- Join the SIG mailing list. <!-- Update with list URL when available -->
- Join the CDF TOC mailing list.
- Join the `#sig-ai` Slack channel in the
  [CDF Slack workspace](https://cdeliveryfdn.slack.com).
- Review this README thoroughly.
- Submit a pull request to add yourself to the Members list above.
- Attend SIG meetings regularly.
  <!-- Update with meeting cadence and calendar link -->
- Contribute to the documentation and reference guide.

Ways to get involved:

- Share your experience integrating AI workloads into CI/CD by joining meetings
  or posting to the mailing list.
- Present a project or tool your organization or community is working on.
- Add a topic to the meeting agenda for group discussion.
- Open an issue to propose new guidance topics, flag gaps, or start a
  collaborative discussion.
- Pick up an existing issue and comment to express interest in contributing.
- Propose or contribute a proof-of-concept implementation or reference
  architecture.

---

## Governance

CI/CD AI is a
[CDF Special Interest Group](https://github.com/cdfoundation/toc/tree/main/sigs).
Governance details for CDF SIGs can be found in the
[CDF Working Groups and SIGs process](https://github.com/cdfoundation/toc/blob/main/GROUPS.md#sigs).
<!-- Update with SIG-specific governance link when established -->

---

## Disclaimer

The guidance produced by the CI/CD AI Special Interest Group is intended to
serve as a community-developed reference. All recommendations should be reviewed
and adapted to fit specific organizational requirements, risk tolerances, and
regulatory obligations.

By using this guide, you agree to do so at your own risk and discretion. The
Continuous Delivery Foundation and the Linux Foundation shall not be liable for
any direct, indirect, incidental, special, exemplary, or consequential damages
(including, but not limited to, procurement of substitute goods or services;
loss of use, data, or profits; or business interruption) however caused and on
any theory of liability, whether in contract, strict liability, or tort
(including negligence or otherwise) arising in any way out of the use of this
guide, even if advised of the possibility of such damage.
