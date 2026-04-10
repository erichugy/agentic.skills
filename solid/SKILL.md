---
name: solid
description: SOLID principles for software design. Apply when designing modules, classes, or system architecture. Covers single responsibility, open/closed, Liskov substitution, interface segregation, and dependency inversion with practical examples.
---

# SOLID Principles

Apply these principles when designing modules, classes, functions, or system architecture. They apply across languages — examples use TypeScript but the concepts are universal.

## S — Single Responsibility Principle

> A module should have one, and only one, reason to change.

Each function, class, or module should own exactly one piece of functionality. If you find yourself saying "and" when describing what it does, split it.

```typescript
// BAD — does validation AND persistence AND notification
class UserService {
  createUser(data: UserInput) {
    this.validate(data)      // validation concern
    this.db.insert(data)     // persistence concern
    this.mailer.send(data)   // notification concern
  }
}

// GOOD — each concern in its own module
class UserValidator { validate(data: UserInput): ValidationResult { ... } }
class UserRepository { insert(user: User): Promise<User> { ... } }
class UserNotifier { sendWelcome(user: User): Promise<void> { ... } }
```

**Test**: If a change to database logic forces you to modify notification logic, SRP is violated.

## O — Open/Closed Principle

> Software entities should be open for extension but closed for modification.

Add new behavior by adding new code, not by changing existing code. Use composition, strategy pattern, or plugin systems.

```typescript
// BAD — adding a new format requires modifying this function
function exportData(data: Data, format: string) {
  if (format === 'csv') { ... }
  else if (format === 'json') { ... }
  else if (format === 'xml') { ... }  // new format = modify existing code
}

// GOOD — new formats are added without touching existing code
type Exporter = (data: Data) => string

const exporters: Record<string, Exporter> = {
  csv: exportCsv,
  json: exportJson,
}

// Adding XML = just add to the registry, no existing code modified
exporters.xml = exportXml
```

**Test**: Can you add a new variant without modifying existing functions? If not, apply O/C.

## L — Liskov Substitution Principle

> Subtypes must be substitutable for their base types without altering correctness.

If code works with a base type, it must also work with any subtype. Subtypes must not weaken postconditions, strengthen preconditions, or throw unexpected errors.

```typescript
// BAD — Square violates Rectangle's contract
class Rectangle {
  setWidth(w: number) { this.width = w }
  setHeight(h: number) { this.height = h }
  area() { return this.width * this.height }
}

class Square extends Rectangle {
  setWidth(w: number) { this.width = w; this.height = w }  // breaks expectations
}

// GOOD — model them as separate types with a shared interface
type Shape = { area(): number }
function createRectangle(w: number, h: number): Shape { ... }
function createSquare(side: number): Shape { ... }
```

**Test**: Can you replace a parent type with any child type and have all existing tests still pass?

## I — Interface Segregation Principle

> No client should be forced to depend on methods it doesn't use.

Split large interfaces into focused ones. Consumers only depend on the slice they need.

```typescript
// BAD — forces all implementations to define methods they don't need
type DataStore = {
  read(key: string): Promise<string>
  write(key: string, value: string): Promise<void>
  delete(key: string): Promise<void>
  subscribe(key: string, cb: Callback): Unsubscribe
  getMetrics(): StoreMetrics
}

// GOOD — consumers pick the interface they need
type Readable = { read(key: string): Promise<string> }
type Writable = { write(key: string, value: string): Promise<void> }
type Deletable = { delete(key: string): Promise<void> }
type Observable = { subscribe(key: string, cb: Callback): Unsubscribe }

// A read-only cache only needs Readable
function buildCache(store: Readable) { ... }
```

**Test**: Does your implementation have methods that throw "not implemented"? If so, the interface is too broad.

## D — Dependency Inversion Principle

> High-level modules should not depend on low-level modules. Both should depend on abstractions.

Pass dependencies in (injection), don't import them directly. This makes code testable and swappable.

```typescript
// BAD — tightly coupled to a specific database
import { PostgresDB } from './postgres'

function getUser(id: string) {
  const db = new PostgresDB()  // hard dependency
  return db.query('SELECT * FROM users WHERE id = $1', [id])
}

// GOOD — depends on an abstraction
type UserRepository = {
  findById(id: string): Promise<User | null>
}

function createUserService(repo: UserRepository) {
  return {
    getUser: (id: string) => repo.findById(id),
  }
}

// Wire up at the boundary
const service = createUserService(new PostgresUserRepository(pool))
```

**Test**: Can you swap the database implementation without changing business logic? If not, apply DIP.

## When to Apply

- **Always** apply SRP — it's the foundation
- Apply O/C when you see growing switch/if chains for variants
- Apply LSP when using inheritance or polymorphism
- Apply ISP when interfaces grow beyond 3-4 methods
- Apply DIP at system boundaries (database, HTTP, file system, third-party services)
- **Don't over-apply** — a simple script doesn't need DIP. Apply when complexity warrants it.
