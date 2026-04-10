---
name: react
description: React and Next.js conventions and performance optimization. Apply when writing, reviewing, or refactoring React components, hooks, Next.js pages, or JSX. Covers performance rules, server components, accessibility, and UI patterns.
---

# React & Next.js Conventions

Apply these when writing, reviewing, or refactoring React/Next.js code.

**Always check for project-specific overrides first** — look for `AGENTS.md` or `CLAUDE.md` in the repo. Project rules override these defaults.

For the full 45-rule Vercel performance guide with code examples, see `/react-best-practices`.

## Component Structure

```typescript
// 1. Imports (grouped: external → parent → sibling)
import { useState } from 'react'
import type { FC, ReactNode } from 'react'

// 2. Types
type ProfileCardProps = {
  user: User
  onEdit: (id: string) => void
}

// 3. Component (function keyword, explicit props type)
export function ProfileCard({ user, onEdit }: ProfileCardProps): ReactNode {
  // hooks first
  const [isEditing, setIsEditing] = useState(false)

  // handlers
  function handleEdit() { ... }

  // render
  return ( ... )
}
```

- Use `function` keyword for components (not arrow functions)
- Explicit `Props` type (not inline)
- Explicit `import type React from 'react'` when using `React.*` types

## Performance — Priority Order

### Critical: Eliminate Waterfalls
- Move `await` into branches where actually used
- Use `Promise.all()` for independent operations
- Use Suspense boundaries to stream content

### Critical: Bundle Size
- Import directly — avoid barrel files (`index.ts` re-exports)
- Use `next/dynamic` for heavy components
- Defer third-party scripts (analytics, logging) until after hydration
- Preload on hover/focus for perceived speed

### High: Server-Side
- Use `React.cache()` for per-request deduplication
- Minimize data serialized to client components
- Restructure components to parallelize fetches
- Use `after()` for non-blocking operations

### Medium: Re-renders
- Don't subscribe to state only used in callbacks
- Extract expensive work into memoized components
- Use primitive dependencies in effects (not objects)
- Use `startTransition` for non-urgent updates
- Use functional `setState` for stable callbacks

## Hooks Rules

- Complete `useCallback`/`useMemo`/`useEffect` dependency arrays — never lie about deps
- Always return cleanup from `useEffect` when adding listeners or timers
- No state updates in render (causes infinite loops)
- Guard state updates after unmount (`isMountedRef` or stale-request detection)

## Event Listeners & Cleanup

```typescript
useEffect(() => {
  const handler = (e: MouseEvent) => { ... }
  document.addEventListener('click', handler)
  return () => document.removeEventListener('click', handler)
}, [])
```

- Every `addEventListener` must have a matching `removeEventListener` in cleanup
- Drag handlers (`mousedown`→`mousemove`→`mouseup`) clean up on unmount via ref
- Cursor/userSelect overrides on `document.body` restored in ALL exit paths

## Keys

- Keys must be stable and unique — never use array indices when items can reorder
- Derive keys from data IDs, not from iteration position

## Accessibility

- `aria-expanded` on collapsible/toggle elements
- `type="button"` on all non-submit buttons
- `aria-label` on icon-only buttons
- Keyboard navigable: Enter to activate, Escape to close menus
- Focus management: return focus to trigger after closing modals/dropdowns

## CSS

- Follow the component's class prefix convention (e.g., `rb-`)
- Consistent active states across all toggles/tabs/buttons
- Smooth transitions with consistent duration
- No `!important` unless absolutely necessary

## Dropdowns & Menus

- Close on outside click
- Handle viewport overflow (`getBoundingClientRect()` to detect, flip direction)
- Sub-menus don't escape viewport edges
- Nested menus don't interfere with parent menus

## localStorage

- All reads wrapped in try/catch (private browsing can throw)
- Parsed data deeply validated before use (not just top-level fields)
- Missing/corrupt values fall back to defaults gracefully

## Conditional Rendering

- Use ternary for simple conditions, `if/else` or `switch` for multiple branches
- Never use nested ternaries in JSX
- Use `&&` carefully — `{count && <Component />}` renders `0` when count is 0. Use `{count > 0 && <Component />}`
