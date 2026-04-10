---
name: code-styles
description: Index of all coding style skills. Read this first to determine which style skills to assign to agents. Use when planning work, assigning skills to workers, or reviewing code across any language or framework.
---

# Code Styles — Skill Index

This is the **master index** of all coding style skills. Planners and orchestrators should read this file to determine which skills to assign to each agent based on the task at hand.

This file stays an **index**, not a duplicate ruleset. Cross-language design defaults and the "greenfield vs existing repo" operating mode live in `/coding-conventions`. Language-specific expression of those rules lives in the language skills such as `/typescript`.

## How to Use This Index

1. Identify the **languages** and **frameworks** involved in the task
2. Identify which **design patterns** are relevant
3. Identify **structural concerns** (file naming, comments, general conventions)
4. Assign the matching skills to each agent in the plan

## Language Skills

| Skill | Applies When | Key Focus |
|-------|-------------|-----------|
| `/typescript` | Any `.ts`, `.tsx`, `.js`, `.jsx` code | Strict types, Zod validation, no `as` casts, naming, imports |
| `/rust` | Any `.rs` code | Ownership, Result/Option, clippy, module organization |

## Framework Skills

| Skill | Applies When | Key Focus |
|-------|-------------|-----------|
| `/react` | React components, Next.js pages, hooks, JSX | Performance (waterfalls, bundle size, re-renders), server components, accessibility |

## Design Pattern Skills

| Skill | Applies When | Key Focus |
|-------|-------------|-----------|
| `/design-patterns` | Index of all patterns — read to pick specific ones | Overview and when-to-use for each pattern |
| `/solid` | Any OOP or module design | Single responsibility, open/closed, Liskov, interface segregation, dependency inversion |
| `/composition` | Class hierarchies, shared behavior, code reuse decisions | Favor composition over inheritance, mixins, HOCs, hooks |
| `/strategy-pattern` | Swappable algorithms, conditional behavior, plugin systems | Encapsulate algorithms, runtime selection, eliminate switch/if chains |
| `/factory-pattern` | Object creation, polymorphism, config-driven instantiation | Centralize creation logic, decouple consumers from concrete types |
| `/observer-pattern` | Event systems, pub/sub, reactive state, notifications | Decouple producers from consumers, event emitters, subscriptions |

## Structure & Style Skills

| Skill | Applies When | Key Focus |
|-------|-------------|-----------|
| `/file-naming` | Creating files/folders, refactoring structure | Names must reveal purpose — `exports/tables.ts` not `utils.ts` |
| `/coding-conventions` | All code — language-agnostic core rules | Error handling, input validation, resource lifecycle, race conditions |
| `/code-comments` | Writing or reviewing comments | Conventional Comments, "only the why", labels (TODO, FIXME, NOTE) |
| `/simplify` | After writing code | Reduce complexity, eliminate redundancy, improve clarity |

## Skill Assignment Guide for Planners

### By task type

| Task Type | Required Skills | Optional Skills |
|-----------|----------------|-----------------|
| TypeScript feature | `/typescript`, `/coding-conventions`, `/file-naming` | `/solid`, `/composition`, design patterns |
| React UI work | `/react`, `/typescript`, `/coding-conventions` | `/file-naming` |
| Rust feature | `/rust`, `/coding-conventions`, `/file-naming` | `/solid`, design patterns |
| API endpoint | `/typescript`, `/coding-conventions` | `/strategy-pattern`, `/factory-pattern` |
| Refactor | `/solid`, `/composition`, `/coding-conventions`, `/simplify` | `/file-naming`, design patterns |
| Code review | `/coding-conventions`, `/code-comments`, language skill, framework skill | Design patterns as relevant |

### By agent role

| Agent Role | Skills to Assign |
|------------|-----------------|
| **Worker/Implementer** | Language skill + framework skill + `/coding-conventions` + `/file-naming` + relevant design patterns |
| **Reviewer** | Language skill + framework skill + `/coding-conventions` + `/code-comments` + `/simplify` |
| **Security Reviewer** | `/coding-conventions` (input validation, error handling sections) |
| **Style Enforcer** | Language skill + `/coding-conventions` + `/code-comments` + `/file-naming` + `/simplify` |
| **Architect/Planner** | `/design-patterns` + `/solid` + `/composition` + `/file-naming` |
