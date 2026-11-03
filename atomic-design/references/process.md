# Applying Atomic Design to a product

Atomic Design is a way to build and maintain a design system consumed by products—not simply a way to sort files. Work moves in both directions: decompose real screens top-down, compose reusable pieces bottom-up, and refine abstractions using real content.

## 1. Inventory an existing interface

Before a broad redesign or refactor:

1. Find visible variants of recurring controls and sections.
2. Record examples, occurrences, and where they are used.
3. Group similar patterns and identify inconsistent values or behavior.
4. Decide what to standardize, combine, keep local, or retire.

| Pattern | Current variants / occurrences | Decision |
|---|---|---|
| Button | Several similar primary styles across screens | Standardize as a semantic variant |
| Form field | Label and error markup repeated in multiple features | Consider a shared composition |

Search the repository for repeated markup, literal styles, and existing tokens before adding abstractions. The result should be an evidence-based list of useful improvements, not a mandate to replace every difference.

## 2. Agree on names and boundaries

- Treat atoms, molecules, organisms, templates, and pages as shared vocabulary—not dogma. Follow established project terms.
- Name components by what they are (`ProductCard`), not where they appear (`HomePageCard`).
- Name variants by intent (`primary`, `danger`) rather than appearance (`blue`, `big`).
- For a disputed boundary, write a one-line rationale and continue.

## 3. Build only what the product needs

Create the foundations and components demonstrated by the inventory or current feature. A useful order is tokens → primitives → repeated compositions → interface sections → layouts → real pages, but revisit earlier choices as the design becomes clearer.

For each shared component, provide appropriate documentation or stories, meaningful tests, accessible behavior, and representative states. Keep a component feature-local until a second or third real use justifies sharing it.

## 4. Validate with templates and pages

- Use placeholder content to check template structure, spacing, and responsive layout.
- Instantiate templates with real or representative data to expose edge cases: empty and large collections, long text, missing media, errors, and loading.
- When a page needs a one-off style or workaround, decide whether the responsible token or component is missing before adding page-specific CSS.
- Keep application wiring (router, API, and global state) in the route/page layer or feature hooks; pass data and actions to UI sections.

Page variations in Storybook or fixtures can test these states without a live backend.

## 5. Migrate incrementally

Avoid a big-bang rewrite. Standardize tokens from observed values, replace repeated primitives screen by screen, then extract repeated compositions and sections. Introduce dependency checks gradually—warnings first, errors once existing violations are addressed. Keep the application deliverable at each step and measure progress against the actual scope.

## 6. Maintain the system

Agree on ownership and contribution expectations. Review real usage before adding shared components; document breaking changes and migration paths; deprecate before removal. Keep a browsable component catalog and periodically check for unused components, duplicated patterns, and token bypasses.

## Component note template

```text
# ComponentName (level: atom | molecule | organism | template | page)
Purpose: one sentence
Use / avoid when:
Main props and events:
States and accessibility:
Dependencies:
Classification rationale (if non-obvious):
```

## Review checklist

- Does an existing component already meet the need?
- Is the responsibility and composition level clear?
- Are dependencies flowing in the intended direction?
- Is this genuinely shared, or should it stay local for now?
- Are tokens, accessibility, states, and tests covered?
