# Spellcheck Implementation

<!-- cspell:ignore defence organised Communi speling misteak -->

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

The project standardizes on US English (`en-US`). Two British spellings in the
existing docs were changed: `defence` to `defense` in
`docs/explanation/agents/agent_harness.md`, and `organised` to `organized` in
`docs/README.md`.

### Baseline Dictionary

Running cspell against `main` flagged a set of words, none of them real typos:
member names and GitHub handles, AI and MLOps terminology, and proper nouns.
These seed `project-words.txt`. The one exception was `Communi-`, a word split
by a hyphen in an ASCII diagram in
`docs/explanation/agents/trust_and_security.md`. The diagram was reflowed to
show the whole word rather than adding a fragment to the dictionary.

### Workflow

- The workflow checks every Markdown file on every run
  (`incremental_files_only: false`), not just changed files. The repository is
  small, and a full check keeps `main` known-good when the dictionary changes.
- Actions are pinned to commit SHAs with a version comment, so Renovate (issue
  #26) can propose updates.
- The job requests only `contents: read` permission.
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
