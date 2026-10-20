# ADR 5 - Require Signed Commits

## History

- Status: proposed
- Deciders: Brett Smith <@xbcsmith>
- Date: 2026-10-20

Technical Story: <https://github.com/cdfoundation/AI/issues/24>

## Context and Problem Statement

The CDFoundation CI/CD AI SIG produces guidance that other teams will adopt,
including guidance on software supply chain integrity and secure delivery
controls. The repository should model the practices it recommends. Today, Git
author and committer fields are plain text that anyone can set to any value, so
nothing in the repository history proves that a commit was made by the person it
claims to be from.

The project is also expected to receive contributions from AI coding agents
operating on behalf of contributors (see `AGENTS.md`). Provenance of those
changes matters: a reviewer needs to know which human or identity stands behind
a commit.

Should the project require that every commit merged into the repository carries
a verified cryptographic signature, and if so, how is that enforced?

## Decision Drivers

- The SIG advocates for supply chain security and provenance; the repository
  should follow its own guidance
- Author and committer identity in Git is unauthenticated and trivially spoofed
- Contributors are public and self-declared (see the README), so there is no
  pre-existing trust relationship to rely on
- Commits produced with the help of AI agents need a clear, accountable identity
  attached to them
- The requirement must be enforceable by tooling rather than by reviewer
  vigilance
- The barrier for new contributors must stay low, and the setup must be
  documented clearly in `CONTRIBUTING.md`
- The solution should use features GitHub already provides, with no additional
  infrastructure to operate

## Considered Options

- Require verified signatures on all commits, enforced with GitHub branch
  protection
- Require a Developer Certificate of Origin (DCO) `Signed-off-by` trailer only
- Encourage signed commits but do not require them

## Decision Outcome

Chosen option: "Require verified signatures on all commits, enforced with GitHub
branch protection", because it is the only option that cryptographically binds a
commit to an identity, it is enforceable by the platform, and it matches the
supply chain practices this SIG recommends.

The following rules apply:

1. Every commit in a pull request must be signed, and GitHub must report it as
   "Verified".
2. Any signing method supported by GitHub is acceptable: GPG, SSH, or S/MIME.
   The project does not mandate one method. SSH signing is the lowest-friction
   choice for most contributors because the same key can be used for
   authentication.
3. The signing key must be registered on the contributor's GitHub account, and
   the commit email must match a verified email on that account.
4. Maintainers will enable the "Require signed commits" rule on the `main`
   branch protection (or an equivalent repository ruleset).
5. Because the rule requires signatures GitHub can verify, pull requests are
   merged with "Squash and merge" or "Create a merge commit". Commits created by
   GitHub's web interface for the merge are signed by GitHub itself.
6. `CONTRIBUTING.md` documents the requirement and links to the setup
   instructions.
7. Commits authored with the help of an AI agent are signed with the key of the
   human contributor who submits them. Agents must not be given access to
   signing keys, and agent-created commits must be re-signed by the contributor
   before the pull request is opened.

This decision does not replace the commit message conventions in
[ADR 2](adr-2-commit-message-style.md); both apply to every commit.

### Positive Consequences

- Commit authorship in the project history can be verified, not just asserted
- Impersonation of a contributor or maintainer through a forged author field is
  detected by GitHub and blocked by branch protection
- The repository demonstrates the provenance practices the SIG recommends
- Enforcement is automatic and requires no custom CI tooling
- Human accountability for AI-assisted changes is explicit

### Negative Consequences

- New contributors must set up a signing key before their first pull request is
  mergeable, which adds friction
- A contributor with unsigned commits on a branch must rewrite history to sign
  them, which is awkward for those unfamiliar with `git rebase`
- "Rebase and merge" cannot be used, because it creates new commits that do not
  carry the contributor's signature
- A verified signature proves control of a key registered on the account, not
  the real-world identity of the person or that the change is trustworthy; code
  review remains necessary
- Maintainers must keep the branch protection setting enabled; it is a
  repository setting and not recorded in the repository itself

## Pros and Cons of the Options

### Require verified signatures on all commits, enforced with GitHub branch protection

- Good, because commits are cryptographically tied to an account-registered key
- Good, because GitHub enforces it at merge time with no extra infrastructure
- Good, because it supports GPG, SSH, and S/MIME, so contributors can reuse keys
  they already have
- Good, because it is consistent with the SIG's supply chain security guidance
- Bad, because it adds setup effort for first-time contributors
- Bad, because unsigned commits require history rewriting to fix
- Bad, because it restricts the available merge methods

### Require a DCO `Signed-off-by` trailer only

- Good, because it is simple to add (`git commit -s`) and widely understood
- Good, because it records the contributor's assertion that they have the right
  to submit the change
- Bad, because the trailer is plain text and can be added by anyone for any name
- Bad, because it provides no authentication of the committer
- Bad, because it addresses licensing provenance, not identity provenance

### Encourage signed commits but do not require them

- Good, because it has no barrier for new contributors
- Good, because it requires no repository configuration
- Bad, because history will contain a mix of verified and unverified commits, so
  no commit can be assumed authentic
- Bad, because voluntary controls are routinely skipped, particularly by
  automated tooling and AI agents
- Bad, because it undermines the SIG's credibility when recommending provenance
  controls

## Links

- [GitHub: Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
- [GitHub: About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [GitHub: Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [GitHub: About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [Developer Certificate of Origin](https://developercertificate.org/)
- Related to [ADR 2 - Commit Message Style](adr-2-commit-message-style.md)
- Implemented by [CONTRIBUTING.md](../../../CONTRIBUTING.md)
