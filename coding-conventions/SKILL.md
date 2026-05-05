---
name: coding-conventions
description: Language-agnostic coding conventions. Apply to all code regardless of language. Covers error handling, input validation, config boundaries, import ownership, resource lifecycle, race conditions, security, and consistency. For symbol naming (predicates, variants, domain constants), see /naming. For filesystem layout, see /file-naming. For language-specific rules, see /typescript, /rust, /react. For transport-specific streaming details such as SSE parsing, load a dedicated reference instead of bloating this file.
---

# Coding Conventions — Language-Agnostic Core

These rules apply to **all code** regardless of language or framework. For language-specific conventions, use the dedicated skills:
- TypeScript/JavaScript → `/typescript`
- Rust → `/rust`
- React/Next.js → `/react`
- Design patterns → `/design-patterns`

For naming, load the dedicated skills:
- Symbols (functions, types, predicates, variants, domain constants) → `/naming`
- Files and folders → `/file-naming`

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
- Prefer parameterizing behavior over subtype hierarchies — inject the operation you want rather than creating a new subclass just to override one step
- Prefer named arguments or option objects once positional parameters stop being self-evident, especially for booleans or repeated scalar types

For the naming side of this rule — avoiding generic verbs (`process`, `handle`, `run`) when the operation has domain meaning — see `/naming`.

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
- Leaf helpers should return errors or throw typed failures; they should not terminate the process directly. Keep `process.exit`, HTTP response finalization, and other top-level failure decisions at the outermost boundary
- Repeated top-level error metadata such as script names, error codes, and operator-facing messages should be centralized into named constants instead of duplicated inline
- For operational paths, define the failure surface explicitly: where the failure is detected, where it is translated, where it is logged/emitted, and what the operator sees

## Environment & Config

- Env var parsing has fallbacks (not bare type conversion)
- Numeric config values clamped to sane ranges
- Missing required env vars logged at startup
- Secrets accessed server-side only — never in client bundles
- Keep runtime config entrypoints obvious — one place reads env/process config, the rest of the code consumes normalized values
- Prefer failing fast at startup for missing required config over discovering config gaps mid-request
- Bootstrap paths may depend only on bootstrap-safe config. Do not let initialization logic depend on runtime values that are loaded later from storage, APIs, or generated config

## Imports & Dependencies

- Imports should reflect ownership boundaries — prefer importing from stable public modules over deep internal paths when the project exposes both
- If a module is treated as a public boundary, it must export the stable surface callers actually need. Otherwise, import the concrete owner modules directly instead of mixing a partial barrel with sibling deep imports
- When splitting a boundary across files such as `types.ts`, `result.ts`, `rows.ts`, and `index.ts`, update the barrel and the internal imports in the same change so helper modules do not become accidental type owners
- Remove stale imports as part of the same change that made them unnecessary
- Consolidate imports only when ownership stays the same. Do not import a symbol from a different layer or package just to reduce the number of import lines
- Avoid circular imports created by convenience barrels or cross-layer shortcuts
- If one module exists only to re-export unstable internals, reconsider the boundary instead of adding more imports to it

## Domain Values

For closed-set values that control branching, validation, persistence, or wire-format shape — and for the principle of promoting them into named domain constants instead of ad hoc string literals — see `/naming`.

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

## Operational Review Lens

When code touches scripts, transport adapters, persistence flows, retries, startup/bootstrap, or telemetry:
- Identify the top-level failure boundary before refactoring internals
- Verify there is one authoritative place that shapes operator-facing metadata
- Check that failure paths are as intentional as success paths
- Prefer a short flow or sequence diagram when the failure route crosses multiple modules

## Functional Completeness

- Features are actually wired up — state is set, handlers are connected, IDs are passed through
- Return values from functions are used (not silently discarded when they carry important data)
- If a feature creates items, there's a way to interact with them

## Code Organization

- Feature-based structure over type-based (`/user/` not `/controllers/`, `/models/`)
- Colocate related code — keep tests, types, and implementation together
- Flat over nested — avoid deep directory nesting when possible
- Single responsibility per file — one concept per file
- If a concept is part of the intended public API, let the folder structure reflect that public name. Do not hide first-class modules behind extra nesting that callers immediately route around

## Code Hygiene

- No dead code — unused functions, variables, types, imports
- No stale comments that describe removed/changed behavior
- No commented-out code — use version control
- No duplicate logic — if a pattern appears twice, flag before a third instance

## Streaming References

- Keep transport-specific parsing guidance out of this core file
- If working on Server-Sent Events or another streaming transport, load the dedicated reference at [`references/sse-stream-parsing.md`](/Users/eric.huang/.agents/skills/coding-conventions/references/sse-stream-parsing.md)
