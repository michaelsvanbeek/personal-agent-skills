---
name: typescript
description: 'TypeScript coding standards and type safety conventions. Use when: creating TypeScript files, defining interfaces and types, writing type-safe code, reviewing TypeScript for type correctness, auditing a codebase for type safety gaps, eliminating any or ts-ignore usage, or improving strict-mode compliance. Covers strict typing, avoiding any and ts-ignore, discriminated unions, Zod runtime validation, immutability patterns, and proper type definitions.'
tags:
  - developer
---

# TypeScript Standards

## When to Use

- Creating or modifying TypeScript files
- Defining types, interfaces, or type utilities
- Reviewing TypeScript code for type safety

## Type Safety

- Create valid, well-defined types for all data structures.
- **Never use `@ts-ignore`** - fix the underlying type issue instead.
- **Never use `any` type** - use `unknown` if the type is truly unknown, then narrow with type guards.
- Prefer `interface` for object shapes that may be extended; use `type` for unions, intersections, and computed types.

## Type Definitions

```typescript
// Prefer interfaces for object shapes
interface User {
  id: string;
  name: string;
  email: string;
  role: UserRole;
}

// Use type for unions and computed types
type UserRole = "admin" | "editor" | "viewer";
type UserMap = Record<string, User>;

// Use generics for reusable patterns
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}
```

## Discriminated Unions

Use discriminated unions for any type that can be one of several variants. Add a literal `type` field as the discriminator:

```typescript
// Define variants with a shared discriminator field
interface LoadingState {
  type: "loading";
}

interface SuccessState<T> {
  type: "success";
  data: T;
}

interface ErrorState {
  type: "error";
  error: string;
  retryable: boolean;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// TypeScript narrows automatically in switch/if blocks
function render(state: AsyncState<User[]>) {
  switch (state.type) {
    case "loading":
      return <Spinner />;
    case "success":
      return <UserList users={state.data} />; // data is typed as User[]
    case "error":
      return <ErrorMessage message={state.error} />;
  }
}
```

### When to Use Discriminated Unions

| Use case | Example |
|----------|---------|
| **State machines** | `idle → loading → success \| error` |
| **Command/event types** | `{ type: "create" } \| { type: "update" } \| { type: "delete" }` |
| **API responses** | Different shapes per endpoint or error type |
| **Configuration variants** | `{ transport: "stdio" } \| { transport: "http", url: string }` |

### Rules

- The discriminator field must be a **string literal type**, not a broad `string`.
- Every variant must include the discriminator field.
- Use `switch` with exhaustive checks — add a `default: never` assertion to catch unhandled variants:

```typescript
function assertNever(x: never): never {
  throw new Error(`Unexpected variant: ${JSON.stringify(x)}`);
}

function handleCommand(cmd: Command) {
  switch (cmd.type) {
    case "create": return handleCreate(cmd);
    case "update": return handleUpdate(cmd);
    case "delete": return handleDelete(cmd);
    default: return assertNever(cmd); // compile error if a variant is missing
  }
}
```

## Runtime Validation with Zod

Use **Zod** to validate data at system boundaries — API responses, config files, user input, environment variables. Zod bridges the gap between TypeScript's compile-time types and runtime reality.

```typescript
import { z } from "zod";

// Define schema — this is the single source of truth
const UserSchema = z.object({
  id: z.string(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(["admin", "editor", "viewer"]),
});

// Derive the TypeScript type from the schema
type User = z.infer<typeof UserSchema>;

// Validate at runtime
function parseUser(data: unknown): User {
  return UserSchema.parse(data); // throws ZodError on invalid input
}

// Safe parse (returns result instead of throwing)
const result = UserSchema.safeParse(data);
if (result.success) {
  console.warn(result.data.name); // typed as User
} else {
  console.error(result.error.issues);
}
```

### When to Use Zod

| Boundary | Example |
|----------|---------|
| **External API responses** | Parse JSON before trusting it |
| **Configuration files** | Validate settings, frontmatter, manifests |
| **Environment variables** | Validate and type env vars at startup |
| **User input** | Form data, URL params, CLI arguments |
| **Webhook payloads** | Verify structure before processing |

