# ADR 3 - Documentation Style

## History

- Status: accepted
- Deciders: Brett Smith <@xbcsmith>
- Date: 2026-10-07

## Context and Problem Statement

As the project grows, AI agents and human contributors produce documentation
inconsistently. Without a shared framework, docs end up in arbitrary locations,
mix instructional and conceptual content, and become hard to maintain or
discover. How should documentation be structured so that every contributor -
human or AI - knows where to put new content and what form it should take?

## Decision Drivers

- Documentation must be navigable by both humans and AI agents without ambiguity
  about where content belongs
- AI agents require deterministic rules to avoid placing implementation
  summaries, tutorials, and reference material in the same location
- Inconsistent file naming and placement breaks cross-links and CI/CD tooling
- Contributors at different experience levels need different types of content
  (learning vs. task completion vs. deep understanding vs. quick lookup)
- The project requires a documentation standard that can be codified into agent
  rules and enforced by linting tooling

## Considered Options

- Ad-hoc documentation with no enforced structure
- Diataxis framework with four defined content types
- Docs-as-Code with a single flat directory and tagging

## Decision Outcome

Chosen option: "Diataxis framework with four defined content types", because it
provides a clear, principled separation of concerns that maps directly to
contributor intent. Each of the four quadrants answers a distinct question,
making placement decisions unambiguous for both humans and AI agents. The
structure can be enforced through directory conventions and codified in
`AGENTS.md`.

### Directory Mapping

| Directory           | Diataxis Type          | Purpose                                        |
| ------------------- | ---------------------- | ---------------------------------------------- |
| `docs/tutorials/`   | Learning-oriented      | Step-by-step lessons for new users             |
| `docs/how-to/`      | Problem-oriented       | Recipes for solving specific problems          |
| `docs/explanation/` | Understanding-oriented | Architecture, decisions, research, and context |
| `docs/reference/`   | Information-oriented   | Technical specifications, APIs, and glossaries |

Implementation summaries produced by AI agents are placed in
`docs/explanation/`.

### Agent Rules Derived From This Decision

The following rules in `AGENTS.md` are direct consequences of adopting Diataxis:

**Rule 2 - Markdown File Naming**: All documentation files use
`lowercase_with_underscores.md`. `README.md` is the only exception. Consistent
naming prevents broken cross-links between Diataxis sections.

**Rule 3 - No Emojis**: No emojis in code, comments, or documentation. Emojis
cause encoding issues and break tooling that processes documentation at build
time.

**Rule 4 - Quality Gates**: Every documentation file must pass the following
before being considered complete:

```bash
markdownlint --fix --config .markdownlint.json "${FILE}"
prettier --write --parser markdown --prose-wrap always "${FILE}"
```

### Positive Consequences

- Placement of any document is unambiguous; contributors ask "what is the reader
  trying to do?" rather than "where should this go?"
- AI agents can be given deterministic rules derived directly from the framework
- Tutorials stay instructional; reference material stays descriptive;
  explanations stay conceptual - no content bleed between types
- Linting and naming conventions can be automated and enforced in CI

### Negative Consequences

- Contributors must learn the Diataxis quadrant model before writing docs
- Some content genuinely spans multiple types (e.g., a tutorial that also serves
  as reference), requiring a judgment call on primary intent
- Strict separation can feel over-engineered for small one-off documents

## Pros and Cons of the Options

### Ad-hoc documentation with no enforced structure

- Good, because it has zero onboarding cost for new contributors
- Good, because no tooling or CI enforcement is required
- Bad, because documents accumulate without a home and become hard to find
- Bad, because AI agents have no deterministic rule for where to place content,
  leading to inconsistent output
- Bad, because mixed content types (tutorial + reference + explanation in one
  file) erode documentation quality over time

### Diataxis framework with four defined content types

- Good, because placement decisions follow a clear, principled model
- Good, because each quadrant serves a distinct reader need
- Good, because the directory structure is machine-readable and enforceable
- Good, because it scales well as the project grows
- Bad, because contributors must internalize the model before contributing
- Bad, because content that spans quadrants requires a judgment call

### Docs-as-Code with a single flat directory and tagging

- Good, because it is simple to implement initially
- Good, because tagging allows flexible categorization without directory
  constraints
- Bad, because tags are not enforced by directory structure and drift over time
- Bad, because flat directories do not scale and become hard to navigate
- Bad, because AI agents cannot reliably infer placement from tags alone

## Links

- [Diataxis Framework](https://diataxis.fr)
- [Documentation Layout Reference](../../reference/documentation_layout.md)
- Implemented by [AGENTS.md](../../../AGENTS.md)
