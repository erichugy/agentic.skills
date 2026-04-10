# General Code Review Checklist

Applies to ALL code changes. Every item must be checked.

## Dead Code & Unused Declarations
- [ ] No unused functions, variables, types, or imports (triggers lint warnings)
- [ ] No dead code paths (unreachable branches, no-op assignments)
- [ ] No stale comments that describe removed/changed behavior
- [ ] No dead `dataset.*` assignments or other DOM property writes that nothing reads

## Type Safety & Conventions
- [ ] `type` over `interface` (unless extending)
- [ ] No `as` type assertions (except isolated narrowing helpers)
- [ ] Explicit return types on ALL exported functions
- [ ] Import ordering: builtin/external → parent → sibling (blank lines between groups)
- [ ] Explicit `import type React from "react"` when using `React.*` types (don't rely on auto-injection)

## Comments
- [ ] Only "the why", never "the what" (Conventional Comments labels: NOTE, TODO, FIXME, etc.)
- [ ] No stale comments that contradict the current code
- [ ] Commented-out code removed (use version control)

## Input Validation
- [ ] Whitespace-only strings rejected where appropriate (`.trim()` before `.min()`)
- [ ] Max lengths on ALL string inputs (including email — max 320)
- [ ] `safeParse` preferred over `parse` for user-facing Zod errors
- [ ] `response.json()` wrapped in try/catch (can throw on invalid/empty JSON even with 2xx)
- [ ] Imported/deserialized data deeply validated before use (not just top-level fields)

## Network & Timeouts
- [ ] All outbound `fetch` calls have timeouts via `AbortController` + `setTimeout`
- [ ] `clearTimeout` in `finally` blocks (not just success path)
- [ ] AbortController refs cleaned up on component unmount (`useEffect` cleanup)

## Resource Lifecycle
- [ ] `setInterval` timers call `.unref()` in Node.js
- [ ] In-memory stores (Maps, Sets) have hard size caps with eviction
- [ ] Opportunistic cleanup in hot paths (don't rely solely on timers in serverless)

## Race Conditions & State
- [ ] Double-submit guards on form/action handlers
- [ ] State updates after unmount prevented (`isMountedRef` or stale-request detection)
- [ ] ALL state setters guarded (not just `setIsLoading` — also `setError`, `setResult`, etc.)
- [ ] Stale request detection: verify the current controller/ID is still active before updating state

## Error Handling
- [ ] Error strings never fed into downstream processing
- [ ] Error messages are user-friendly and actionable
- [ ] Failed operations logged with context (URL, status code)
- [ ] Silent `catch` blocks have at least a warning log
- [ ] Failed data fetches never produce misleading "empty" UI — show a warning when data couldn't be loaded vs. when it's genuinely empty (e.g., "0 files" should not mean "fetch failed silently")
- [ ] `Promise.all` reviewed for partial-failure impact — use `Promise.allSettled` when independent operations should not discard each other's results on failure

## Environment & Config
- [ ] Env var parsing has try/except with fallbacks (not bare `int()` / `Number()`)
- [ ] Numeric config values clamped to sane ranges
- [ ] Missing required env vars logged at startup

## Consistency
- [ ] Date/time APIs use UTC consistently
- [ ] `??` vs `||` used correctly — `??` doesn't catch empty strings, `||` does
- [ ] String comparison case consistency — if some operators lowercase, ALL should (e.g., `equals` should match `contains` behavior)
- [ ] Code formatting consistent with codebase (multi-line types, trailing semicolons)
- [ ] Magic numbers extracted to named constants in a shared location, not inline in function bodies
- [ ] Logic duplicated between server and client lifted into a shared module (don't maintain two copies)

## Functional Completeness
- [ ] Features are actually wired up — state is set, handlers are connected, IDs are passed through
- [ ] If a feature creates items (e.g., filter groups), there's a way to add content to them
- [ ] Return values from functions are used (not silently discarded when they carry important data like IDs)

## Security
- [ ] Sensitive values only accessed server-side
- [ ] No sensitive info leaked in error responses
- [ ] Rate limiting fallback key uses `||` (not `??`) and has a non-null default
