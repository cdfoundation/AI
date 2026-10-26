# Contributing to the CI/CD AI Special Interest Group

Thank you for your interest in the CDF CI/CD AI Special Interest Group (SIG).
Participation is open to the public, and contributions of all sizes are welcome,
including new guidance, corrections, reference architectures, proof-of-concept
implementations, and documentation fixes.

Read the [README](README.md) first for the goals, scope, and non-goals of the
SIG.

## Getting Started

1. Join the `#sig-ai` Slack channel in the
   [CDF Slack workspace](https://cdeliveryfdn.slack.com).
2. Join the [CDF TOC mailing list](https://lists.cd.foundation/g/cdf-toc)
   mailing list and the
   [SIG mailing list](https://lists.cd.foundation/g/sig-ai).
3. Attend SIG meetings when you can.
4. Add yourself to the Members list in the [README](README.md) with a pull
   request. Membership is self-declared.

## Ways to Contribute

- Open an issue to propose new guidance topics, flag gaps, or start a
  discussion.
- Pick up an existing issue and comment to say you are working on it.
- Share your experience integrating AI workloads into CI/CD in a SIG meeting or
  on the mailing list.
- Present a project or tool your organization or community is working on.
- Propose or contribute a proof-of-concept implementation or reference
  architecture.
- Improve the documentation in [docs](docs/README.md).

## Tracking Work

The SIG tracks all work in
[GitHub Issues](https://github.com/cdfoundation/AI/issues), as described in
[ADR 4](docs/explanation/arch/adr-4-use-github-issues.md). Please open or find
an issue before starting significant work so others can see what is in progress.
Reference the issue in your pull request.

## Contribution Workflow

1. Fork the repository, add the original repository as the `upstream` remote
   (`git remote add upstream https://github.com/cdfoundation/AI.git`), and
   create a branch from `main`.
2. Make your changes. Keep each pull request focused on one topic.
3. Run the documentation quality checks (see [Documentation](#documentation)).
4. Commit using the [commit message style](#commit-messages). All commits must
   be [signed](#signed-commits).
5. Push your branch and open a pull request against `main`. Describe what
   changed and why, and link the related issue.
6. Respond to review feedback. Maintainers merge the pull request once it is
   approved and all commits are verified.

## Signed Commits

All commits must carry a verified signature. GitHub must show each commit in
your pull request as "Verified", or the pull request cannot be merged. The
reasoning is recorded in
[ADR 5](docs/explanation/arch/adr-5-require-signed-commits.md).

GitHub accepts GPG, SSH, and S/MIME signatures. SSH signing is usually the
simplest if you already use an SSH key with GitHub. Details on verification,
merge methods, and repository settings are in the
[verified commits reference](docs/reference/verified_commits.md).

### Set Up SSH Signing

SSH signing needs Git 2.34 or later. Run these commands once, replacing the key
path with your own public key:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Then add the same public key to your GitHub account as a **Signing Key** under
Settings, SSH and GPG keys. A key used for authentication is not automatically
registered for signing.

To check SSH signatures locally, tell Git which keys to trust, using the same
public key as above:

```bash
echo "$(git config user.email) namespaces=\"git\" $(cat ~/.ssh/id_ed25519.pub)" >> ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

### Set Up GPG Signing

Generate a key and tell Git to use it:

```bash
gpg --full-generate-key
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey <KEY_ID>
git config --global commit.gpgsign true
```

Then add the public key to your GitHub account. See the GitHub documentation
linked below for the export and upload steps.

### Verify Your Setup

The email address in `user.email` must be a verified email on your GitHub
account. Make a commit and check its signature locally:

```bash
git commit -m "docs: fix typo in readme"
git log --show-signature -1
```

With SSH signing, the local check needs the allowed signers file from the SSH
section. Without it, Git prints
`error: gpg.ssh.allowedSignersFile needs to be configured and exist for ssh signature verification`
followed by `No signature`, even though the commit is signed. In that case, rely
on the badge.

After you push, the commit should display a "Verified" badge on GitHub.

### Fix Unsigned Commits

If a pull request contains unsigned commits, re-sign them and update your
branch. In a fork, re-sign only your own commits. Your local `main` is usually
behind `upstream`, so a rebase that replays upstream commits re-signs them with
your key and adds them to your pull request.

First, list the commits that exist only on your branch:

```bash
git fetch upstream
git log --oneline upstream/main..HEAD
```

The list should show only your own commits, plus any merge of `upstream/main`
made by the Update branch button. If it shows commits you did not write, stop
and ask in the pull request.

Then re-sign them by rebasing onto `upstream/main`:

```bash
git rebase --committer-date-is-author-date --exec "git commit --amend --no-edit -S" upstream/main
git push --force-with-lease
```

Alternatively, keep your branch's base and re-sign everything after the last
commit that is already correct:

```bash
git rebase --committer-date-is-author-date --exec "git commit --amend --no-edit -S" -i <last_good_commit_hash>
git push --force-with-lease
```

With `-i`, Git opens an editor with the list of commits to replay. Save and
close it to continue. Use the hash of the commit just before the first commit in
the `git log` output above. See the
[verified commits reference](docs/reference/verified_commits.md).

To sign only the most recent commit, use `git commit --amend --no-edit -S`.

### Merging

Maintainers merge using "Squash and merge". GitHub signs the squashed commit, so
the contributor's signature is verified at merge time but is not kept on `main`.
Maintainers with bypass permission may instead merge locally at the command line
and push; see [ADR 5](docs/explanation/arch/adr-5-require-signed-commits.md).
"Rebase and merge" is not used because GitHub rewrites the commits and cannot
sign them with the contributor's key. The squash commit message is the pull
request title and description, so write the description as the final commit
body.

### Further Reading

- [Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
- [Telling Git about your signing key](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
- [About commit signature verification](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification)

## Commit Messages

Commit messages follow the Conventional Commits style defined in
[ADR 2](docs/explanation/arch/adr-2-commit-message-style.md) and summarized in
the [commit guidelines](docs/reference/commits.md). In short:

```text
<type>[optional scope]: <description>
<BLANK LINE>
[optional body]
<BLANK LINE>
[optional footer(s)]
```

- Use a lowercase type such as `feat`, `fix`, or `docs`.
- Write a lowercase, imperative description with no trailing punctuation.
- Keep the first line to 50 characters and body lines to 72 characters.
- Explain what changed and why in the body when it is not obvious.

## Documentation

Documentation follows the [Diataxis](https://diataxis.fr) framework, as
described in [ADR 3](docs/explanation/arch/adr-3-documentation-style.md) and the
[documentation layout](docs/reference/documentation_layout.md). Place new
documents in the directory that matches the reader's need:

| Directory           | Use for                                        |
| ------------------- | ---------------------------------------------- |
| `docs/tutorials/`   | Step-by-step lessons for new users             |
| `docs/how-to/`      | Recipes for solving specific problems          |
| `docs/explanation/` | Architecture, decisions, research, and context |
| `docs/reference/`   | Technical specifications and glossaries        |

Conventions:

- Use `.md` for Markdown and `.yaml` for YAML files.
- Name documentation files `lowercase_with_underscores.md`.
  `README.md`,`LICENSE.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`,
  `AGENTS.md`, and `CHANGELOG.md` are the only exceptions.
- Do not use emojis in code, comments, documentation, or commit messages.

Before opening a pull request, run both checks on every Markdown file you
changed:

```bash
markdownlint --fix --config .markdownlint.json "${FILE}"
prettier --write --parser markdown --prose-wrap always "${FILE}"
```

## Proposing an Architecture Decision

Significant decisions are recorded as Architecture Decision Records (ADRs), as
described in [ADR 1](docs/explanation/arch/adr-1-use-adrs.md). To propose one,
copy the [template](docs/explanation/arch/draft/adr-template.md), fill it in,
and open a pull request. See the [ADR README](docs/explanation/arch/README.md)
for the full process.

## Contributing with AI Agents

AI-assisted contributions are welcome. The contributor who submits the pull
request is responsible for the content and for the commit signatures. Agents
working in this repository must follow the rules in [AGENTS.md](AGENTS.md).
Review everything an agent produces before you commit it, and do not give agents
access to your signing keys.

## License

This project is licensed under the Apache License, Version 2.0. By contributing,
you agree that your contributions will be licensed under the same terms. See
[LICENSE](LICENSE).
