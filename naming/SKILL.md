---
name: naming
description: Cross-cutting symbol naming conventions for functions, variables, types, parameters, and tagged variants. Apply when introducing or renaming any named symbol in any language. Covers predicate naming, narrowing-predicate target naming, union-vs-variant naming, avoiding wrapper predicates, generic-verb smells, and promoting closed-set values into named domain constants. For filesystem layout, see /file-naming. For language-specific casing tables and syntax (type guards, enum variants, etc.), load the matching language skill.
---

# Naming Conventions

Apply these when introducing or renaming any named symbol — functions, variables, types, parameters, tagged variants, constants. The rules below are language-agnostic; load the language skill (`/typescript`, `/rust`, `/react`) for per-language casing tables and syntax. For filesystem layout, see `/file-naming`.

## Core Principle

> A name should answer the question its caller is asking.

Read the call site aloud. If the name doesn't make the line read like prose — or worse, reads like a tautology — the name is wrong, not the call.

## Quick Rules

- Predicates are questions. Name them so the call site reads as one
- Type guards and other narrowing predicates are named after what they narrow **to**, not the union they're checking
- The union owns the concept name; variants get the qualifier
- Avoid wrapper predicates that re-type an existing predicate without adding a check
- A 4+ content-word symbol name is a smell — split the function or reframe the call site
- Avoid generic verbs (`process`, `handle`, `run`, `manage`) when the operation has a domain meaning (`encodeSessionToken`, `compareUsersByCreatedAt`)
- Promote closed-set values into named domain constants — don't scatter string literals across handlers, schemas, and telemetry

## Predicates as Questions

The call site is the readability test. If the predicate doesn't make the line read like a question, rename.

```
# call site
if (isSelectedForExport(id, filter)) { ... }            ✓ reads like a question
if (isIdIncludedInExportSelection(id, filter)) { ... }  ✗ verbose; the union name leaks into the predicate
if (isExportSelection(filter)) { ... }                  ✗ tautology — of course the export filter is "an export selection"
```

A 4+ content-word predicate name is almost always one of:

- The function is doing two things (narrowing AND a membership/value check) — **split it**
- The call site is missing domain framing — **reframe** (`isExportable(id)` instead of `isIdIncludedInExportSelection(id, filter)`)

## Narrowing Predicates Are Named After the Variant

A type guard, refinement, or pattern match narrows from a broader type to a specific variant. Name it after the variant — not after the union being checked.

```typescript
type ExportSelection = 'all' | 'none' | ExportSelectionRule

// BAD — name doesn't say what we narrowed to
const isExportSelection = (v: ExportSelection): v is ExportSelectionRule => ...

// GOOD — narrows to the rule variant
const isExportSelectionRule = (v: ExportSelection): v is ExportSelectionRule => ...
```

```rust
enum ExportSelection { All, None, Rule { mode: Mode, ids: Vec<String> } }

// BAD — generic; says nothing about which variant
fn is_export_selection(s: &ExportSelection) -> bool { matches!(s, ExportSelection::Rule { .. }) }

// GOOD — names the variant being matched
fn is_explicit_rule(s: &ExportSelection) -> bool { matches!(s, ExportSelection::Rule { .. }) }
```

```python
# Python equivalent — predicate over a Union[Literal['all'], Literal['none'], ExportSelectionRule]
def is_export_selection_rule(value: ExportSelection) -> TypeGuard[ExportSelectionRule]: ...
```

## Union Owns the Concept; Variant Gets the Qualifier

When the inner shape sounds broader than the union it lives inside, the mental model is inverted. The union should own the concept name; variants take qualifiers.

```typescript
// BAD — the object form sounds like the broader concept
type ExportSelection = { mode: 'include' | 'exclude'; ids: string[] }
type ExportSelectionFilter = 'all' | 'none' | ExportSelection

// GOOD — the union is the concept; the variant is qualified
type ExportSelectionRule = { mode: 'include' | 'exclude'; ids: string[] }
type ExportSelection = 'all' | 'none' | ExportSelectionRule
```

The same rule applies to Rust enums, Haskell sum types, Kotlin sealed classes, and Python tagged dataclasses — anywhere a tagged set of variants lives under a single umbrella name.

## Avoid Wrapper Predicates

A predicate that just re-types an existing predicate against a type alias adds a hop without clarifying the narrowing target. Drop it and call the underlying predicate directly.

```typescript
// BAD — same logic as isExportSelection, just typed against a downstream alias
const isTableExportSelection = (v: TableExportFilter): v is Exclude<TableExportFilter, 'all' | 'none'> =>
  isExportSelection(v)

// GOOD — drop the wrapper; call sites use the original predicate
```

A wrapper predicate is usually a sign that a downstream type alias was introduced where a re-export would have done. The wrapper compounds the problem at every call site.

## Avoid Generic Verbs When the Operation Has Domain Meaning

```
process(user)        # process what? saves? validates? ranks?
handle(event)        # handles how?
run(thing)           # runs what?
manage(resource)     # manages how?
```

Replace with the verb that actually names the operation:

```
encodeSessionToken(user)
applyDiscount(order)
publishBotConfig(bot)
acquireDatabaseLock(resource)
```

Generic verbs are forgivable in adapter layers where the framework supplies the meaning (HTTP `handle`, lifecycle `run`, queue `process`). Anywhere else, prefer the verb that names the actual operation.

## Promote Closed-Set Values into Named Domain Constants

If a value controls branching, validation, persistence, or wire-format shape, it should have a named representation rather than ad hoc string literals scattered across the codebase.

```typescript
// BAD — strings drift across handlers, schemas, telemetry
if (job.scope === 'persistent') { ... }
emit({ scope: 'persistent', ... })

// GOOD — one source of truth
export const PERSISTENCE_SCOPE = { JOB: 'job', RESULT: 'result' } as const
type PersistenceScope = typeof PERSISTENCE_SCOPE[keyof typeof PERSISTENCE_SCOPE]
```

```rust
// BAD
if status == "pending" { ... }

// GOOD
#[derive(Debug, Clone, Copy)]
enum JobStatus { Pending, Running, Done, Failed }
```

Keep raw protocol strings at the boundary when needed, but map them into domain-level names as soon as practical. The same rule applies to operational metadata such as failure boundaries, warning scopes, and event tags — treat them as domain values, not disposable strings.

For language-specific encodings (Zod schema/type/enum trio in TypeScript; Rust `enum` derive patterns; etc.), see the matching language skill.

## Review Checklist

- [ ] Does the call site read like the question being asked?
- [ ] Does each predicate's name describe what it narrows TO, not what it checks against?
- [ ] Does the union own the concept, with variants taking qualifiers?
- [ ] Are there wrapper predicates that just re-type an existing one against an alias?
- [ ] Are generic verbs (`process`, `handle`, `run`) hiding a real operation?
- [ ] Are closed-set values represented as named domain constants, not raw strings?
- [ ] Is any symbol name 4+ content words long? If so, can it be split or reframed?

## Related Skills

- `/file-naming` — file and folder naming (filesystem layout)
- `/typescript`, `/rust`, `/react` — per-language casing tables and language-specific syntax for type guards, enums, and schema/type/enum trios
- `/coding-conventions` — language-agnostic rules for everything else (error handling, validation, resource lifecycle, etc.)
