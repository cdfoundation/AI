# Documentation Layout

CDFoundation AI documentation follows the
[Diataxis Framework](https://diataxis.fr) wherever it makes sense.

## Directory Structure

| Directory           | Type                   | Purpose                                        |
| ------------------- | ---------------------- | ---------------------------------------------- |
| `docs/tutorials/`   | Learning-oriented      | Step-by-step lessons for new users             |
| `docs/how-to/`      | Problem-oriented       | Recipes for solving specific problems          |
| `docs/explanation/` | Understanding-oriented | Architecture, decisions, research, and context |
| `docs/reference/`   | Information-oriented   | Technical specifications, APIs, and glossaries |

## Sections

### Tutorials - Learning-Oriented

Tutorials get users started. They must be repeatable and work every time. A good
tutorial inspires confidence as the reader learns the system.

- Focus on practical steps only. Do not explain why things work; explanation
  distracts the reader and belongs in the Explanation section.
- Make tutorials bulletproof. If a step can fail, add a check or a note.
- Assume the reader is new to the subject.

### How-to Guides - Problem-Oriented

How-to guides take the reader through a series of steps to solve a specific
problem. Think of them as recipes.

- The reader already understands the system well enough to ask "How do I do X?".
  The guide answers that question directly.
- Do not explain why something works. Explanation gets in the way of action.
- Focus on the problem and the solution, not on background context.

### Reference - Information-Oriented

Reference documentation describes the technical machinery: classes, functions,
APIs, configuration options, and glossary entries.

- The only job of reference material is accurate, consistent description.
- Do not explain how to solve a problem. Think of an encyclopedia entry about a
  recipe rather than the recipe itself.
- Consistency and structure matter. Reference material should read like a
  dictionary: predictable, scannable, and complete.

### Explanation - Understanding-Oriented

Explanations discuss background, context, and rationale. They help the reader
make sense of the subject.

- Cover architecture decisions, research, trade-offs, and the reasoning behind
  choices.
- Do not give instructions or technical descriptions. Those belong in How-to or
  Reference sections.
- Architecture Decision Records (ADRs) belong here under
  `docs/explanation/arch/`.

## Quadrant Map

|                                  | Documentation Layout  |                                 |
| -------------------------------- | --------------------- | ------------------------------- |
|                                  | Practical Steps       |                                 |
| Tutorials                        |                       | How-to Guides                   |
| Learning-Oriented                |                       | Problem-Oriented                |
|                                  |                       |                                 |
| Most useful when we are studying |                       | Most useful when we are working |
|                                  |                       |                                 |
| Understanding-Oriented           |                       | Information-Oriented            |
| Explanation                      |                       | Reference                       |
|                                  | Theoretical Knowledge |                                 |
