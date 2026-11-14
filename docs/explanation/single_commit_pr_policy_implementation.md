# Single Commit Pull Request Policy Implementation

## Overview

CONTRIBUTING.md now has a "One Commit Per Pull Request" section. It asks that
every pull request contain exactly one commit, and shows how to squash a branch
into one commit and update the pull request with `git push --force-with-lease`.

## Background

Reviewers were already asking contributors to squash their commits, for example
on [PR #38](https://github.com/cdfoundation/AI/pull/38), but the expectation was
not written down. A contributor on that pull request asked whether this was
repository policy and suggested recording it. The first version of
[PR #39](https://github.com/cdfoundation/AI/pull/39) added the rule to
AGENTS.md. Review pointed out that AGENTS.md is for AI agents, and that a squash
and force-push instruction there contradicts its rule that agents do not run git
commands. The rule was moved to CONTRIBUTING.md, and AGENTS.md is unchanged.

## Audience and Location

- The rule is aimed at the people who open pull requests, so it lives in
  [CONTRIBUTING.md](../../CONTRIBUTING.md), which GitHub links from the pull
  request page.
- [AGENTS.md](../../AGENTS.md) still tells agents not to run git commands. The
  contributor who submits the pull request performs the squash, including for
  agent-assisted changes.

## Rationale

- The repository allows only squash merges; merge commits and rebase merges are
  disabled.
- GitHub builds the squash commit message from the messages of all commits on
  the branch. With one commit, the reviewed message is the one that lands on
  `main`.
- One commit gives reviewers a single message and set of trailers to check.

## Relation to Commit Message Style

[ADR 2](arch/adr-2-commit-message-style.md) and the
[commit guidelines](../reference/commits.md) define how a commit message is
written. This policy defines how many commits a pull request carries. The two
are complementary: the single commit must follow the ADR 2 style. No ADR was
written for this rule because it records an existing review practice rather than
a new decision.

## Enforcement

No CI check or branch protection rule enforces a single commit. The policy is
documentation of an existing review practice; reviewers may ask for a squash
before approving.

## Related Work

[PR #28](https://github.com/cdfoundation/AI/pull/28) added the full
CONTRIBUTING.md. This change adds one section to that file, between
"Contribution Workflow" and "Signed Commits", and leaves the rest of the file
unchanged.
