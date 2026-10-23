# Verified Commits Reference

Reference for how signed commits are verified, merged, and enforced in this
repository. The decision is recorded in
[ADR 5](../explanation/arch/adr-5-require-signed-commits.md). The steps for
contributors are in the Signed Commits section of
[CONTRIBUTING.md](../../CONTRIBUTING.md).

## Signing Methods

GitHub accepts three signing methods. The project does not mandate one.

| Method | `gpg.format` | Registered on GitHub as | Notes                                         |
| ------ | ------------ | ----------------------- | --------------------------------------------- |
| GPG    | `openpgp`    | GPG key                 | Default value of `gpg.format`                 |
| SSH    | `ssh`        | Signing Key             | Requires Git 2.34 or later                    |
| S/MIME | `x509`       | See GitHub docs         | Not covered by the steps in `CONTRIBUTING.md` |

For any method, the commit email must match a verified email on the GitHub
account that owns the key. An SSH key used for authentication is not
automatically registered for signing.

## Verifying Signatures Locally

The "Verified" badge on GitHub is the authority for merging. A local check is
useful for catching problems before pushing.

- **GPG**: `git log --show-signature -1` reports the signature.
- **SSH**: with `gpg.format=ssh`, Git also needs `gpg.ssh.allowedSignersFile`
  pointing to an existing file that lists the trusted public keys. Without it,
  `git log --show-signature -1` prints
  `error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification`
  followed by `No signature`, even when the commit is signed. This was
  reproduced locally during review.

An SSH user who does not want to maintain an allowed signers file can rely on
the Verified badge after pushing. The setup commands are in
[CONTRIBUTING.md](../../CONTRIBUTING.md).

## Merge Methods

| Method                 | Status                 | Signature on `main`                                                                                                          |
| ---------------------- | ---------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Squash and merge       | Allowed                | The squashed commit is signed by GitHub. The contributor's signatures are verified at merge time but not retained            |
| Create a merge commit  | Disabled               | Not applicable. "Allow merge commits" is turned off                                                                          |
| Rebase and merge       | Not used               | GitHub always creates new commit SHAs, even when a fast-forward is possible, and cannot sign them with the contributor's key |
| Local merge at the CLI | Bypass permission only | A fast-forward keeps the contributor's signed commits. A local merge commit must be signed by the maintainer                 |

Squash stays enabled because merge commits are already off. Disabling squash as
well would leave rebase as the only method, which the ADR rules out.

A local merge pushed by a maintainer with bypass permission is not checked by
the signed-commit rule, so signing that merge relies on the maintainer. Keep the
bypass list limited to maintainers.

## Repository Settings

These settings live in GitHub, not in the repository, so maintainers must keep
them enabled.

| Setting                               | Value                              |
| ------------------------------------- | ---------------------------------- |
| Require signed commits                | On, for `main`                     |
| Require a pull request before merging | On                                 |
| Block force pushes                    | On                                 |
| Allow merge commits                   | Off                                |
| Squash default commit message         | Pull request title and description |

Because a pull request is required and force pushes are blocked, only a
maintainer with bypass permission can push a local merge to `main`.

## Squash Commit Message

With the default squash message setting, a pull request with two or more commits
produces the pull request title and a list of its commits. The
`Merge branch 'main' into ...` commits created by the Update branch button never
reach `main` as commits under a squash merge, but they appear as bullets in that
default list.

With the squash default set to "Pull request title and description", the commit
on `main` is the pull request title followed by the pull request description.
The title should follow the conventional style in [commits.md](commits.md), and
the description becomes the commit body, so write it as the final commit body.

## Fork Workflow and Re-signing

In a fork, the contributor's local `main` is usually behind the upstream `main`.
If the pull request branch has merged upstream `main`, for example through the
Update branch button, rebasing onto local `main` drops those merge commits and
replays the upstream commits that local `main` lacks. Those commits are then
re-signed with the contributor's key and end up in the pull request.

Re-sign only the commits on the pull request branch, starting after the last
commit that is already correct. The command is in
[CONTRIBUTING.md](../../CONTRIBUTING.md). Rebasing onto `upstream/main` after
`git fetch upstream` also works.

## Related

- [ADR 5 - Require Signed Commits](../explanation/arch/adr-5-require-signed-commits.md)
- [CONTRIBUTING.md](../../CONTRIBUTING.md)
- [commits.md](commits.md)

## Links

- [GitHub: Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
- [GitHub: Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [GitHub: About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)
- [GitHub: Configuring commit squashing for pull requests](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/configuring-commit-squashing-for-pull-requests)
