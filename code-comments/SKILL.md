---
name: code-comments
description: Code comment conventions following Conventional Comments standard. Apply when writing or reviewing code comments.
---

# Code Comments - Conventional Comments

Apply the Conventional Comments standard when writing or reviewing code comments.

## Project-Specific Conventions (MANDATORY)

**Before reviewing comments, check for project-specific rules.** Look for:
1. `AGENTS.md` in the current workspace or app directory (e.g., `apps/web/AGENTS.md`)
2. `CLAUDE.md` in the repo root or workspace root

Project-specific comment rules (e.g., "Comments: only the why, never the what") **override** the defaults below.

## Format

```
<label> [decorations]: <subject>
```

- **label**: Category of the comment (required)
- **decorations**: Additional context in parentheses (optional)
- **subject**: The main message (required)

## Labels

| Label | Meaning | When to use |
|-------|---------|-------------|
| `TODO` | Task to complete | Future work that should be done |
| `FIXME` | Known issue | Problem that needs fixing but isn't blocking |
| `HACK` | Workaround | Technical debt, temporary solution |
| `BUG` | Known bug | Documented bug, often with issue reference |
| `NOTE` | Important context | Explanation that helps understanding |
| `QUESTION` | Needs clarification | Uncertainty that needs resolution |
| `OPTIMIZE` | Performance | Potential performance improvement |
| `DEPRECATED` | Scheduled for removal | Code that will be removed |

## Decorations

Add decorations in parentheses after the label for additional context:

| Decoration | Meaning |
|------------|---------|
| `(blocking)` | Must be resolved before merge |
| `(non-blocking)` | Nice to have, not required |
| `(if-minor)` | Only fix if making other changes nearby |
| `(issue)` | Has associated issue tracker reference |

## Examples

```typescript
// TODO: Add pagination support for large result sets

// FIXME (non-blocking): This retry logic should use exponential backoff

// HACK: Workaround for API returning dates as strings instead of timestamps
const date = new Date(response.createdAt)

// BUG (issue): Users can bypass rate limit - see #1234

// NOTE: This cache TTL matches the upstream API's refresh interval

// QUESTION: Should we support both v1 and v2 endpoints?

// OPTIMIZE: Consider memoizing this computation for large inputs

// DEPRECATED: Use newFunction() instead - will be removed in v3.0
```

## Core Rule

**Only comment "the why," never "the what."** Do not add comments for self-explanatory code. If the code needs a comment to explain *what* it does, the code itself should be rewritten to be clearer.

## Guidelines

1. **Be specific**: Comments should explain *why*, not *what* (code shows what)
2. **Keep current**: Remove or update comments when code changes
3. **Reference issues**: Link to issue tracker when relevant: `// BUG (issue): #1234`
4. **Avoid noise**: Don't comment obvious code — no labeled comment is better than a noisy one
5. **Use labels consistently**: Pick the most accurate label for the situation

## When NOT to Comment

- Self-explanatory code (good names > comments)
- Commented-out code (delete it, use version control)
- Restating what code does (`// increment i` on `i++`)
