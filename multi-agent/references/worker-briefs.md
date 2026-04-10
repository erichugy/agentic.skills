# Worker Brief Template

Use this template when constructing prompts for each worker subagent.

## Template (Independent Worker — Wave 1)

For workers with no dependencies, branching off the base branch:

```
## Context

You are one of {N} workers exploring different approaches to this task:

> {task description}

Base branch: `{base_branch}`. Your worktree has its own branch automatically.

## Your Direction

**Approach: {variant_name}**

{2-4 sentences describing the specific direction, aesthetic, technical approach, or
architectural strategy this worker should follow. Be concrete and specific.}

## Constraints

- Preserve all existing functionality — nothing should break
- {additional constraints from the user, e.g., "don't add new dependencies"}
- {files/directories to avoid modifying}

## Deliverables

{Specific list of files to create or modify, or a description of the expected output}

Ensure changes are complete and self-contained. Another developer should be able to
check out your branch and see a fully working result.

## Before Committing — Self-Review Checklist

Run through this checklist before committing. Fix any issues BEFORE the reviewer sees your code.

- Input validation: `.trim()` before `.min()`, max lengths on all strings, `safeParse` for user-facing errors, `response.json()` in try/catch
- Network: All outbound fetch calls have AbortController timeouts, `clearTimeout` in `finally` blocks
- Resource lifecycle: `setInterval` uses `.unref()`, in-memory stores have hard size caps with eviction
- Race conditions: Double-submit guards, `isMountedRef` or stale-request detection on ALL state updates
- Config: Env var parsing with try/except + fallbacks, numeric values clamped to sane ranges
- Error handling: User-friendly messages (not raw ZodError), failed ops logged with context, no silent catch blocks
- Consistency: UTC everywhere, `||` not `??` for empty-string fallbacks, `export const dynamic = "force-dynamic"` on stateful API routes
- Conventions: `type` over `interface`, no `as` casts, explicit return types, import ordering, only "why" comments

## When Finished

1. Stage and commit all changes using conventional commits format: `<type>(<scope>): <description>` (e.g., `refactor(web): extract API route for contact form`). Do NOT use `multi-agent` as the commit type.
2. Do NOT create a pull request
3. Do NOT push to remote
4. Do NOT merge into any branch
```

## Template (Dependent Worker — Wave 2+)

For workers that build on a prerequisite worker's output:

```
## Context

You are a worker building on top of a previous worker's output.

> {task description}

## Prerequisites

A previous worker ({prerequisite_variant_name}) has already completed work on branch
`{prerequisite_branch}`. You are building on top of their changes.

**Before starting any work**, run:
git fetch origin && git checkout -b {your_branch} {prerequisite_branch}

This ensures your work starts from where the prerequisite left off.

## What Was Done Before You

{Brief summary of what the prerequisite worker built — key files modified, APIs created,
patterns established. Be specific enough that this worker understands the foundation.}

## Your Direction

**Approach: {variant_name}**

{2-4 sentences describing what this worker should build on top of the prerequisite's
work. Reference specific files, APIs, or components from the prerequisite.}

## Constraints

- Preserve all changes from the prerequisite — do not revert or modify their work unless necessary
- {additional constraints from the user}
- {files/directories to avoid modifying}

## Deliverables

{Specific list of files to create or modify, or a description of the expected output}

Ensure changes are complete and self-contained. The branch should include both the
prerequisite's changes and your new work.

## When Finished

## Before Committing — Self-Review Checklist

Run through this checklist before committing. Fix any issues BEFORE the reviewer sees your code.

- Input validation: `.trim()` before `.min()`, max lengths on all strings, `safeParse` for user-facing errors, `response.json()` in try/catch
- Network: All outbound fetch calls have AbortController timeouts, `clearTimeout` in `finally` blocks
- Resource lifecycle: `setInterval` uses `.unref()`, in-memory stores have hard size caps with eviction
- Race conditions: Double-submit guards, `isMountedRef` or stale-request detection on ALL state updates
- Config: Env var parsing with try/except + fallbacks, numeric values clamped to sane ranges
- Error handling: User-friendly messages (not raw ZodError), failed ops logged with context, no silent catch blocks
- Consistency: UTC everywhere, `||` not `??` for empty-string fallbacks, `export const dynamic = "force-dynamic"` on stateful API routes
- Conventions: `type` over `interface`, no `as` casts, explicit return types, import ordering, only "why" comments

## When Finished

1. Stage and commit all changes using conventional commits format: `<type>(<scope>): <description>` (e.g., `refactor(web): extract API route for contact form`). Do NOT use `multi-agent` as the commit type.
2. Do NOT create a pull request
3. Do NOT push to remote
4. Do NOT merge into any branch
```

## Tips for Effective Briefs

- **Be specific about direction** — "use Inter font, 4px border-radius, blue-gray palette" beats "make it modern"
- **Include concrete examples** — reference real websites, design systems, or code patterns
- **Specify what NOT to do** — "don't change the navigation structure" prevents scope creep
- **Keep constraints identical** across all workers so results are directly comparable
- **Include file paths** — tell the worker exactly which files matter most
- **Set scope boundaries** — "focus on the home page only" prevents workers from doing too much or too little
