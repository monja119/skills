# Atomic Design in React and TypeScript

This compact login example shows responsibility boundaries, not a required directory layout. It assumes modern React and TypeScript; adapt imports, ref handling, and Storybook APIs to the versions already installed.

## Tokens and atom

Store shared design values in the project's existing theme mechanism. Components consume semantic tokens rather than defining their own palette.

```css
:root {
  --color-surface: #fff;
  --color-text: #1a1a1a;
  --color-brand: #2f5bea;
  --color-danger: #c62828;
  --space-control: 0.5rem;
  --radius-control: 0.5rem;
}
```

An atom wraps a native element without taking on domain behavior:

```tsx
import type { ComponentProps } from "react";

type ButtonProps = ComponentProps<"button"> & {
  variant?: "primary" | "secondary";
};

export function Button({
  variant = "primary",
  type = "button",
  className,
  ...props
}: ButtonProps) {
  return (
    <button
      type={type}
      data-variant={variant}
      className={["button", className].filter(Boolean).join(" ")}
      {...props}
    />
  );
}
```

In a real component, merge caller `className` with internal styles, expose only coherent variants, and keep focus and disabled states accessible. React 18 components that need refs should use `forwardRef`; use the repository's React-version conventions.

## Molecule: accessible `FormField`

Compose primitives into one unit. The molecule presents a field; its parent owns validation policy.

```tsx
import { useId, type ComponentProps } from "react";

type FormFieldProps = Omit<ComponentProps<"input">, "id"> & {
  label: string;
  hint?: string;
  error?: string;
};

export function FormField({ label, hint, error, ...inputProps }: FormFieldProps) {
  const id = useId();
  const message = error ?? hint;
  const messageId = `${id}-message`;

  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      <input
        {...inputProps}
        id={id}
        aria-invalid={error ? true : undefined}
        aria-describedby={message ? messageId : undefined}
      />
      {message && (
        <p id={messageId} role={error ? "alert" : undefined}>
          {message}
        </p>
      )}
    </div>
  );
}
```

## Organism: `LoginForm`

An organism may own form UI state and loading behavior. Inject the operation instead of importing an API client or router.

```tsx
import { useState, type FormEvent } from "react";

type LoginValues = { email: string; password: string };
type LoginFormProps = {
  onSubmit: (values: LoginValues) => Promise<void>;
  error?: string;
};

export function LoginForm({ onSubmit, error }: LoginFormProps) {
  const [values, setValues] = useState<LoginValues>({ email: "", password: "" });
  const [loading, setLoading] = useState(false);

  async function handleSubmit(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();
    setLoading(true);
    try {
      await onSubmit(values);
    } finally {
      setLoading(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <FormField
        label="Email"
        type="email"
        autoComplete="email"
        required
        value={values.email}
        onChange={(event) =>
          setValues((current) => ({ ...current, email: event.target.value }))
        }
      />
      <FormField
        label="Password"
        type="password"
        autoComplete="current-password"
        required
        value={values.password}
        onChange={(event) =>
          setValues((current) => ({ ...current, password: event.target.value }))
        }
        error={error}
      />
      <Button type="submit" disabled={loading}>
        {loading ? "Signing in…" : "Sign in"}
      </Button>
    </form>
  );
}
```

In production, decide where rejected submissions map to visible errors using the application-wiring rule in [`process.md`](process.md). Keep the policy outside the organism.

## Template and page

The template owns layout slots, not route data or domain copy:

```tsx
import type { ReactNode } from "react";

export function AuthTemplate({
  header,
  children,
  footer,
}: {
  header?: ReactNode;
  children: ReactNode;
  footer?: ReactNode;
}) {
  return (
    <div className="auth-layout">
      {header && <header>{header}</header>}
      <main>{children}</main>
      {footer && <footer>{footer}</footer>}
    </div>
  );
}
```

The route page connects application services and passes data/actions down:

```tsx
function LoginPage() {
  const navigate = useNavigate();
  const [error, setError] = useState<string>();

  async function handleSubmit(values: LoginValues) {
    try {
      await login(values);
      navigate("/");
    } catch {
      setError("We could not sign you in. Check your details and try again.");
    }
  }

  return (
    <AuthTemplate>
      <h1>Sign in</h1>
      <LoginForm onSubmit={handleSubmit} error={error} />
    </AuthTemplate>
  );
}
```

This example keeps page-specific mapping in the route. Extract it to a feature hook only when the flow is shared or substantial; use the test strategy in [`process.md`](process.md).

## Stories and tests

Create stories for meaningful states rather than every theoretical prop combination: default, error, loading, empty, long content, and narrow viewport as applicable. Page variations should use fixtures or mocked services, not require a live backend.

Test by responsibility:

| Level | Useful checks |
|---|---|
| Atom | Native semantics, variants, disabled/focus states, accessible name |
| Molecule | Label/input association, hint/error announcement |
| Organism | User interaction, emitted callback values, loading/error behavior |
| Template | Slot rendering, semantic structure, responsive layout |
| Page | Route-owned outcome mapping and application integration; if a feature hook owns the flow, test it through that hook and retain one route wiring check |

Example organism assertion:

```tsx
it("submits the entered credentials", async () => {
  const onSubmit = vi.fn().mockResolvedValue(undefined);
  render(<LoginForm onSubmit={onSubmit} />);
  await userEvent.type(screen.getByLabelText("Email"), "person@example.com");
  await userEvent.type(screen.getByLabelText("Password"), "secret");
  await userEvent.click(screen.getByRole("button", { name: "Sign in" }));
  expect(onSubmit).toHaveBeenCalledWith({
    email: "person@example.com",
    password: "secret",
  });
});
```

## Enforce boundaries where useful

Use the repository's existing lint or architecture tooling to prevent upward imports. A reasonable policy allows molecules to import atoms, organisms to compose atoms/molecules/organisms, templates to compose UI sections, and pages to import templates and lower levels. Also keep router, API-client, and global-store imports out of low-level UI. Verify plugin configuration against the installed version before adopting a new rule.

## Conventions

- Match the existing feature/component folder organization; Atomic Design does not require five global directories.
- Prefer named exports and one primary component per file when that matches the codebase.
- Use slots/children for composition; keep external spacing and placement with the parent.
- Keep accessibility in the implementation, not as a later checklist-only task.
