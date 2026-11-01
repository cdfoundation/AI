# Security Blog Posts

| **status**   | Draft                    |
| ------------ | ------------------------ |
| **category** | explanation              |
| **tags**     | ai-security, ci-cd, blog |

This section collects short posts about security in AI-augmented delivery
pipelines. The posts are understanding-oriented: each one explains a problem and
a way to reason about it, rather than prescribing a single implementation.

Posts are not tutorials. If you are looking for definitions, start with the
[AI Glossary](../../reference/ai_glossary.md). If you are looking for direct
answers to common questions, start with the
[AI Security FAQ](../faq/security.md). If you are designing a system, the
[agent trust and security explanation](../agents/trust_and_security.md) covers
the architecture in more depth.

## Posts

| Post                                                                 | Description                                                                                        |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| [prompt_injection_in_cicd.md](prompt_injection_in_cicd.md)           | Why every artifact that reaches an agent is an input channel, and where the trust boundary belongs |
| [securing_the_ai_supply_chain.md](securing_the_ai_supply_chain.md)   | Treating models, datasets, and agent configuration as pipeline artifacts that need provenance      |
| [least_privilege_for_ai_agents.md](least_privilege_for_ai_agents.md) | Scoping agent permissions as if the agent were a service account with a natural language interface |

## Adding a post

- Use `lowercase_with_underscores.md` for the filename, as required by
  [AGENTS.md](../../../AGENTS.md).
- Keep the metadata table at the top of the file and set the status to `Draft`
  until the post has been reviewed.
- Link to related documentation at the end so readers can move from opinion to
  reference material.
- Run the quality gates from [AGENTS.md](../../../AGENTS.md) before opening a
  pull request. Both `markdownlint` and `prettier` must pass.
- Do not use the posts to recommend a specific vendor or product. The SIG is
  vendor-neutral.

## Related documentation

- [AI Glossary](../../reference/ai_glossary.md)
- [AI Security FAQ](../faq/security.md)
- [Agent Trust and Security](../agents/trust_and_security.md)
- [Documentation Layout](../../reference/documentation_layout.md)
