# Least Privilege for AI Agents

| **status**   | Draft                             |
| ------------ | --------------------------------- |
| **category** | explanation                       |
| **tags**     | ai-agents, least-privilege, ci-cd |

An agent in a pipeline is a principal. It has an identity, credentials, and the
ability to take actions that other systems will trust. The fact that its
decisions are made by a language model rather than a script does not change the
security model. It changes the failure mode.

A script is predictable. It does exactly what it was written to do, even when
that is wrong. It can still have bugs and injection flaws: a workflow that
interpolates `${{ github.event.pull_request.title }}` into a shell command is
the classic CI example. But you cannot convince a script to do something else by
asking it nicely. An agent can be persuaded. That difference is the reason agent
permissions deserve more scrutiny than the permissions of a normal pipeline
step, not less.

## Why broad credentials happen

Agents are usually introduced to solve a workflow problem, and the shortest path
to a working demo is a token with broad access. The agent needs to read the
repository, so it gets a token that can also write. It needs to check a
deployment, so it gets credentials that can also deploy. It needs to look up
documentation, so it gets unrestricted network access.

None of these decisions looks unreasonable in isolation. Together they produce
an entity that can read private code, act on the internet, and change
production, all while processing text written by anyone who can open a pull
request. That is the same combination that makes prompt injection dangerous: the
[lethal trifecta](../agents/trust_and_security.md#the-lethal-trifecta) of
private data, untrusted content, and an outbound path.

## Scope the capability, not the prompt

Instructions in a system prompt are advisory. An agent can be talked out of
them, and a model that follows them today may not follow them tomorrow. Scope
the capability instead, so that the worst outcome is bounded even when the
instructions fail. These controls are enforced by the harness, not by the model.
If the only thing standing between an injected instruction and a production
change is a sentence in a prompt, there is no control at all.

### Give the agent an identity, not a copy of yours

Do not share a long-lived agent account across workflows. Give each run its own
identity through workload identity federation, such as OIDC token exchange with
the pipeline's own identity provider, and mint short-lived credentials that are
scoped to the job and expire when it ends. The mechanism matters because it
removes the long-lived secret that would otherwise be copied into a runner, a
configuration file, or a context window.

Remove ambient credentials at the same time. An agent should not inherit the
runner's cloud identity, the developer's local configuration, or a mounted
credential directory. A review agent that runs on a shared runner should not be
able to reach the same cloud account that the runner uses to publish artifacts.

### Make reads and writes different jobs

Read access is not the same capability as write access, and a review agent
rarely needs both. A review agent needs to read the repository and post a
comment. Give it a token that can read and comment, and nothing else. If the
comment has to become a commit, make that a separate job with its own approval
step and its own credential.

The same rule applies to tools. The tool registry for a triage agent should not
include deployment tools because they might be useful later. A tool that is not
registered cannot be called, regardless of what the model has been told to do.
The
[trust and security explanation](../agents/trust_and_security.md#structural-boundaries-vs-policy)
calls this a structural boundary, and it is stronger than a policy that asks the
agent to refrain.

### Separate the orchestrator from the executor

The component that plans does not need the same permissions as the component
that runs commands. A planner that reads the task, chooses a sequence of steps,
and writes a plan can work with a read-only view of the environment and no
outbound network. The executor that runs the steps can have narrow write access
to the specific systems involved, and nothing more.

This split gives you two smaller principals instead of one large one. It also
gives you a useful checkpoint: the plan is data that can be reviewed, logged, or
gated before the executor touches anything. If the planner is injected, the
executor still only exposes the operations it was built to perform. If the
executor is compromised, the planner's read-only view limits what was available
to the attack in the first place.

### Approval gates and break glass

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

## A worked example: the labeling agent

A triage agent reads new issues and suggests labels. A reasonable scope looks
like this: read access to issues in one repository, permission to add labels but
not to close, edit, or comment on issues, no repository contents token, no cloud
credentials, egress limited to the issue tracker API and the model endpoint, and
a spend and time limit per run. The agent is still useful. It cannot leak
repository contents because it never had them.

The label permission looks harmless until you follow it downstream. Many
repositories key automation on labels: a `deploy-target` label can trigger a
deployment, a `security` label can page an on-call engineer, and a
`priority/critical` label can jump a queue. An agent that can add labels can
therefore start work that nobody asked it to start. If the model is fooled by an
issue body, the worst outcome is not a wrong label. It is a deployment that no
human approved.

Scope for that outcome. Either the agent proposes a label for a human to apply,
or the labels it can write are limited to a set with no automation attached. The
harness should enforce which labels are writable, not the prompt. That is what
least privilege looks like for an agent: not a better instruction, but a smaller
blast radius.

## What to do next

Pick one agent in a pipeline and list the credentials it can reach and the
actions it can take. Then remove everything that is not needed for its stated
job. Start with the write capabilities and the ambient credentials, because
those turn a wrong answer into a production change.

If you want to compare notes with other teams doing this, see the
[ways to get involved](../../../README.md#new-members) in the repository README,
or open an issue describing the agent and the permissions you are trying to
reduce.

## Related documentation

- [Agent Trust and Security](../agents/trust_and_security.md)
- [AI Security FAQ](../faq/security.md)
- [Prompt Injection in CI/CD](prompt_injection_in_cicd.md)
- [Securing the AI Supply Chain](securing_the_ai_supply_chain.md)
- [AI Agents FAQ](../faq/agents.md)
