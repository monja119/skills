# Atomic Design levels: classification guide

Use the levels to reason about responsibility and composition. A component's name or folder alone does not determine its level; follow the codebase convention and classify by what it owns.

## Tokens: design foundations

Tokens encode shared design decisions: semantic colors, typography, spacing, radii, shadows, motion, breakpoints, and layering.

- Name component-facing tokens by role (`--color-danger`), not raw value (`--red-500`).
- UI primitives consume tokens instead of repeating literal design values.
- Changing a token should update the system without editing every consumer.

## Atoms: context-free primitives

Atoms provide one small UI capability and have no domain knowledge. Examples: `Button`, `Input`, `Label`, `Checkbox`, `Icon`, `Avatar`, `Badge`, `Spinner`, `Heading`, and `Link`.

**Keep them:** prop-driven, accessible, and independent of API clients, global stores, routers, and other UI components (except shared low-level utilities). Extend native element props where appropriate. Expose clear variants, accept `className` consistently, and forward refs when the framework version requires it.

**Review signals:** contradictory boolean props, a large presentation-prop surface, hard-coded domain copy, effects or data fetching, or importing another UI component may indicate a molecule, a variant API, or a misplaced responsibility.

**Verify:** semantic role and accessible name, variants, disabled/focus behavior, and `className`/ref forwarding as applicable.

## Molecules: one focused unit

A molecule combines primitives into one unit with one responsibility. Examples: `FormField` (label, input, message), `SearchBar`, and a product summary card.

It receives data through props and emits events. Keep its state limited to trivial presentation behavior; validation policy, network effects, and page-specific knowledge belong higher in the composition.

**Review signals:** its purpose needs multiple unrelated verbs, it owns side effects, or it becomes a large section. A one-off composition can stay local to its organism until reuse is real.

**Verify:** accessible relationships such as label/input and error descriptions, plus help, error, and disabled states.

## Organisms: interface sections

An organism is a recognizable, relatively self-contained section, such as a `Header`, `LoginForm`, `ProductList`, `CommentThread`, or `CheckoutSummary`. It may compose atoms, molecules, and other organisms.

Organisms can coordinate UI state (form values, menu visibility, loading) and repeated components. Inject external actions through callbacks such as `onSubmit` or `onSelect`; keep API and global application wiring outside.

**Review signals:** it controls the whole page layout, imports a global service/store, or has grown so large that its sub-responsibilities deserve their own components.

**Verify:** realistic interaction flows, loading/error/empty states, and emitted callbacks.

## Templates: content-agnostic layouts

A template arranges interface sections and defines the page's content structure. It owns layout, ordering, responsive behavior, and slots (`header`, `sidebar`, `children`, `footer`), not domain data.

**Review signals:** props such as `user` or `products`, router hooks, fetching, or domain-specific copy suggest page/application responsibility.

**Verify:** slots with varied content, absent optional regions, and relevant breakpoints.

## Pages: real content and application wiring

A page instantiates a template with representative content. It is where the system is tested against real constraints: empty or large lists, long text, missing images, errors, loading, localization, and narrow screens.

In React apps, distinguish two useful roles:

| Role | Responsibility |
|---|---|
| Route page | Connects router, data, state, and navigation to the template and organisms. |
| Page variation | Story or fixture that exercises realistic content and edge cases, usually with mocked data. |

Keep route pages thin where possible. Reusable UI behavior belongs in an organism; shared layout belongs in a template.

## Common edge cases

| Component | Likely classification | Why |
|---|---|---|
| Generic styled `Card` container | Atom; molecule if it owns structured content | Classification depends on composition and responsibility. |
| `ProductCard` with image, title, price, and action | Molecule | One responsibility: present a product. |
| `ProductList` with sorting | Organism | A section that coordinates repeated items and state. |
| Modal shell and modal content | Shell: atom/primitive; content: often organism | Separate reusable mechanics from domain content. |
| Navigation menu | Molecule or organism | Use the level matching its state and orchestration. |
| Form with validation and submission | Organism | Coordinates form state and actions. |
| App-wide layout | Template | Defines structure rather than route-specific data. |
| Toast rendering and provider | UI primitive plus infrastructure | The provider is not necessarily part of the component hierarchy. |
| Sortable/paginated data table | Organism | Coordinates data presentation and interaction. |

## Promote, simplify, extract

- **Promote** when composition or orchestration grows enough to change the responsibility (for example, a molecule becomes an organism).
- **Simplify** when higher-level policy can move to a parent, leaving a focused, context-free component.
- **Extract to shared UI** after a second or third concrete use, not on the assumption of future reuse.
- **Record** a short reason when a boundary is genuinely debatable.
