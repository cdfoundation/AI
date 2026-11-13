# Spellcheck Implementation

<!-- cspell:ignore defence organised behaviour Communi speling misteak -->

This document describes the spellcheck stage added for
[issue #36](https://github.com/cdfoundation/AI/issues/36).

## Motivation

The repository is almost entirely Markdown and had no CI. Typos were only caught
when a reviewer noticed them. A spellcheck stage gives contributors fast
feedback without needing a reviewer.

## What Was Added

| Path                                | Purpose                                         |
| ----------------------------------- | ----------------------------------------------- |
| `cspell.json`                       | cspell configuration                            |
| `project-words.txt`                 | Project dictionary seeded with the baseline     |
| `.github/workflows/spellcheck.yaml` | Runs cspell on pull requests and pushes to main |
| `docs/how-to/run_spellcheck.md`     | Local usage and how to add words                |

`AGENTS.md` Rule 4 now includes the cspell command, so agent-authored docs are
checked too.

## Design Decisions

### Tool

cspell was chosen for its Markdown support, project dictionaries, inline ignore
directives, and a maintained GitHub Action.

### Locale

The project standardizes on US English (`en-US`). Three British spellings in the
existing docs were changed: `defence` to `defense` in
`docs/explanation/agents/agent_harness.md`, `organised` to `organized` in
`docs/README.md`, and `behaviour` to `behavior` in `docs/explanation/ci.md`. No
British spellings are added to the dictionary, since an entry would let them
through everywhere.

### Baseline Dictionary

Running cspell against `main` flagged a set of words, none of them real typos:
member names and GitHub handles, AI and MLOps terminology, and proper nouns.
These seed `project-words.txt`. The one exception was `Communi-`, a word split
by a hyphen in an ASCII diagram in
`docs/explanation/agents/trust_and_security.md`. The diagram was reflowed to
show the whole word rather than adding a fragment to the dictionary.

Because the check covers every file, it would fail other open pull requests once
they pick up `main`. The dictionary therefore also includes the words those pull
requests use: the git configuration keys `signingkey`, `gpgsign`, and `keyid`
(#28), and `cicd`, `unpickling`, `Safetensors`, `Unreviewed`, and `Mithril`
(#32).

### Workflow

- The workflow checks every Markdown file on every run
  (`incremental_files_only: false`), not just changed files. The repository is
  small, and a full check keeps `main` known-good when the dictionary changes.
- Actions are pinned to commit SHAs with a version comment, so Renovate (issue
  #26) can propose updates.
- The job requests only `contents: read` permission, and the checkout does not
  persist credentials.
- The job matches the markdownlint workflow: a five-minute timeout, a
  concurrency group that cancels superseded pull request runs, and a manual
  `workflow_dispatch` trigger.
- `strict: true` fails the job on any unknown word, and `inline: error` shows
  each problem as an annotation on the pull request diff.

### Local Version Pinning

The local command pins the cspell CLI version (`npx cspell@10.4.0`). The GitHub
Action bundles its own cspell release, so the two can drift slightly; the
configuration and dictionary are shared, so results should match in practice.

## Verification

- `npx cspell@10.4.0 lint --no-progress "**/*.md"` reports 0 issues across all
  Markdown files.
- A file containing `speling misteak` fails the check with exit code 1.
