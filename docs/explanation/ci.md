# Markdownlint CI Implementation

## Overview

The `Markdown Lint` workflow in `.github/workflows/markdownlint.yaml` runs
markdownlint against every Markdown file in the repository on pull requests and
on pushes to `main`. It enforces the lint rules defined in `.markdownlint.json`
so that documentation quality does not depend on each contributor running the
linter locally.

The workflow exists because:

- ADR 3 (`docs/explanation/arch/adr-3-documentation-style.md`) requires that the
  documentation standard be "enforced by linting tooling" and states that
  linting can be "automated and enforced in CI".
- AGENTS.md Rule 4 requires markdownlint to pass before a documentation task is
  considered complete. Running it in CI makes that rule verifiable for every
  change, including changes made by agents or contributors who skipped the local
  step.

## Triggers

The workflow runs on three events:

- `pull_request`: every pull request, regardless of target branch or changed
  files.
- `push` to the `main` branch: verifies the state of `main` after merges.
- `workflow_dispatch`: manual runs from the Actions tab.

A `concurrency` group keyed on the workflow name and git ref cancels any
in-progress run when a newer run starts for the same ref, so superseded pushes
do not consume runner time.

The job requests only `contents: read` permission and checks out the repository
without persisting credentials.

## Configuration Source

Lint rules come from `.markdownlint.json` at the repository root. The workflow
passes this file explicitly with `--config .markdownlint.json`, so the CI result
matches the local result. Changing a rule in that file changes both.

The current configuration enables all default rules and disables or relaxes four
of them: MD013 (line length), MD024 (duplicate headings, siblings only), MD033
(inline HTML), and MD041 (first line must be a top-level heading).

## Design Decisions

### Command-line interface instead of `markdownlint-cli2-action`

The workflow invokes `markdownlint-cli` directly through `npx`. The official
`markdownlint-cli2-action` was considered but not chosen. A single `npx` command
is easy to read, is identical to the command a contributor runs locally, and
needs no additional action dependency beyond the Node.js setup step.

### Version pinning

The linter is pinned to `markdownlint-cli@0.49.1`. Without a pin, `npx` would
resolve the latest published version, and a new release could introduce rules
that fail existing documents without any change to the repository. Upgrading the
linter is an explicit change to the workflow file.

Node.js is pinned to major version 24 through `actions/setup-node`.

### SHA-pinned actions

Both third-party actions are referenced by full commit SHA with the release
version kept as a trailing comment:

- `actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`
- `actions/setup-node@949feb2413d6458794dcd2491c4babbbce0c15c1 # v7.1.0`

A tag such as `@v7` can be moved by its maintainer. A commit SHA is immutable,
so the workflow runs the same code until someone changes the reference on
purpose.

### No `paths` filter

The workflow does not limit itself to changes under `docs/` or to `*.md` paths.
Markdown files exist outside `docs/` (for example `README.md` and `AGENTS.md` at
the root), and a path filter would be easy to get wrong when new directories are
added. The job takes a few seconds, so running it on every pull request costs
little.

### Check mode instead of `--fix`

CI runs markdownlint in check mode. It reports violations and exits non-zero but
never edits files. Auto-fixing in CI would push changes that the author never
reviewed and would hide the cause of the failure. Contributors fix violations
locally with `--fix` and commit the result.

## Running the Same Check Locally

From the repository root, run:

```bash
npx --yes markdownlint-cli@0.49.1 --config .markdownlint.json "**/*.md"
```

This is the exact command the workflow runs. To auto-fix a single file before
committing, use the command from AGENTS.md Rule 4:

```bash
markdownlint --fix --config .markdownlint.json "${FILE}"
```

## Known Impact

Pull request #9 contains Markdown that does not pass the rules enabled in
`.markdownlint.json`. Once this workflow is merged, that pull request will fail
the check until these two fixes are made on its branch:

- MD012 (multiple consecutive blank lines) in `docs/how-to/README.md`.
- MD047 (files must end with a single newline) in `docs/tutorials/README.md`.

These are mechanical fixes. Run `markdownlint --fix` on the two files, or apply
the change by hand.

## Out of Scope

- **Prettier enforcement.** AGENTS.md Rule 4 also requires
  `prettier --write --parser markdown --prose-wrap always`. This workflow checks
  only markdownlint. Adding a Prettier check in CI would be a separate change
  with its own formatting rules to settle.
- **Branch protection.** The workflow reports a status check, but marking it as
  required for merging to `main` is a repository setting. It must be configured
  in GitHub branch protection rules and is not part of this change.
