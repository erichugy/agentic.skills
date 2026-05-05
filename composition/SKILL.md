---
name: composition
description: Composition over inheritance principle. Apply when designing shared behavior, code reuse strategies, class hierarchies, or modular feature assembly. Covers function composition, explicit module contracts, injected capabilities, hooks, and when inheritance is actually appropriate.
---

# Composition Over Inheritance

Favor composing small, focused pieces over building deep class hierarchies. This principle applies across languages and paradigms.

Composition is also a module-boundary tool, not just an OOP rule. Use it when a feature can be assembled from explicit parts such as profile, metadata, schema, adapter, and execution-target modules instead of one growing registry or god object.

## The Problem with Inheritance

Inheritance creates tight coupling and fragile hierarchies:
- Changes to a base class cascade to all descendants
- Multiple inheritance leads to diamond problems
- Deep hierarchies are hard to reason about
- You inherit everything, even what you don't need

## Composition Strategies

### 1. Function Composition

Combine small functions into larger behaviors:

```typescript
// BAD — inheritance for shared behavior
class BaseProcessor {
  validate(data: Data) { ... }
  transform(data: Data) { ... }
  save(data: Data) { ... }
}
class UserProcessor extends BaseProcessor { ... }
class OrderProcessor extends BaseProcessor { ... }

// GOOD — compose functions
const validateUser = (data: UserInput) => UserSchema.parse(data)
const transformUser = (user: User) => ({ ...user, name: user.name.trim() })
const saveUser = (repo: UserRepo) => (user: User) => repo.insert(user)

// Compose into a pipeline
function processUser(data: UserInput, repo: UserRepo) {
  const validated = validateUser(data)
  const transformed = transformUser(validated)
  return saveUser(repo)(transformed)
}
```

### 2. Object Composition (Has-A over Is-A)

```typescript
// BAD — "is-a" hierarchy
class Animal { move() { ... } }
class FlyingAnimal extends Animal { fly() { ... } }
class SwimmingAnimal extends Animal { swim() { ... } }
// A duck flies AND swims — now what?

// GOOD — "has-a" composition
type Movable = { move(): void }
type Flyable = { fly(): void }
type Swimmable = { swim(): void }

function createDuck(): Movable & Flyable & Swimmable {
  return {
    move() { ... },
    fly() { ... },
    swim() { ... },
  }
}
```

### 3. React: Hooks over HOCs over Inheritance

```typescript
// BAD — inheritance (never do this in React)
class AuthenticatedComponent extends React.Component { ... }
class UserPage extends AuthenticatedComponent { ... }

// BETTER — HOC (use sparingly)
const withAuth = (Component: FC) => (props: any) => {
  const user = useAuth()
  if (!user) return <Redirect to="/login" />
  return <Component {...props} user={user} />
}

// BEST — custom hooks (preferred)
function UserPage() {
  const user = useAuth()       // compose behaviors via hooks
  const theme = useTheme()
  const data = useUserData(user.id)

  if (!user) return <Redirect to="/login" />
  return <Profile user={user} data={data} />
}
```

### 4. Dependency Injection

Pass capabilities in rather than inheriting them:

```typescript
// BAD — inherit to get logging
class LoggingService { log(msg: string) { ... } }
class UserService extends LoggingService { ... }

// GOOD — inject what you need
type Logger = { log(msg: string): void }

function createUserService(logger: Logger, repo: UserRepo) {
  return {
    async createUser(data: UserInput) {
      const user = await repo.insert(data)
      logger.log(`Created user ${user.id}`)
      return user
    }
  }
}
```

### 4.1 Module Composition

When a domain grows by adding variants, prefer a small module contract over a central registry that keeps absorbing special cases.

```typescript
type ClientModule = {
  profile: ClientProfile
  metadata: ClientMetadata
  requestSchemas: ClientRequestSchemas
  adapters: {
    bootstrap: BootstrapAdapter
  }
  executionTargets: readonly ClientExecutionTarget[]
}
```

This keeps each variant assembled from stable parts and makes additions mostly additive instead of requiring edits across a large coordinator.

Use this especially when transport-specific concerns are bleeding into workflow code. If a feature has concepts like init payloads, health-check metadata, replay builders, and persistence adapters, compose those as separate boundary-owned modules instead of letting one handler coordinate every special case.

### 5. Parameterization over Subtyping

If the variation is "same workflow, different rule", pass the rule in:

```typescript
type Normalizer<T> = (value: T) => string

function dedupeBy<T>(items: readonly T[], normalize: Normalizer<T>): T[] {
  const seen = new Set<string>()

  return items.filter((item) => {
    const key = normalize(item)
    if (seen.has(key)) return false
    seen.add(key)
    return true
  })
}
```

This is often cleaner than `BaseThing` plus subclasses that override one hook method. Use subtype polymorphism when objects truly have distinct lifecycles or identities. Use parameterization when you are selecting behavior.

### 6. Mixins (When Appropriate)

For cross-cutting concerns that genuinely need to be mixed into multiple types:

```typescript
type Timestamped = { createdAt: Date; updatedAt: Date }
type SoftDeletable = { deletedAt: Date | null; isDeleted: boolean }

type User = { id: string; name: string } & Timestamped & SoftDeletable
```

## When Inheritance IS Appropriate

- **Framework requirements** — React class components (legacy), ORM models that require it
- **True "is-a" relationships** — `HttpError extends Error` (an HTTP error truly IS an error)
- **Shallow hierarchies** — one level of inheritance is usually fine
- **Abstract base classes** — when you need to enforce a contract with shared implementation

**Rule of thumb**: If the hierarchy is deeper than 2 levels, refactor to composition.

## Review Checklist

- Can this behavior be assembled from a few explicit parts instead of a larger hierarchy or registry?
- Are variant-specific details isolated in their own modules?
- Is there a small, stable contract for adding the next variant?
- Are raw protocol or config details leaking too far past the boundary?
- Would a new implementation mostly add files, or would it require editing many unrelated modules?
- Does each module own one seam clearly: schema, metadata builder, adapter, persistence mapper, or workflow coordinator?
- If a rename or behavior change happens, can you update one boundary module instead of chasing the same concept through handlers, clients, and docs?
