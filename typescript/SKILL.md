---
name: typescript
description: TypeScript and JavaScript coding conventions. Apply when writing, reviewing, or refactoring any .ts, .tsx, .js, or .jsx code. Covers strict types, Zod validation, naming, imports, and type safety rules.
---

# TypeScript Conventions

Apply these when writing, reviewing, or refactoring TypeScript/JavaScript code.

For deeper rationale and examples behind the Jane Street/OCaml-inspired rules, see `references/functional-design.md`.

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

## Strict Mode

Always enable strict mode in `tsconfig.json`. Never use `// @ts-ignore` or `// @ts-nocheck`.

## Naming

| Element | Convention | Example |
|---------|-----------|---------|
| Functions, variables, parameters | `camelCase` | `getUserProfile` |
| Types, classes, enums | `PascalCase` | `UserProfile` |
| Private class members | Leading underscore | `_internalState` |
| Constants (true constants) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
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

## No `unknown` (Except Catch Blocks)

- Allow `unknown` in catch blocks, then narrow it immediately
- Do not use `unknown` as a general input model for application code; validate boundary data with Zod and work with typed values afterward

## Error Handling

- Use custom error classes at meaningful boundaries
- Handle specific error types first
- Log failures with context, then rethrow or map to a user-safe error

## Nullish Operators

- `??` — only catches `null` and `undefined` (NOT empty strings)
- `||` — catches all falsy values including `''`, `0`, `false`

Use `||` when empty strings should fall back. Use `??` when `0` or `false` are valid values.
