---
name: coding-conventions
description: Language-agnostic coding conventions. Apply to all code regardless of language. Covers error handling, input validation, config boundaries, import ownership, reusable domain values, resource lifecycle, race conditions, security, and consistency. For language-specific rules, see /typescript, /rust, /react. For transport-specific streaming details such as SSE parsing, load a dedicated reference instead of bloating this file.
---

# Coding Conventions — Language-Agnostic Core

These rules apply to **all code** regardless of language or framework. For language-specific conventions, use the dedicated skills:
- TypeScript/JavaScript → `/typescript`
- Rust → `/rust`
- React/Next.js → `/react`
- Design patterns → `/design-patterns`

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

## Input Validation

- Trim whitespace before length checks (`.trim()` before `.min()`)
- Max lengths on ALL string inputs (including email — max 320)
- Validate external data at system boundaries (user input, API responses, deserialized data)
- Deeply validate imported/deserialized data — not just top-level fields
- Prefer graceful validation (return errors) over throwing for user-facing inputs
- Be explicit about transport boundaries: parse raw input once at the edge, convert it into typed domain data, and keep the rest of the system insulated from wire-format quirks

## Explicit Semantics

- Prefer explicit behavior over ambient magic — pass comparators, equality, clocks, randomness, codecs, and side-effecting capabilities in when semantics matter
- Avoid overly generic helpers with hidden behavior (`process`, `handle`, `run`) when the real operation has domain meaning (`compareUsersByCreatedAt`, `encodeSessionToken`)
- Prefer parameterizing behavior over subtype hierarchies — inject the operation you want rather than creating a new subclass just to override one step
- Prefer named arguments or option objects once positional parameters stop being self-evident, especially for booleans or repeated scalar types

## Network & Timeouts

- All outbound network calls have timeouts (language-appropriate mechanism)
- Timeout cleanup in finally/defer/drop blocks — not just the success path
- Cancel in-flight requests on component unmount or scope exit

## Resource Lifecycle

- Timers and intervals are cleaned up (unref in Node.js, cancel in other contexts)
- In-memory stores (maps, caches) have hard size caps with eviction
- Opportunistic cleanup in hot paths — don't rely solely on timers (especially in serverless)
- File handles, database connections, sockets are closed in finally/defer/drop blocks

## Race Conditions & State

- Double-submit guards on form/action handlers
- Stale request detection — verify the current request is still active before updating state
- All state updates guarded against stale/unmounted contexts
- Concurrent writes to shared state use appropriate synchronization

## Error Handling

- Error messages are user-friendly and actionable (not raw framework errors)
- Failed operations logged with context (URL, status code, operation name)
- No silent catch blocks — at minimum, log a warning
- Sensitive information never leaked in error messages (no stack traces, internal URLs, secrets)
- Error strings never fed into downstream processing
- Failed data fetches must not produce misleading "empty/zero" UI — distinguish "no data" from "fetch failed"
- Use `Promise.allSettled` over `Promise.all` when independent operations should preserve partial results on failure

## Environment & Config

- Env var parsing has fallbacks (not bare type conversion)
- Numeric config values clamped to sane ranges
- Missing required env vars logged at startup
- Secrets accessed server-side only — never in client bundles
- Keep runtime config entrypoints obvious — one place reads env/process config, the rest of the code consumes normalized values
- Prefer failing fast at startup for missing required config over discovering config gaps mid-request

## Imports & Dependencies

- Imports should reflect ownership boundaries — prefer importing from stable public modules over deep internal paths when the project exposes both
- Remove stale imports as part of the same change that made them unnecessary
- Avoid circular imports created by convenience barrels or cross-layer shortcuts
- If one module exists only to re-export unstable internals, reconsider the boundary instead of adding more imports to it

## Domain Values

- Do not scatter repeated string literals when they represent a closed set of valid values
- Promote reusable domain values into shared constants, enums, or equivalent typed constructs early
- If a value controls branching, validation, persistence, or network payload shape, it should usually have a named domain representation rather than ad hoc string literals
- Keep raw protocol strings at the boundary when needed, but map them into domain-level names as soon as practical

## Consistency

- Date/time APIs use UTC consistently
- String comparison case sensitivity is consistent across all operators
- Code formatting matches the project's existing style
- Import ordering follows project conventions

## Working Mode

- In greenfield code, choose the clearest architecture and strongest defaults up front: explicit boundaries, small pure functions, immutable-by-default data flow, and names that reveal intent
- In an existing repo, preserve local patterns unless they are actively causing bugs, confusion, or repeated churn
- Optimize for minimal, high-signal diffs in existing code — don't rewrite modules just to make them look like your preferred style
- When improving existing code, strengthen boundaries and naming at the seam you are already touching instead of doing broad style migrations

## Functional Completeness

- Features are actually wired up — state is set, handlers are connected, IDs are passed through
- Return values from functions are used (not silently discarded when they carry important data)
- If a feature creates items, there's a way to interact with them

## Code Organization

- Feature-based structure over type-based (`/user/` not `/controllers/`, `/models/`)
- Colocate related code — keep tests, types, and implementation together
- Flat over nested — avoid deep directory nesting when possible
- Single responsibility per file — one concept per file

## Code Hygiene

- No dead code — unused functions, variables, types, imports
- No stale comments that describe removed/changed behavior
- No commented-out code — use version control
- No duplicate logic — if a pattern appears twice, flag before a third instance

## Streaming References

- Keep transport-specific parsing guidance out of this core file
- If working on Server-Sent Events or another streaming transport, load the dedicated reference at [`references/sse-stream-parsing.md`](/Users/eric.huang/.agents/skills/coding-conventions/references/sse-stream-parsing.md)
