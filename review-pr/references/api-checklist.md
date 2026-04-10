# API & Backend Review Checklist

Applies to Next.js API routes, server-side code, and backend services.

## Route Configuration
- [ ] `export const runtime = "nodejs"` present on routes using runtime state or Node APIs
- [ ] `export const dynamic = "force-dynamic"` on routes using env vars or in-memory state
- [ ] Explicit `Promise<NextResponse>` return types on ALL exported handler functions
- [ ] CORS headers present on public-facing endpoints

## HTTP Methods
- [ ] OPTIONS handler returns `204 No Content` with CORS headers (not a JSON body)
- [ ] HEAD handler doesn't return a body
- [ ] All expected HTTP methods are handled (GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS)

## Request Validation
- [ ] Request body parsed in try/catch (`req.json()` can throw)
- [ ] Zod `safeParse` for user-facing validation errors
- [ ] String inputs trimmed and length-capped
- [ ] Numeric inputs parsed with fallbacks (not bare `parseInt`/`Number`)

## Rate Limiting
- [ ] IP extraction uses `||` (not `??`) to catch empty strings
- [ ] Fallback key for missing IP (not `null` which skips limiting)
- [ ] In-memory rate limit maps have hard size caps with eviction
- [ ] Cleanup interval uses `.unref()` to not block Node.js exit
- [ ] Opportunistic cleanup in hot paths (don't rely solely on timers)

## Error Responses
- [ ] No sensitive info in error messages (no stack traces, no internal URLs)
- [ ] Error messages are user-friendly and actionable
- [ ] Consistent error shape (`{ error: string }`) across all error responses
- [ ] HTTP status codes are correct (400 validation, 429 rate limit, 500 server, 503 unavailable)

## Environment Variables
- [ ] Server-only vars never prefixed with `NEXT_PUBLIC_`
- [ ] Missing required vars return 503 with graceful message (not crash)
- [ ] Env var parsing with try/catch and fallback defaults
- [ ] Startup warnings logged for missing optional vars

## Python/Flask Backend
- [ ] Module-level singletons are thread-safe (use locks for lazy init)
- [ ] Imports at top level (not inside function bodies unless conditional)
- [ ] `datetime.utcnow()` for UTC consistency (not `datetime.now()`)
- [ ] Exception handling: specific types first, broad `except Exception` last with logging
- [ ] Error strings from caught exceptions not fed into downstream data processing
