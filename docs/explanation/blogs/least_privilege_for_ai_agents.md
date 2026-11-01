# Least Privilege for AI Agents

| **status**   | Draft                             |
| ------------ | --------------------------------- |
| **category** | explanation                       |
| **tags**     | ai-agents, least-privilege, ci-cd |

An agent in a pipeline is a principal. It has an identity, credentials, and the
ability to take actions that other systems will trust. The fact that its
decisions are made by a language model rather than a script does not change the
security model. It changes the failure mode.

A script does exactly what it was written to do. An agent can be persuaded to do
something else. That difference is the reason agent permissions deserve more
scrutiny than the permissions of a normal pipeline step, not less.

## Why broad credentials happen

Agents are usually introduced to solve a workflow problem, and the shortest path
to a working demo is a token with broad access. The agent needs to read the
repository, so it gets a token that can also write. It needs to check a
deployment, so it gets credentials that can also deploy. It needs to look up
documentation, so it gets unrestricted network access.

None of these decisions looks unreasonable in isolation. Together they produce
an entity that can read private code, act on the internet, and change
production, all while processing text written by anyone who can open a pull
request. That is the same combination that makes prompt injection dangerous.

## Scoping the capability, not the prompt

Instructions in a system prompt are advisory. An agent can be talked out of
them, and a model that follows them today may not follow them tomorrow. Scope
the capability instead, so that the worst outcome is bounded even when the
instructions fail:

- **One identity per run.** Do not share a long-lived agent account across
  workflows. Issue short-lived credentials scoped to the job, and let them
  expire.
- **Read and write are different jobs.** A review agent needs read access. If it
  also needs to comment, give it a token that can comment and nothing else.
- **Bind tools to the task.** The tool registry for a triage agent should not
  include deployment tools because they might be useful later.
- **Remove ambient credentials.** The agent should not inherit the runner's
  cloud identity or the developer's local configuration.
- **Constrain the filesystem and network.** Limit writable paths to a workspace
  and restrict egress to known endpoints. Exfiltration requires a destination.
- **Separate the orchestrator from the executor.** The component that plans does
  not need the same permissions as the component that runs commands.

These controls are enforced by the harness, not by the model. That is the point.
If the only thing standing between an injected instruction and a production
change is a sentence in a prompt, there is no control at all.

## Approval gates and break glass

Some actions are too consequential to delegate without a human in the loop. A
useful rule is to require approval for anything that persists beyond the job:
pushing commits, merging, deploying, deleting resources, rotating secrets, or
changing permissions.

Approval should be specific. "Approve agent actions" is not a meaningful gate,
because the approver cannot evaluate an unbounded set of actions. Show the
proposed action, the target, and the reason, and require approval for that
action only.

Keep a documented break glass path for emergencies, and make every use of it
visible. An approval process that blocks urgent work will be bypassed, and a
bypassed process teaches the organization that the controls are optional.

## Audit trails that answer questions

Agent logs are often a transcript of a conversation, which is hard to review and
harder to query. Prefer structured events for each tool call: the agent
identity, the run identifier, the tool, the arguments, the policy decision, the
approver if any, and the result.

With that in place, you can answer the questions that matter after an incident:

- Which run made this change?
- What did the agent read before it acted?
- Was the action approved, and by whom?
- Which credential was used, and when did it expire?

Correlate the events with the pipeline run and store them somewhere the agent
cannot modify. An audit log that the agent can write to is not an audit log.

## A worked example

A triage agent reads new issues and suggests labels. A reasonable scope looks
like this:

- read access to issues in one repository
- permission to add labels, but not to close, edit, or comment on issues
- no repository contents token
- no cloud credentials
- egress limited to the issue tracker API and the model endpoint
- a spend and time limit per run

The agent is still useful. It is also unable to leak repository contents,
because it never had them, and unable to publish anything, because it has no
destination and no write scope beyond labels. If the model is fooled by an issue
body, the worst outcome is a wrong label.

That is what least privilege looks like for an agent: not a better instruction,
but a smaller blast radius.

## Related documentation

- [Agent Trust and Security](../agents/trust_and_security.md)
- [AI Security FAQ](../faq/security.md)
- [Prompt Injection in CI/CD](prompt_injection_in_cicd.md)
- [AI Agents FAQ](../faq/agents.md)
