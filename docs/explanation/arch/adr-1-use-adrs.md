# ADR 1 - Documenting Architecture Decisions

## History

- Status: accepted
- Deciders: Brett Smith <@xbcsmith>
- Date: 2026-10-07

## Context and Problem Statement

The following text is adapted from a
[Michael Nygard blog post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
on architecture decision records. It is reproduced here so the rationale is
available directly in the repository without requiring an external reference.

Architecture for agile projects has to be described and defined differently. Not
all decisions are made at once, nor are all of them complete when the project
begins.

Agile methods are not opposed to documentation, only to valueless documentation.
Documents that assist the team itself can have value, but only if they are kept
up to date. Large documents are never kept up to date. Small, modular documents
have at least a chance at being updated.

Nobody ever reads large documents, either. Most developers have been on at least
one project where the specification document was larger (in bytes) than the
total source code size. Those documents are too large to open, read, or update.
Bite-sized pieces are easier for all stakeholders to consume.

One of the hardest things to track during the life of a project is the
motivation behind certain decisions. A new person coming on to a project may be
perplexed, baffled, or infuriated by some past decision. Without understanding
the rationale or consequences, this person has only two choices:

1. Blindly accept the decision.

   This response may be acceptable if the decision is still valid. It may not be
   good, however, if the context has changed and the decision should really be
   revisited. If the project accumulates too many decisions accepted without
   understanding, then the development team becomes afraid to change anything
   and the project collapses under its own weight.

2. Blindly change it.

   Again, this may be acceptable if the decision needs to be reversed. On the
   other hand, changing the decision without understanding its motivation or
   consequences could mean damaging the project's overall value without
   realizing it (e.g., the decision supported a non-functional requirement that
   has not been tested yet).

It is better to avoid either blind acceptance or blind reversal.

## Decision Outcome

We will keep a collection of records for architecturally significant decisions:
those that affect the structure, non-functional characteristics, dependencies,
interfaces, or construction techniques of the CDFoundation AI project.

An Architecture Decision Record (ADR) is a short Markdown file describing a set
of forces and a single decision made in response to those forces. The decision
is the central piece; specific forces may appear in multiple ADRs.

ADRs are stored in the project repository under
`docs/explanation/arch/adr-NNN.md`. A template can be found in
`docs/explanation/arch/draft/adr-template.md`.

ADRs are numbered sequentially and monotonically. Numbers are not reused.

If a decision is reversed, the old ADR is kept but marked as superseded. It is
still relevant to know that the decision was made, even if it is no longer
current.

## Consequences

One ADR describes one significant decision for the project. The decision should
be something that has an effect on how the rest of the project is built or
operated.

The consequences of one ADR are very likely to become the context for subsequent
ADRs.

Developers and project stakeholders can see the ADRs, even as the team
composition changes over time. The motivation behind previous decisions is
visible for everyone, present and future. Nobody is left wondering "What were
they thinking?" and the right time to revisit old decisions will be clear from
changes in the project's context.

## Links

- <https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions>
- <https://github.com/joelparkerhenderson/architecture-decision-record>
- <https://adr.github.io/>
