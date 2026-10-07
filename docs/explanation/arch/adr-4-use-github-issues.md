# ADR 4 - Use GitHub Issues to Track Work

## History

- Status: accepted
- Deciders: Brett Smith <@xbcsmith>
- Date: 2026-10-07

## Context and Problem Statement

The CDFoundation CI/CD AI SIG needs a way to track proposals, tasks, guidance
topics, and deliverables across contributors. Without a shared, discoverable
system, work gets lost, contributors cannot find items to pick up, and progress
toward SIG goals is difficult to measure. How should the SIG track and
coordinate its work?

## Decision Drivers

- The project is already hosted on GitHub; work tracking should live in the same
  place as the repository
- All SIG work is public and community-driven; the tracking system must be
  accessible to anyone without additional accounts or access requests
- New contributors need a discoverable backlog to find ways to contribute
- Work items must be linkable to pull requests and commits so that context is
  not lost when changes are merged
- The system must be maintainable with no operational overhead for SIG members

## Considered Options

- GitHub Issues with GitHub Projects for lightweight triage and prioritization
- A third-party project management tool (e.g., Jira, Linear, Trello)
- Mailing list and meeting agendas only, with no formal issue tracking

## Decision Outcome

Chosen option: "GitHub Issues with GitHub Projects", because the project is
already on GitHub, Issues are public by default, require no additional tooling
or accounts, and integrate directly with pull requests and commits. GitHub
Projects provides enough lightweight triage (status columns, labels, milestones)
for a community SIG without introducing external dependencies.

### Positive Consequences

- All work is visible to the public; anyone can browse open issues, comment, and
  self-assign
- Issues link directly to pull requests and ADRs, keeping context in one place
- No additional accounts, costs, or administrative overhead
- Contributors can open issues to propose guidance topics, flag gaps, or start
  discussions without needing commit access
- GitHub Projects boards provide lightweight milestone and status tracking on
  top of Issues when needed

### Negative Consequences

- GitHub Issues lacks advanced project management features such as story points,
  time tracking, and dependency graphs
- Without consistent labeling and triage discipline, the backlog can become
  difficult to navigate
- Notifications and signal-to-noise ratio require active management as the SIG
  grows

## Pros and Cons of the Options

### GitHub Issues with GitHub Projects

- Good, because it is native to the repository with no additional setup
- Good, because it is free and publicly accessible to all contributors
- Good, because issues integrate with pull requests, commits, and ADRs
- Good, because GitHub Projects adds optional lightweight triage without leaving
  GitHub
- Bad, because it lacks advanced project management features needed by larger
  teams
- Bad, because backlog quality depends on consistent labeling and triage habits

### Third-party project management tool

- Good, because it offers richer features such as story points, sprints, and
  dependency tracking
- Good, because some tools provide better reporting and roadmap views
- Bad, because it requires contributors to create and maintain separate accounts
- Bad, because it fragments context between the tool and the GitHub repository
- Bad, because it introduces operational overhead and potential costs for the
  SIG

### Mailing list and meeting agendas only

- Good, because it requires no tooling or setup
- Good, because it is familiar to open-source contributors accustomed to
  mailing-list-driven projects
- Bad, because there is no persistent, searchable backlog for new contributors
  to discover open work
- Bad, because items discussed in meetings or on the mailing list are easily
  lost with no formal tracking
- Bad, because progress toward SIG deliverables becomes opaque to the broader
  community

## Links

- [GitHub Issues documentation](https://docs.github.com/en/issues)
- [GitHub Projects documentation](https://docs.github.com/en/issues/planning-and-tracking-with-projects)
- [CDFoundation AI SIG repository](https://github.com/cdfoundation/sig-ai)
