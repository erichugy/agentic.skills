# Functional Design in TypeScript

This reference translates the useful parts of OCaml and Jane Street style into TypeScript.

The goal is not to imitate OCaml syntax or force everything into point-free code. The goal is to import the habits that make the codebase easier to reason about:

- explicit semantics instead of hidden behavior
- parameterization over subtype hierarchies
- narrow module boundaries
- variant modeling instead of flag soup
- immutable-by-default data flow

## What Transfers Well

| OCaml / Jane Street habit | TypeScript analogue | Why it helps |
|---|---|---|
| Specific operations over polymorphic magic | Pass `compare`, `equals`, `parse`, `encode`, `now` explicitly | Callers can see semantics at the call site |
| Parametric polymorphism | Generics plus injected behavior | Reuse logic without subclass trees |
| Variants + pattern matching | Discriminated unions + exhaustive `switch` | Illegal states become harder to represent |
| Signatures hide representation | Narrow exports, factory/parser boundaries, branded types | Internals can change without breaking callers |
| Immutable data by default | `readonly`, pure transforms, copy-on-write updates | Fewer accidental state bugs |
| Labeled arguments | Named option objects | Calls stay readable when several args share the same type |

## What Does Not Transfer Directly

- OCaml functors and the module system do not map cleanly to TypeScript. Usually the right translation is a factory function or a higher-order function, not an elaborate type-level simulation.
- Hindley-Milner inference is stronger than TypeScript's inference. In TS, exported APIs often still need explicit return types and boundary validation.
- Persistent immutable data structures are not native in JavaScript. Prefer immutable interfaces and isolate mutation rather than pretending all updates are free.

## Pattern 1: Parameterize Behavior, Do Not Subclass for It

If two implementations share the same workflow but differ by one rule, pass the rule in.

```typescript
type Comparator<T> = (left: T, right: T) => number

const sortBy = <T>(items: readonly T[], compare: Comparator<T>): T[] =>
  [...items].sort(compare)

const usersByCreatedAt = sortBy(users, (left, right) =>
  left.createdAt.localeCompare(right.createdAt),
)
```

Prefer this over:

```typescript
class BaseSorter<T> {
  sort(items: readonly T[]): T[] {
    return [...items].sort(this.compare)
  }

  protected compare(left: T, right: T): number {
    throw new Error('Not implemented')
  }
}
```

Why:

- fewer moving parts
- no fake inheritance hierarchy
- easier testing because behavior is just a function
- clearer call sites because the variation is explicit

Use subtype polymorphism when objects truly differ in lifecycle, identity, or framework behavior. Use parameterization when you are selecting an algorithm or policy.

## Pattern 2: Make Semantics Explicit at Boundaries

If correctness depends on a decision, name that decision in the API.

Examples:

- sorting: accept a comparator
- equality: accept `equals`
- IDs: parse them once, then use a branded type
- time: inject `now`
- randomness: inject `random`
- serialization: expose explicit `parse` and `encode`

```typescript
type Clock = { now(): Date }

function createSession(clock: Clock): Session {
  return {
    createdAt: clock.now().toISOString(),
  }
}
```

This is the TypeScript version of avoiding ambient, polymorphic behavior that "works" but hides intent.

## Pattern 3: Prefer Variants over Boolean Matrices

When a value can be in one of several states, model the states directly.

```typescript
type SyncState =
  | { kind: 'idle' }
  | { kind: 'running'; startedAt: string }
  | { kind: 'failed'; message: string }
  | { kind: 'complete'; updatedCount: number }
```

Avoid shapes like:

```typescript
type SyncState = {
  isRunning: boolean
  hasFailed: boolean
  isComplete: boolean
  error?: string
  updatedCount?: number
}
```

The variant form is closer to OCaml's algebraic data types: it prevents impossible combinations and forces exhaustive handling.

## Pattern 4: Hide Representation Behind Narrow APIs

Keep unchecked shapes and implementation details inside the module.

```typescript
type UserId = {
  readonly value: string
}

function parseUserId(raw: string): UserId {
  if (raw.length === 0) throw new Error('Invalid user id')
  return { value: raw }
}
```

The key idea is not the wrapper itself. The key idea is that callers should not construct important domain values ad hoc.

Good signs:

- one parser or factory owns validation
- callers work with a domain type, not arbitrary strings
- exports expose capability, not internals

If a project uses branded types, keep any unavoidable assertion contained to one boundary module rather than spreading casts through application code.

## Pattern 5: Immutable by Default, Mutation by Exception

Default to pure transforms and `readonly` inputs:

```typescript
function renameUser(user: Readonly<User>, name: string): User {
  return { ...user, name }
}
```

Allow mutation when:

- performance has been measured
- the framework expects mutation
- the mutation is isolated behind a narrow abstraction

The rule is not "never mutate." The rule is "make mutation local and intentional."

## Pattern 6: Use Named Option Objects for Ambiguous Calls

If a function takes more than a couple of scalar arguments, especially booleans, use an object.

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

This gives TypeScript something close to labeled arguments. The trade-off is a slightly larger call site, but the gain is long-term readability.

## Review Questions

Ask these when reviewing TypeScript:

1. Is the behavior explicit, or does the code rely on hidden defaults?
2. Is a subclass being used where a function parameter would be simpler?
3. Are variants modeled as discriminated unions, or as loose bags of flags?
4. Does the module hide its representation, or do callers depend on raw shapes?
5. Is mutation isolated and justified?
6. Would named option objects make the call site easier to read?

## Source Notes

These ideas were distilled from OCaml and Jane Street materials, especially:

- Jane Street Base README: <https://github.com/janestreet/base/blob/master/README.org>
- Real World OCaml: <https://dev.realworldocaml.org/>

Use those as inspiration, not as a mandate to make TypeScript look like OCaml.
