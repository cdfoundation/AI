# Large Language Models: FAQs

| **title** | Large Language Models: FAQs |
| --- | --- |
| **status** | Draft |
| **category** | technology |
| **tags** | llm, ci-cd, ai |

LLMs can help with code reviews, test generation, troubleshooting, and other software delivery tasks. Understanding how they work helps teams use them effectively without assuming they behave like traditional software.

## Frequently asked questions

### What makes an LLM different from traditional software?

Traditional software follows rules written by developers. An LLM generates responses based on patterns learned during training and the context it receives. This makes it useful for tasks that require interpretation, but its answers can be incorrect or vary between runs.

### Why can an LLM give different answers to the same question?

LLMs can use probabilistic token selection to generate responses. Settings, model versions, and serving environments can also affect the result. Even when configured for deterministic generation, identical inputs do not guarantee identical outputs in every environment.

### When should I use prompting, RAG, or fine-tuning?

- **Prompting:** Use instructions and examples to guide the model's response.
- **RAG:** Give the model relevant information from external sources, such as internal documentation.
- **Fine-tuning:** Train the model further when you need more consistent behavior for a specific task.

Start with prompting, consider RAG when the model needs additional knowledge, and evaluate fine-tuning when the other approaches do not meet your requirements.

### What does a context window limit mean?

A context window limits how much information a model can process in one request. This includes instructions, conversation history, retrieved documents, and other inputs, along with any output tokens the model must accommodate. Large inputs can consume the available space and make important details harder to surface.

### How should I version an LLM?

Track the model version alongside its prompts, configuration, tools, and relevant data sources. These components can all affect behavior. Keeping them versioned makes it easier to understand changes, investigate problems, and reproduce results where the serving environment allows it.

### How is model promotion different from a normal software release?

Software promotion usually focuses on moving a tested artifact between environments. Model promotion must also consider how well the model performs. A model that runs successfully may still produce poor results, so evaluation should be part of the promotion decision.

### Can I trust an LLM to make CI/CD decisions?

Not without appropriate checks. LLMs can misunderstand instructions or produce incorrect answers. They can assist with decisions, but actions that affect production systems, credentials, or data should have suitable authorization and safeguards.

## Related documentation

- [AI Glossary](../../reference/ai_glossary.md)
- [LLM explanations](../llm/)
- [AI Agents FAQ](agents.md)
- [AI Security FAQ](security.md)
- [MLOps FAQ](mlops.md)
