# Guidelines for Commits

The AI repository uses automation to tag, create changelogs, and release reports
for projects. In order for automation to do its job, the format of the
`git commit` messages must follow very precise rules. For this we have decided
messages should follow the
[Conventional Commits Spec](https://www.conventionalcommits.org/en/v1.0.0) as a
baseline. Following these rules has benefits beyond automation. It leads to more
readable messages that are easy to follow when looking through the project
history. It also removes any doubt about the intention of the commit.

## Commit Message Format

Each commit message consists of a first line (type, optional scope, and
description), an optional body, and optional footers:

```text
<type>[optional scope]: <description>
<BLANK LINE>
[optional body]
<BLANK LINE>
[optional footer(s)]
```

Rules for each part:

- **type**: lowercase, required. Must be one of the types listed in
  [Commit Types](#commit-types) below.
- **scope**: lowercase, optional. Placed in parentheses after the type, e.g.
  `feat(api): ...`. Describes the area of the codebase affected.
- **description**: lowercase, imperative mood, no trailing punctuation. The
  total length of the first line must not exceed 50 characters. A correctly
  formed description should complete the sentence: "If applied, this commit will
  _description_."
- **body**: optional free-form text providing additional context. Each line must
  not exceed 72 characters. May span multiple lines.
- **footers**: optional key-value metadata as defined by the Conventional
  Commits specification (e.g. `Ref: #42`, `BREAKING CHANGE: <description>`).

Example messages:

```text
docs: add git tagging rules
```

```text
feat(api): add streaming response support

Add server-sent event support to the inference endpoint so that clients
can receive tokens incrementally rather than waiting for full completion.

Ref: #108
```

Further details on the message format are recorded in
[adr-2](../explanation/arch/adr-2-commit-message-style.md).

## Revert

If the commit reverts a previous commit, it should begin with `revert:`,
followed by the first line of the reverted commit. The body should say "This
reverts commit \<hash\>.", where the hash is the SHA of the commit being
reverted.

## Commit Types

The type must be one of the following:

- build: Changes that affect the build system or external dependencies
- chore: Changes to the repo that do not affect the code or binaries
- ci: Changes to CI configuration files and scripts
- docs: Documentation only changes
- feat: A new feature
- fix: A bug fix
- perf: A code change that improves performance
- release: A commit for bundling smaller changes into a version bump (no code
  changes)
- refactor: A code change that neither fixes a bug nor adds a feature
- style: Changes that do not affect the meaning of the code (white-space,
  formatting, missing semicolons, etc)
- test: Adding missing tests or correcting existing tests

## Tagging Rules

Definition of the Semantic Versions and commits that increment major, minor, and
patch. The semver will be used in the tag of the commit. We do not support
semver pre-release tags.

**Major.Minor.Patch** is represented as `v2.1.15`

### Major

- Any commit with a `BREAKING CHANGE` footer, or a `!` after the type (e.g.
  `feat!: remove legacy endpoint`)

### Minor

- feat

### Patch

- fix
- refactor
- perf
- test
- style
- chore
- ci
- build
- docs
- release

## Tagging Example

```bash
git checkout main
git pull
git tag -n
git tag -a v0.1.8 -m "refactor: simplify inference pipeline 0.1.8"
git tag -n
git push --tags
```
