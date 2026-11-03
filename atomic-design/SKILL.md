---
name: atomic-design
description: Apply Brad Frost's Atomic Design to React/TSX when creating or refactoring UI components, pages, forms, design systems, or Storybook stories; choosing component boundaries or placement; breaking down a design; or organizing a frontend component architecture. Use the five levels as a composition model, not a mandatory folder taxonomy.
license: Complete terms in LICENSE.txt
---

# Atomic Design

Atomic Design (Brad Frost) treats an interface as a system of reusable parts, composed from abstract foundations into concrete pages. Use it to improve consistency and make design decisions discussable—not to impose a fixed folder structure.

**It is a mental model, not a rigid taxonomy.** Follow the repository's established conventions. If a level is ambiguous, choose the lowest honest level, record the reason briefly, and move on.

## The five levels and their token foundation

| Level | Purpose | Typical contents | Example |
|---|---|---|---|
| **Token** | Abstract design foundation used by UI primitives. | Semantic color, type, spacing, motion, and similar values | `--color-danger` |
| **Atom** | Small, context-independent UI primitive. | Tokens and native elements | `Button`, `Input`, `Label` |
| **Molecule** | Small group of primitives with one responsibility. | Atoms | `FormField`, `SearchBar` |
| **Organism** | Distinct, self-contained section of an interface. | Atoms, molecules, or other organisms | `Header`, `LoginForm` |
| **Template** | Content-agnostic page structure and layout. | Slots and interface sections | `AuthTemplate` |
| **Page** | A template instantiated with representative, real content. | Template and data | `LoginPage` |

For classification criteria, edge cases, and promotion guidance, read [`references/levels.md`](references/levels.md).

## Composition rules

- Dependencies flow downward: `Page → Template → Organism → Molecule → Atom → Token`. Organisms may compose other organisms; avoid upward imports.
- Keep low-level components context-free: no API, router, or global-store access in atoms or molecules. Pass actions down as props; send events upward.
- Let parents control placement. Components own internal layout, not external margins or page positioning.
- Prefer children and slots over layers of boolean configuration props.
- A template owns layout and slots, not fetching, application state, or domain copy. A page/route connects application data and actions.
- Extract shared UI after a second or third real use; keep one-off pieces local to their feature.

## Workflow

1. **Inspect before adding.** Find existing components, tokens, styles, and local conventions; extend or reuse where appropriate.
2. **Decompose the need.** Sketch the page-to-token composition when scope is broad; share the sketch with the user when it clarifies decisions.
3. **Classify by responsibility.** Use the decision path below; do not force every level into a small project.
4. **Build and verify incrementally.** Start with needed foundations and primitives, then compose upward. Test each piece in its actual context.
5. **Exercise real content.** Cover empty, loading, error, long-content, and responsive states where relevant; feed discoveries back into the responsible component.
6. **Check boundaries and accessibility.** Apply the checklist before handing off.

When the implementation is React/TSX and needs concrete patterns, read [`references/react-tsx.md`](references/react-tsx.md). For an existing-system inventory, incremental migration, or governance, read [`references/process.md`](references/process.md).

For the route-page versus feature-hook decision, and where to test that logic, read the application-wiring guidance in [`references/process.md`](references/process.md).

## Placement decision

1. Is it a design value? → **Token**.
2. Is it a context-free primitive with one simple role? → **Atom**.
3. Does it combine primitives into one focused unit? → **Molecule**.
4. Is it a recognizable interface section with orchestration or local UI state? → **Organism**.
5. Does it define layout and content slots without choosing the content? → **Template**.
6. Does it connect a template to concrete application data or represent a real content variation? → **Page**.

## Example decomposition

```text
LoginPage (route, data, navigation)
└─ AuthTemplate (header / content / footer slots)
   ├─ Header (existing organism)
   └─ LoginForm (organism; local UI state; injected onSubmit)
      ├─ FormField ×2 (molecule: Label + Input + message)
      └─ Button (atom)
```

## Completion checklist

- [ ] Dependencies follow the intended direction; no API, router, or store access has leaked into low-level UI.
- [ ] Design values use the project's tokens; shared abstractions have real reuse.
- [ ] Components preserve semantic HTML, keyboard access, visible focus, and accessible labels/states.
- [ ] Stories or tests cover relevant variants and meaningful edge cases.
- [ ] Ambiguous classification or repository-specific exceptions are documented briefly.
