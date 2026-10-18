# AI Agents: FAQs

| **title** | AI Agents: FAQs |
| --- | --- |
| **status** | Draft |
| **category** | technology |
| **tags** | ai-agents, automation, ci-cd |

AI agents use models to understand goals, choose actions, and interact with tools. In software delivery, they can help investigate failures, review changes, and handle tasks that require more than a fixed sequence of steps.

## Frequently asked questions

### What's the difference between a workflow and an agent?

A workflow follows a predefined sequence of steps. An agent can decide what to do next based on its goal and the information it receives. Many systems combine both: workflows provide structure, while agents handle decisions that require flexibility.

### When should I use an agent instead of a pipeline step?

Use a regular pipeline step for predictable tasks such as running tests or deploying an artifact. Use an agent when a task requires interpreting results, choosing tools, or adapting its approach. Agents add complexity, so they are most useful when that flexibility makes a real difference.

### What is an agent harness, and why does it matter?

A harness connects the model to the tools and systems it needs to do its work. It also manages execution, permissions, and limits. Without this layer, a model can generate a response but cannot safely manage a complete tool-using workflow on its own.

### How should I think about agent security?

An agent's access determines what it can do if it makes a mistake or follows malicious instructions. Limit its tools, credentials, and permissions to what the task requires. For sensitive actions, use independent policy checks and approval controls rather than relying on the model to follow instructions.

### When are multi-agent systems useful?

Multi-agent systems divide work among agents with different responsibilities. For example, one agent might investigate a build failure while another reviews security risks. They are useful when tasks benefit from specialization, but coordination costs and conflicting results can make a single agent the better choice.

### Are agents predictable?

Not always. An agent may choose different tools or actions when given the same task, especially when model outputs or external results vary. Execution limits, logs, and evaluations help teams understand what happened and keep behavior within acceptable boundaries.

### Should agents be allowed to deploy to production?

Only when the use case and safeguards justify that level of access. An agent that recommends a deployment has a different risk profile from one that executes it. Permissions, policy checks, and human approvals should reflect the potential impact of a mistake.

## Related documentation

- [AI Glossary](../../reference/ai_glossary.md)
- [Agent explanations](../agents/)
- [LLM FAQ](llm.md)
- [AI Security FAQ](security.md)
- [MLOps FAQ](mlops.md)
