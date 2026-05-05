---
name: rust
description: Rust coding conventions. Apply when writing, reviewing, or refactoring any .rs code. Covers ownership, error handling, casing, module organization, and idiomatic patterns. For symbol naming principles (predicates, enum-variant naming, generic-verb smells, closed-set values as named constants), see /naming.
---

# Rust Conventions

Apply these when writing, reviewing, or refactoring Rust code.

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

For cross-cutting symbol naming (predicates as questions, narrowing-predicate target naming, enum-variant naming, generic-verb smells, promoting closed-set values into named constants), see `/naming`. The casing table below is the Rust-specific layer on top of those principles.

## Casing

| Element | Convention | Example |
|---------|-----------|---------|
| Functions, methods, variables | `snake_case` | `get_user_profile` |
| Types, traits, enums | `PascalCase` | `UserProfile` |
| Constants, statics | `UPPER_SNAKE_CASE` | `MAX_CONNECTIONS` |
| Modules, crates | `snake_case` | `user_profile` |
| Lifetimes | Short lowercase | `'a`, `'ctx` |
| Type parameters | Single uppercase or short PascalCase | `T`, `Key` |

## Error Handling

Use `Result<T, E>` for recoverable errors. Never use `.unwrap()` or `.expect()` in library code or production paths:

```rust
// BAD — panics in production
let config = fs::read_to_string("config.toml").unwrap();

// GOOD — propagate with ?
let config = fs::read_to_string("config.toml")?;

// GOOD — context with anyhow/thiserror
let config = fs::read_to_string("config.toml")
    .context("Failed to read config file")?;
```

### Custom Error Types

Use `thiserror` for library errors, `anyhow` for application errors:

```rust
#[derive(Debug, thiserror::Error)]
enum AppError {
    #[error("User not found: {0}")]
    UserNotFound(String),
    #[error("Database error")]
    Database(#[from] sqlx::Error),
    #[error("Invalid input: {0}")]
    Validation(String),
}
```

### Option Handling

Use combinators over match when the logic is simple:

```rust
// Prefer
let name = user.name.as_deref().unwrap_or("anonymous");

// Over
let name = match &user.name {
    Some(n) => n.as_str(),
    None => "anonymous",
};
```

## Ownership & Borrowing

- Prefer borrowing (`&T`) over ownership when the function doesn't need to own the data
- Use `&str` over `String` in function parameters
- Use `impl Into<String>` or `AsRef<str>` for flexible APIs
- Clone explicitly and intentionally — never to "make the borrow checker happy" without understanding why
- Use `Cow<'_, str>` when a function sometimes needs to allocate and sometimes doesn't

## Structs & Enums

- Derive common traits: `#[derive(Debug, Clone, PartialEq)]` at minimum
- Use `#[non_exhaustive]` on public enums that may grow
- Builder pattern for structs with many optional fields
- Newtype pattern for type safety: `struct UserId(u64)` over bare `u64`

## Module Organization

```
src/
├── lib.rs or main.rs    <- Public API / entry point
├── config.rs            <- Configuration
├── error.rs             <- Error types
├── models/              <- Data types
│   ├── mod.rs
│   └── user.rs
├── handlers/            <- Request handlers / business logic
│   ├── mod.rs
│   └── auth.rs
└── utils/               <- Shared utilities
```

- Re-export public types from `mod.rs` or `lib.rs`
- Keep modules focused on a single domain concept
- Use `pub(crate)` for internal-only items

## Formatting & Linting

- Always run `cargo fmt` before committing
- Always run `cargo clippy -- -D warnings` and fix all warnings
- Enable `#![deny(clippy::all)]` in lib.rs/main.rs for strict linting

## Concurrency

- Prefer `tokio` for async runtime (unless the project uses something else)
- Use `Arc<Mutex<T>>` sparingly — prefer message passing with channels
- Use `RwLock` when reads vastly outnumber writes
- Never hold a lock across an `.await` point

## Performance Idioms

- Use iterators over manual loops: `.iter().filter().map().collect()`
- Preallocate with `Vec::with_capacity()` when size is known
- Use `&[T]` over `&Vec<T>` in function signatures
- Avoid unnecessary allocations — `format!` allocates, string slicing doesn't
