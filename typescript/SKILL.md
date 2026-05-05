---
name: typescript
description: TypeScript and JavaScript coding conventions. Apply when writing, reviewing, or refactoring any .ts, .tsx, .js, or .jsx code. Covers strict types, Zod validation, casing, imports, and type safety rules. For symbol naming principles (predicates, variants, domain constants), see /naming. For filesystem layout, see /file-naming.
---

# TypeScript Conventions

Apply these when writing, reviewing, or refactoring TypeScript/JavaScript code.

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

## When to Use

Use this skill for:
- Writing, reviewing, or refactoring `.ts`, `.tsx`, `.js`, or `.jsx` files
- Type safety, Zod validation, import organization, and strict-mode questions
- API contracts, boundary schemas, discriminated unions, and type-modeling choices
- TypeScript-specific casing conventions layered on top of `/naming`

Also use:
- `/naming` for predicates, narrowing guards, union/variant names, generic-verb smells, and domain constants
- `/file-naming` for filesystem layout
- `/react` for React/Next.js component conventions
- [functional design rationale](references/functional-design.md) for Jane Street/OCaml-inspired rules

## Quick Reference

| Area | Rule | Details |
|------|------|---------|
| Strict mode | Enable strict mode; never use `// @ts-ignore` or `// @ts-nocheck` | See [type modeling](references/type-modeling.md). |
| Type declarations | Prefer `type` over `interface` unless declaration merging or class implementation is needed | See [type modeling](references/type-modeling.md). |
| Exports | Add return types on exported functions | Inference is fine for small internal helpers. |
| Subtyping | Prefer generics plus explicit behavior parameters over subclass trees | See [type modeling](references/type-modeling.md). |
| Variants | Use discriminated unions over flag combinations | See [type modeling](references/type-modeling.md). |
| Runtime values | Model closed branching values once with schema-backed domain constants | See [type modeling](references/type-modeling.md). |
| Imports | Group imports by source and alphabetize within each group | See [imports and validation](references/imports-validation.md). |
| Type-only imports | Use `import type` for type-only imports | See [imports and validation](references/imports-validation.md). |
| External data | Validate API responses, user inputs, and deserialized data with Zod | See [imports and validation](references/imports-validation.md). |
| Assertions | Avoid `as`; use Zod or type guards instead | See [imports and validation](references/imports-validation.md). |
| Ambiguous calls | Use named option objects for multiple booleans or repeated scalar types | See [module boundaries](references/module-boundaries.md). |
| Boundaries | Keep exports narrow and boundary ownership obvious | See [module boundaries](references/module-boundaries.md). |
| `unknown` | Allow in catch blocks only, then narrow immediately | See [module boundaries](references/module-boundaries.md). |
| Nullish operators | Use `||` when empty strings should fall back; use `??` when `0` or `false` are valid | See [module boundaries](references/module-boundaries.md). |

## Casing

| Element | Convention | Example |
|---------|-----------|---------|
| Functions, variables, parameters | `camelCase` | `getUserProfile` |
| Types, classes, enums | `PascalCase` | `UserProfile` |
| Private class members | Leading underscore | `_internalState` |
| Constants (true constants and schema-derived enum objects) | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `PERSISTENCE_WARNING_SCOPE` |
| Files | `kebab-case` or match export name | `user-profile.ts` |

For cross-cutting symbol naming, see `/naming`. The casing table is the TypeScript-specific layer on top of those principles.

## Boundary Review Checklist

- Does this boundary shape have one obvious owner?
- Are branching values modeled once as shared domain values rather than repeated strings?
- Will the next caller import the same schema/type, or are we encouraging local copies?
- If this contract changes next month, is there one migration point or many?

## Decision Points

- Reach for classes when you need lifecycle, identity, or framework integration; do not introduce subclasses just to swap parsing, sorting, formatting, or validation logic.
- Use `.parse()` when invalid data should fail fast; use `.safeParse()` when you need a fallback path.
- Let boundary-owned schema names reflect current domain behavior, not stale transport history.
- Import cleanup must preserve ownership. Never pull a symbol from a different layer just because another import from that layer already exists.

## Reference Files

- [Type modeling](references/type-modeling.md) — strict mode, types vs interfaces, parametric APIs, discriminated unions, and schema-backed domain constants
- [Imports and validation](references/imports-validation.md) — import ordering, `import type`, Zod validation, no `as`, and schema naming
- [Module boundaries](references/module-boundaries.md) — option objects, narrow exports, opaque boundaries, aliases, `unknown`, errors, and nullish operators
- [Functional design rationale](references/functional-design.md) — deeper rationale and examples behind the Jane Street/OCaml-inspired rules
