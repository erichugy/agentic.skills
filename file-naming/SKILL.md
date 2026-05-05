---
name: file-naming
description: File and folder naming conventions. Apply when creating files/folders or refactoring structure. Prefer concise descriptive names, use folder context, avoid vague catch-all files, keep small project-agnostic helpers in utils/, and colocate feature-specific code. Read the React reference when organizing pages or component groups. For symbol naming (functions, types, predicates, variants, domain constants), see /naming.
---

# File & Folder Naming Conventions

Use this skill when naming or reorganizing files and folders. For symbol naming (functions, types, predicates, variants, domain constants), see `/naming`.

## Reference Loading Rule

This skill is intentionally split into a short core guide plus focused references.

Agents should read the relevant reference when the task touches that area. Treat the references as the detailed evidence and further-reading material for this skill, not as optional background.

Only skip a reference when you are completely sure it is unnecessary for the current task.

## Core Principle

> Prefer the shortest name that is still clear in context.

The goal is not maximum precision in every filename. The goal is clear, scannable names with obvious boundaries.

## Quick Rules

- Let the folder provide the domain; let the file provide the role
- Prefer concise descriptive names over bloated names that repeat folder context
- Avoid vague catch-all files such as `utils.ts`, `helpers.ts`, `common.ts`, or `misc.ts`
- `utils/` is for small, project-agnostic helpers only; feature-specific code stays with the feature
- `index.ts` is only for barrels, never implementation
- Avoid `foo/foo.ts` when the file adds no role beyond the folder name. If `foo/` already provides the concept, prefer a role file such as `types.ts`, `schema.ts`, `create.ts`, or `parse.ts`
- Feature-local `types.ts` is good when it clearly owns the contracts for one concept folder. The thing to avoid is a broad cross-feature `types/` bucket that mixes unrelated ownership
- If consumers are meant to import a concept directly, prefer a real file or barrel at that path. Do not bury first-class public concepts under extra nesting like `types/rows` unless that nesting is itself part of the intended API
- Use the existing project casing convention unless there is a strong reason to normalize it

## Naming Heuristic

1. Reject vague bucket names
2. Reject names that repeat obvious folder context
3. Prefer a filename that states the file's role. In `user/`, choose `types.ts` for contracts or `create.ts` for behavior, not `user.ts`
4. If code belongs to one feature, route, or domain, colocate it there
5. If code is small and genuinely project-agnostic, `utils/` is acceptable

## Examples

```text
# GOOD — concise and contextual
home/
  hero.tsx

requests/
  page-client.tsx
  components/
    filter-bar.tsx
    detail-panel.tsx

utils/
  datetime.ts
  io.ts

# BAD — vague
utils.ts
thing.tsx
shared.ts

# BAD — overly verbose
home-page-hero-section-component.tsx
request-bin-filter-toolbar-component.tsx
```

## React Guidance

When working with React routes, pages, or component groups, read:

- [references/react-component-organization.md](references/react-component-organization.md)

Read this reference by default for React page and component organization work unless you are completely sure the core rules already cover the decision.

Use that reference for:

- page composition
- route-local component placement
- grouping related components into folders
- naming the top-level composed component inside a component folder

## Utils Guidance

When deciding whether code belongs in `utils/`, read:

- [references/generic-utils.md](references/generic-utils.md)

Read this reference by default when placing helpers or deciding between `utils/` and feature-local code unless you are completely sure the boundary is obvious.

## Review Checklist

- [ ] Can you infer the purpose from the filename plus its parent folder?
- [ ] Is the name concise without becoming vague?
- [ ] Does the file avoid repeating the folder's domain unnecessarily?
- [ ] If the folder already supplies the concept, does the filename still communicate a real role instead of restating the concept?
- [ ] Does the folder structure match the intended import surface, or are consumers immediately reaching around it?
- [ ] If the code is feature-specific, is it colocated with that feature?
- [ ] If the code is in `utils/`, is it genuinely small and project-agnostic?
- [ ] Is any `index.ts` file acting purely as a barrel?
