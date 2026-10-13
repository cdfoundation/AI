# AI Security: FAQs

| **title** | AI Security: FAQs |
| --- | --- |
| **status** | Draft |
| **category** | technology |
| **tags** | ai-security, prompt-injection, ci-cd |

AI systems introduce new security risks when they process untrusted content or can take actions through tools. Understanding these risks helps teams use AI in CI/CD without exposing sensitive data or granting unnecessary access.

## Frequently asked questions

### What is prompt injection, and why does it matter?

Prompt injection happens when malicious content tries to manipulate an AI system into ignoring its intended instructions. In CI/CD, that content could appear in a pull request, source file, build log, or issue. If an agent can also access sensitive data or execute commands, the impact can go beyond an incorrect response.

### What is the lethal trifecta in AI security?

The lethal trifecta describes three conditions that can combine to create a serious security risk:

- Access to private data.
- Exposure to untrusted content.
- The ability to send information to an external destination.

Together, these capabilities can allow malicious content to influence a system into exposing sensitive information. Limiting access and external communication helps reduce this risk.

### Why isn't a context window a safe place for secrets?

A context window is information the model can use to generate responses, not a secure storage system. Secrets included in prompts or tool outputs could appear in responses or be passed to other tools. Keep credentials in a dedicated secret manager and expose them only when needed.

### How should agent permissions be scoped?

Give agents only the access required for their assigned tasks. An agent reviewing code does not automatically need permission to deploy it or access production credentials. Enforce permissions outside the model, and add approval checks for sensitive actions.

### What is an AI SBOM?

An AI software bill of materials (AI SBOM) is an inventory of the components that make up an AI system. Depending on its scope, it may include models, datasets, frameworks, libraries, and external services. It helps teams understand dependencies and assess supply-chain risks, but it does not guarantee that a system is safe.

### Can a system prompt prevent prompt injection?

A system prompt can guide model behavior, but it cannot guarantee that malicious content will be ignored. Security should not depend on the model following instructions every time. Restrict tool access, validate sensitive actions independently, and keep untrusted content from gaining unnecessary privileges.

### How should secrets be handled in AI pipelines?

Store secrets in a dedicated secret-management system and grant access only when required. Avoid placing credentials directly in prompts, source files, or logs. Also consider whether model providers and connected tools retain or expose the information they receive.

## Related documentation

- [AI Glossary](../../reference/ai_glossary.md)
- [LLM FAQ](llm.md)
- [AI Agents FAQ](agents.md)
- [MLOps FAQ](mlops.md)
