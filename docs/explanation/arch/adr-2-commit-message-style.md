# ADR 2 - Commit Message Style

## History

- Status: accepted
- Deciders: Brett Smith <@xbcsmith>
- Date: 2026-10-07

## Context

In order to attain consistency in commit messages, as well as to support
automated tools to create CHANGELOGS and other reporting, we will be adopting
the style described in this document.

## Decision Drivers

- Lack of consistency in commit messages.
- Adoption of "conventional commits" style for automated tooling.
- Concise messages for `git shortlog` output, changelogs, pull request views,
  and other tooling that surfaces commit history.

## Decision Outcome

We are combining ideas from the following sources:

- Chris Beams' post about
  [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit)
- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
- [Angular guidelines](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#-commit-message-guidelines)
- Tim Pope's post about
  [commit messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)

## Style Details

```text
<type>[optional scope]: <description>
<BLANK LINE>
[optional body]
<BLANK LINE>
[optional footer(s)]
```

1. All commit messages will start with a lowercased conventional commit type,
   with optional lowercased scope. Type and scope are defined in the
   Conventional Commits specification.

2. After the type, the message must include a lowercase, short, imperative
   description. An imperative statement is “spoken or written as if giving a
   command or instruction”. e.g. "add tests for search limits". The description
   will be all lowercase with no trailing punctuation. See the referenced
   documentation for more information about imperative statements.

   Chris Beams explains that a properly formed commit description should always
   be able to complete the following sentence:

   - If applied, this commit will _your description line here_

3. Total length of first line of message should be limited to 50 characters.
   This is inclusive of the commit type, scope, description, and issue.

4. Following the first line and a blank line, a commit body may be included to
   provide extra detail. Body lines may contain uppercase and lowercase letters
   plus punctuation, but should be limited to 72 characters in length (per
   line). The body may have as many lines as needed.

   While not required, the extra information in a body is usually appreciated by
   reviewers, and can reduce questions about the code under review.

5. Following the body and a blank line, footers may be included, as defined by
   the Conventional Commits specification.

6. The commit message (description, body, and footers) should explain _what_ was
   changed and _why_. The details of _how_ should be explained by the code and
   comments.

## Notes

1. Supported commit types are listed in the
   [commits.md](../../reference/commits.md) reference documentation.

2. Chris Beams' and Tim Pope's referenced blog posts recommend capitalizing the
   first letter of the commit message. However, the Angular guidelines on which
   Conventional Commits is based, recommends not capitalizing the first letter.
   This ADR follows the Angular/Conventional Commits style, resulting in a
   lowercased commit description.

3. The 50 and 72 character limits are basic industry conventions. Tim Pope
   provides details about these limits in his blog post (see reference
   documentation). As Tim notes, the 50 and 72 character limits are not hard
   limits, but are good goals that will result in pleasing formatting from tools
   that present commit message (like git log).

## Examples

### Minimal commit message

```text
fix: fix add user crash
```

### Commit message with body

```text
feat: add agent harness integration

Integrate the agent harness tooling to provide a consistent interface
for AI agent interactions across the CDFoundation AI project.
```

### Commit message with scope, body, and footers

```text
docs(security): add AI security best practices

Add a document covering AI security best practices.

Tags: AI Security
Ref: AI Documentation
```
