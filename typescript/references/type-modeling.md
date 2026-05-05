# Type Modeling

Detailed TypeScript type-modeling rules for strict mode, type aliases, parametric APIs, and variants.

## Strict Mode

Always enable strict mode in `tsconfig.json`. Never use `// @ts-ignore` or `// @ts-nocheck`.

## Types over Interfaces

Prefer `type` over `interface` unless you specifically need declaration merging or class implementation:

```typescript
type UserProfile = {
  id: string
  name: string
  email: string
}
```

## Parametric APIs over Subtype APIs

Prefer generics plus explicit behavior parameters over subclass trees that override one method:

```typescript
type Comparator<T> = (left: T, right: T) => number

const sortBy = <T>(items: readonly T[], compare: Comparator<T>): T[] =>
  [...items].sort(compare)
```

Reach for classes when you need lifecycle, identity, or framework integration. Do not introduce subclasses just to swap parsing, sorting, formatting, or validation logic.

## Explicit Return Types

Always add return types on exported functions. Inference is fine for small internal helpers.

## Discriminated Unions over Flag Combinations

For naming the union, its variants, and any narrowing type guards, see `/naming` — the union owns the concept name, variants take qualifiers, and guards are named after what they narrow to.

Model variants as tagged unions and exhaustively handle them:

```typescript
type SyncResult =
  | { kind: 'success'; updatedCount: number }
  | { kind: 'conflict'; conflicts: string[] }
  | { kind: 'failure'; message: string }

function renderResult(result: SyncResult): string {
  switch (result.kind) {
    case 'success':
      return `${result.updatedCount} records updated`
    case 'conflict':
      return `${result.conflicts.length} conflicts`
    case 'failure':
      return result.message
    default: {
      const _exhaustive: never = result
      return _exhaustive
    }
  }
}
```

When a closed set of values drives branching, persistence, or error reporting, define it once as a schema-backed domain value and reuse it everywhere:

```typescript
export const PersistenceWarningScopeSchema = z.enum(['job', 'result'])
export const PERSISTENCE_WARNING_SCOPE = PersistenceWarningScopeSchema.enum
export type PersistenceWarningScope = z.infer<typeof PersistenceWarningScopeSchema>
```

This prevents review churn from ad hoc string literals drifting across handlers, schemas, and telemetry tags.

Keep the three roles visually distinct:
- `FooSchema` for the parser/validator
- `type Foo` for the inferred TypeScript type
- `FOO` for the runtime enum-like value object produced from `.enum`
