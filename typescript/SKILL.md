---
name: typescript
description: TypeScript and JavaScript coding conventions. Apply when writing, reviewing, or refactoring any .ts, .tsx, .js, or .jsx code. Covers strict types, Zod validation, casing, imports, and type safety rules. For symbol naming principles (predicates, variants, domain constants), see /naming. For filesystem layout, see /file-naming.
---

# TypeScript Conventions

Apply these when writing, reviewing, or refactoring TypeScript/JavaScript code.

For deeper rationale and examples behind the Jane Street/OCaml-inspired rules, see `references/functional-design.md`.

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

For cross-cutting symbol naming (predicates as questions, narrowing-predicate target naming, union-vs-variant naming, generic-verb smells, closed-set values as domain constants), see `/naming`. The casing table below is the TypeScript-specific layer on top of those principles.

## Strict Mode

Always enable strict mode in `tsconfig.json`. Never use `// @ts-ignore` or `// @ts-nocheck`.

## Casing

| Element | Convention | Example |
|---------|-----------|---------|
| Functions, variables, parameters | `camelCase` | `getUserProfile` |
| Types, classes, enums | `PascalCase` | `UserProfile` |
| Private class members | Leading underscore | `_internalState` |
| Constants (true constants and schema-derived enum objects) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `PERSISTENCE_WARNING_SCOPE` |
| Files | `kebab-case` or match export name | `user-profile.ts` |

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

## Import Ordering

Group imports with blank lines between groups, alphabetized within each:

```typescript
// 1. Builtin / external
import { readFile } from 'node:fs/promises'
import { z } from 'zod'

// 2. Parent directories
import { AppConfig } from '../config'

// 3. Sibling / index
import { helpers } from './helpers'
```

Use `import type` for type-only imports:

```typescript
import type React from 'react'
import type { FC, ReactNode } from 'react'
```

## Zod Validation — All External Data

All external API responses, user inputs, and deserialized data MUST be validated with Zod:

```typescript
const UserSchema = z.object({
  id: z.string(),
  name: z.string().trim().min(1).max(200),
  email: z.string().email().max(320),
})

type User = z.infer<typeof UserSchema>

const user = UserSchema.parse(data)
```

Use `.parse()` when invalid data should fail fast. Use `.safeParse()` when you need a fallback path.

For boundary-owned shapes, let the schema name reflect the domain behavior instead of the transport history. Prefer `InitMessageRequestBodySchema` over a stale name like `BootstrapMessageRequestBodySchema` once the meaning is clearly "init payload".

## No `as` Type Assertions

`as` bypasses type checking. Use Zod or type guards instead:

```typescript
// BAD
const user = response.data as User

// GOOD
const user = UserSchema.parse(response.data)
```

## Named Option Objects for Ambiguous Calls

Once a function takes multiple booleans or repeated scalar types, prefer an option object over positional arguments:

```typescript
type RetryOptions = {
  maxAttempts: number
  backoffMs: number
  jitter: boolean
}

function fetchWithRetry(url: string, options: RetryOptions): Promise<Response> {
  // ...
}
```

This is the TypeScript analogue of labeled arguments: it makes call sites readable without memorizing argument order.

## Narrow Exports and Opaque Boundaries

- Export the smallest public surface that matches the module's responsibility
- Prefer factory/parser functions at boundaries instead of exposing raw unchecked shapes
- Use branded types when two values are both `string` or `number` but mean different things (`UserId` vs `SessionId`)
- Prefer `readonly` arrays and object properties by default unless mutation is required for performance or framework constraints
- Keep bootstrap-only constants separate from runtime-loaded config. Code that loads config must not depend on the config value it is still trying to read
- Give each important boundary shape one clear owning module. Callers should import the contract from that owner instead of recreating local variants
- When a contract changes, prefer a deliberate migration path over temporary duplicate shapes with nearly identical names
- If you need to rename a boundary type or schema, rename the value, type, parser, and file together so transport-history names do not linger beside domain-accurate names
- If you split a boundary into helper files such as `types.ts`, `result.ts`, and `index.ts`, update the barrel and all internal imports in the same change. Helper modules should not accidentally become the new public owner for contract types
- If consumers import a boundary barrel and still need sibling deep imports from the same folder, either promote those symbols into the barrel or stop treating that barrel as the public boundary
- If a file primarily exports schemas, inferred types, and enum-like constants for one concept folder, prefer a role name like `types.ts` or `schema.ts`. Do not use duplicated names like `user/user.ts` or `signal/signal.ts` unless the file also owns the main behavior for that concept

## Alias and Import Refactors

- Before switching code to a path alias, verify `tsconfig` exposes the form callers need. If consumers import both `@foo` and `@foo/bar`, the config usually needs both the bare alias and the wildcard alias
- After moving or splitting contracts, run `rg` on old import paths and a full `tsc` pass before considering the refactor complete
- Import cleanup must preserve ownership. Never pull a symbol from a different layer just because another import from that layer already exists

## No `unknown` (Except Catch Blocks)

- Allow `unknown` in catch blocks, then narrow it immediately
- Do not use `unknown` as a general input model for application code; validate boundary data with Zod and work with typed values afterward

## Error Handling

- Use custom error classes at meaningful boundaries
- Handle specific error types first
- Log failures with context, then rethrow or map to a user-safe error

## Contract Review Checklist

- Does this boundary shape have one obvious owner?
- Are branching values modeled once as shared domain values rather than repeated strings?
- Will the next caller import the same schema/type, or are we encouraging local copies?
- If this contract changes next month, is there one migration point or many?

## Nullish Operators

- `??` — only catches `null` and `undefined` (NOT empty strings)
- `||` — catches all falsy values including `''`, `0`, `false`

Use `||` when empty strings should fall back. Use `??` when `0` or `false` are valid values.
