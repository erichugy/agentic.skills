# Module Boundaries

Detailed guidance for option objects, exports, boundary ownership, alias refactors, `unknown`, errors, and nullish operators.

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
