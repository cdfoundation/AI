# Run the Spellcheck

This repository uses [cspell](https://cspell.org/) to check spelling in all
Markdown files. The same check runs in CI on every pull request and on every
push to `main` (see `.github/workflows/spellcheck.yaml`).

## Check Spelling Locally

Requires Node.js. From the repository root, run:

```bash
npx cspell@10.4.0 lint --no-progress "**/*.md"
```

To check a single file:

```bash
npx cspell@10.4.0 lint --no-progress "docs/path/to/file.md"
```

The command exits non-zero and lists each unknown word with its file, line, and
column when it finds a problem.

## Fix a Reported Word

1. If the word is a typo, correct it in the document.
2. If the word uses British spelling, change it to US English. The project
   standardizes on `en-US`.
3. If the word is correct (a name, GitHub handle, product, acronym, or technical
   term), add it to the project dictionary as described below.

## Add a Word to the Project Dictionary

1. Open `project-words.txt` at the repository root.
2. Add the word on its own line, keeping the list sorted alphabetically
   (case-insensitive).
3. Run the local check again to confirm it passes.

Matching is case-insensitive, so add each word once. Only add words that are
spelled correctly; do not add misspellings to make the check pass.

## Ignore a Word in One File Only

For a word that is valid in one place but should not be accepted everywhere, add
an inline directive to the Markdown file instead of the dictionary:

```markdown
<!-- cspell:ignore someword -->
```

## Configuration

- `cspell.json` sets the language (`en-US`), registers the project dictionary,
  and lists ignored paths.
- `project-words.txt` holds the project dictionary.