### Zod Patterns

```typescript
// Discriminated union schemas
const EventSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("click"), x: z.number(), y: z.number() }),
  z.object({ type: z.literal("keypress"), key: z.string() }),
]);

// Coercion for env vars (strings → typed values)
const EnvSchema = z.object({
  PORT: z.coerce.number().default(3000),
  DEBUG: z.coerce.boolean().default(false),
  DATABASE_URL: z.string().url(),
});

// Reusable schema composition
const PaginationSchema = z.object({
  limit: z.number().int().min(1).max(100).default(25),
  cursor: z.string().optional(),
});
```

### Rules

- **Schema is the source of truth** — derive TypeScript types with `z.infer<>`, don't duplicate.
- **Validate at boundaries, trust internally** — once data passes Zod, pass the typed result through your code without re-validating.
- **Use `safeParse` for user-facing errors** — returns structured error details. Use `parse` for internal assertions where failure is a bug.

## Immutability Patterns

Enforce immutability at the type level to prevent accidental mutations in shared state:

```typescript
// Deep immutable wrapper — prevents mutations at any depth
type DeepImmutable<T> = T extends Map<infer K, infer V>
  ? ReadonlyMap<DeepImmutable<K>, DeepImmutable<V>>
  : T extends Set<infer S>
    ? ReadonlySet<DeepImmutable<S>>
    : T extends object
      ? { readonly [K in keyof T]: DeepImmutable<T[K]> }
      : T;

// Use for state that must not be mutated
interface AppState {
  readonly settings: DeepImmutable<Settings>;
  readonly permissions: DeepImmutable<PermissionRules>;
}
```

### Rules

- Use `readonly` on interface properties that should not be mutated after creation.
- Use `ReadonlyArray<T>` or `readonly T[]` for arrays that should not be modified.
- Use `Readonly<T>` for shallow immutability, `DeepImmutable<T>` for state shared across boundaries.
- Update immutable state with spread: `{ ...prev, field: newValue }`, never with assignment.

## Abort Signal / Cancellation

Propagate `AbortSignal` through async operations for proper cancellation:

```typescript
async function fetchData(url: string, signal: AbortSignal): Promise<Data> {
  const response = await fetch(url, { signal });
  return response.json() as Promise<Data>;
}

// Type guard for abort errors (multiple sources)
function isAbortError(error: unknown): boolean {
  return (
    error instanceof DOMException && error.name === "AbortError" ||
    error instanceof Error && error.name === "AbortError"
  );
}

// Usage with cleanup
const controller = new AbortController();
try {
  const data = await fetchData("/api/users", controller.signal);
} catch (error) {
  if (isAbortError(error)) return; // expected cancellation, not an error
  throw error;
}
```

- Pass `AbortSignal` to all async functions that support cancellation.
- Check `signal.aborted` before expensive operations in long-running loops.
- Distinguish abort errors from real failures — don't log aborts as errors.

## Best Practices

- Enable `strict` mode in `tsconfig.json`.
- Prefer `readonly` for properties that should not be mutated.
- Use `as const` for literal type inference on constants.
- Leverage type narrowing with `typeof`, `instanceof`, and custom type guards over type assertions.
- Use `satisfies` operator to validate types while preserving inference:

```typescript
// satisfies checks the type without widening — preserves literal types
const ROUTES = {
  home: "/",
  about: "/about",
  settings: "/settings",
} as const satisfies Record<string, string>;
// ROUTES.home is typed as "/" (literal), not string
```

## tsconfig.json

Enable strict mode and recommended compiler options:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

## Module Organization

- One primary export per file. Co-locate related types in the same file.
- Use barrel exports (`index.ts`) sparingly and only at module boundaries.
- Prefer named exports over default exports for better refactoring support.

For React-specific patterns with TypeScript, TailwindCSS, and Vite, see the **web-ui skill**. For runtime validation at API boundaries using Zod, see the **api-design skill**.
