---
name: file-naming
description: File and folder naming conventions. Apply when creating files/folders or refactoring project structure. Names must reveal purpose — no vague names like utils.ts or helpers.ts.
---

# File & Folder Naming Conventions

Every file and folder name must make its purpose immediately obvious. A developer should be able to understand what a file does without opening it.

## Core Principle

> If you have to open a file to know what it does, the name is wrong.

## Rules

### 1. Names Must Be Specific

```
# BAD — vague, could contain anything
utils.ts
helpers.ts
common.ts
misc.ts
index.ts (when it's not a barrel file)
types.ts (when it contains types for multiple unrelated things)
constants.ts (when it mixes unrelated constants)

# GOOD — specific, self-documenting
date-formatting.ts
string-validation.ts
api-error-handling.ts
```

### 2. Two Valid Structures for Domain + Action

When a file involves both a domain (what) and an action (how), choose one of these structures:

```
# Option A: Domain folder, action file
exports/
  tables.ts
  charts.ts

# Option B: Action folder, domain file
tables/
  export.ts
  import.ts

# Choose based on what varies more:
# - If the domain is fixed and actions vary → Option B (tables/)
# - If the action is fixed and domains vary → Option A (exports/)
```

### 3. Folder Carries the Domain, File Carries the Role

Once a folder establishes the domain, filenames should stop repeating that domain.

```
# GOOD — folder provides the domain context
runtime/
  defaults.ts
  load-from-env.ts
  apply-cli-overrides.ts

health-check/
  run.ts
  retry.ts
  result.ts
  wait-for-verdict.ts

# BAD — repeated domain in every filename
runtime/
  runtime-config.ts
  runtime-config-defaults.ts

health-check/
  run-health-check-audit.ts
  retry-health-check-audit.ts
  health-check-result.ts
  wait-for-health-check-verdict.ts
```

Short names like `run.ts` or `result.ts` are acceptable only when the parent folder fully provides the missing context.

### 4. Folder Names = Category, File Names = Specific Thing

```
# GOOD
handlers/
  user-authentication.ts
  order-processing.ts
  payment-webhook.ts

models/
  user-profile.ts
  order-line-item.ts

# BAD — folder repeats what files already say
user-handlers/
  handler.ts      ← redundant, what handler?
```

### 5. Utility Buckets Become Folders, Not Files

Avoid `utils.ts`, `helpers.ts`, or similar grab-bag files, especially inside an already-scoped folder.

```
# BAD
cli/
  utils.ts

# GOOD
cli/
  parse-options.ts
  select-targets.ts
  print-usage.ts
  map-with-concurrency.ts
```

If a `utils.ts` file would hold more than one distinct responsibility, split it into purpose-named files or a subfolder.

### 6. File Casing

| Convention | When | Example |
|-----------|------|---------|
| `kebab-case` | Default for all files | `user-profile.ts` |
| `PascalCase` | React components (if project convention) | `UserProfile.tsx` |
| `snake_case` | Rust, Python files | `user_profile.rs` |

Follow the existing project convention. If mixed, standardize to the dominant pattern.

### 7. No Abbreviations (Unless Universal)

```
# BAD
auth-mgr.ts
usr-svc.ts
btn-grp.tsx

# GOOD
auth-manager.ts
user-service.ts
button-group.tsx

# OK — universally understood abbreviations
api.ts
db.ts
url-parser.ts
```

### 8. Test Files Mirror Source Files

```
src/
  user-profile.ts
  order-processing.ts

tests/ (or __tests__/)
  user-profile.test.ts
  order-processing.test.ts
```

### 9. Module Folders Over Flat Files

When a concept has both types and implementation (or multiple related files), prefer a module folder with `types.ts` + role files + `index.ts` barrel over flat files at the parent level.

```
# BAD — flat files at the parent level
orchestrators/
  health-check-contracts.ts
  health-checks.ts

# GOOD — module folder with clear roles
orchestrators/
  health-checks/
    index.ts          # barrel
    types.ts          # contracts, type definitions
    run.ts            # implementation

# BAD — type file and strategy files as siblings
persistence/
  sinks.ts            # type + re-exports
  local-file-sink.ts
  bp-bot-sink.ts

# GOOD — type file with strategies subfolder
persistence/
  sinks.ts            # type + re-exports from strategies
  strategies/
    local/
      sink.ts
      source.ts
    bp-bot/
      sink.ts
      source.ts
```

This pattern scales: add a new strategy by adding a file in `strategies/`, not by creating a new sibling at the parent level.

### 10. Consumer-Facing Module Folders Need A Barrel

If a folder represents one thing that consumers import as a single module, add an `index.ts` barrel so the import surface stays stable whether that module is implemented as:

- one file today
- a folder with multiple internal files later
- or collapsed back to one file later

This keeps structure changes internal to the module instead of forcing consumer import churn.

```text
# GOOD — consumer imports from one stable module path
config/
  index.ts
  bootstrap.ts
  limits.ts

# Also GOOD later — same consumer-facing path can collapse back to one file
config.ts
```

Use this when the folder is effectively one public module. Do not add a barrel just for private internal subfolders that are not intended as a stable import surface.

### 11. Index Files Are Only Barrel Files

`index.ts` should only re-export from sibling files. It should never contain implementation logic.

```typescript
// index.ts — GOOD: barrel file
export { UserProfile } from './user-profile'
export { OrderSummary } from './order-summary'

// index.ts — BAD: implementation in index
export function calculateTotal() { ... }  // Put this in calculate-total.ts
```

### 12. Config Files Are Scoped

```
# BAD
config.ts          ← config for what?

# GOOD
database-config.ts
auth-config.ts
feature-flags.ts
```

### 13. Planning Docs Do Not Override Bad Names

If an implementation plan, PR comment, or user sketch proposes vague or redundant names, flag that before scaffolding. Do not blindly copy a weak name into the codebase just because it appeared in a plan.

```
# If the plan says this:
cli/
  utils.ts

# Propose this instead:
cli/
  parse-options.ts
  select-targets.ts
```

## Refactoring Checklist

When reviewing file structure, flag:
- [ ] Any file named `utils`, `helpers`, `common`, `misc`, or `shared` that mixes concerns
- [ ] Any `index.ts` that contains implementation (not just re-exports)
- [ ] Any consumer-facing module folder that should expose a stable import path but has no `index.ts` barrel
- [ ] Any file where you can't tell the purpose from the name alone
- [ ] Any abbreviation that isn't universally understood
- [ ] Any folder that repeats information already in the filename
- [ ] Any filename inside a domain folder that repeats the folder's domain noun
- [ ] Any plan or instructions doc that prescribes vague or redundant names without justification
