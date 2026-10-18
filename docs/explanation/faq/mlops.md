# MLOps and Model Delivery: FAQs

| **title** | MLOps and Model Delivery: FAQs |
| --- | --- |
| **status** | Draft |
| **category** | technology |
| **tags** | mlops, model-delivery, ci-cd |

MLOps brings software engineering practices to the development and operation of machine learning models. Model delivery adds the processes needed to evaluate, promote, deploy, and monitor models reliably.

## Frequently asked questions

### What is continuous training, and when is it useful?

Continuous training means retraining a model as new data becomes available or its performance changes. It is useful when real-world conditions evolve, such as when customer behavior changes. Retraining should happen when there is a clear need, not simply because new data exists.

### What does a model registry do?

A model registry keeps track of models and their versions, along with relevant information such as evaluation results, training history, and ownership. It helps teams identify which model is being tested or deployed and understand where it came from.

### What is an evaluation gate?

An evaluation gate checks whether a model meets agreed requirements before it moves to the next stage. These requirements might include prediction quality, safety, latency, or cost. Passing the gate provides evidence that the model is ready for further testing or deployment, though it cannot guarantee perfect performance.

### How is model promotion different from software release promotion?

Software promotion generally moves the same artifact through environments after testing. Model promotion also considers how well the model performs against defined evaluation criteria. A model that loads successfully is not necessarily a model that produces good results.

### What is data drift?

Data drift occurs when incoming data changes compared with the data used as a reference, such as the training data. For example, a model may start receiving customer requests about products that did not exist when it was trained. Drift can signal a need for investigation, but it does not always mean the model performs worse.

### What is model drift?

Model drift is commonly used to describe a decline in a model's performance or relevance over time. This can happen when real-world behavior changes and the model's predictions become less accurate. Teams can identify it by monitoring model outcomes and comparing them with expected results.

### What's the difference between data drift and model drift?

Data drift means the incoming data has changed. Model drift generally means the model's performance has changed. One does not automatically imply the other: data can change without affecting accuracy, and accuracy can decline even when basic data distributions appear stable.

### Does every model need continuous training?

No. Models operating in stable environments may not need frequent retraining. Retraining also introduces cost and can make performance worse if the new data is poor or unrepresentative. The right approach depends on how quickly conditions change and how well the model continues to perform.

### Why do models need monitoring after deployment?

Pre-release tests cannot cover every real-world situation. Monitoring helps teams spot changing data, declining performance, and operational issues after deployment. These signals help determine whether a model needs investigation, retraining, or replacement.

## Related documentation

- [AI Glossary](../../reference/ai_glossary.md)
- [LLM FAQ](llm.md)
- [AI Agents FAQ](agents.md)
- [AI Security FAQ](security.md)
