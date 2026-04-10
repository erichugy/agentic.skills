---
name: plan
description: Planning skill for architects and orchestrators. Use before any non-trivial task to create a precise execution plan that assigns specific skills to each agent. Triggers on "plan", "design", "architect", or when /workflow or /multi-agent needs to determine agent assignments.
---

# Plan — Precision Agent Planning

Create execution plans that assign **specific skills** to each agent. Every agent in the plan must know exactly which skills to leverage and why.

## When to Use

- Before `/workflow` Phase 3 (execution) — the planner determines skill assignments for each worker and reviewer
- Before `/multi-agent` Phase 2 (dispatch) — the planner writes precise worker briefs with skill lists
- Any time you need to break work into agents and want optimal skill coverage

## Process

### Step 1: Analyze the Task

1. **Read the task requirements** — what needs to be built/changed/fixed
2. **Identify languages** — which `.ts`, `.rs`, `.py`, `.go` files are involved?
3. **Identify frameworks** — React, Next.js, Express, Actix, etc.?
4. **Identify architectural concerns** — new modules, refactoring, event systems, etc.?
5. **Identify structural concerns** — new files/folders being created?

### Step 2: Read the Style Index

**MANDATORY**: Read `/code-styles` to understand available style skills and the assignment guide. This index maps task types and agent roles to specific skills.

### Step 3: Determine Agent Roles

For each agent in the plan, determine:
- **Role**: Worker, Reviewer, Security Reviewer, Style Enforcer, Researcher
- **Task scope**: What specific files/features this agent owns
- **Skills**: The exact list of skills this agent should leverage

### Step 4: Assign Skills Per Agent

Use the `/code-styles` assignment guide, but also apply judgment:

| Agent Role | Always Assign | Conditionally Assign |
|------------|--------------|---------------------|
| **Worker** | Language skill, `/coding-conventions`, `/file-naming` | Framework skill, design pattern skills (based on task) |
| **Reviewer** | Language skill, `/coding-conventions`, `/code-comments`, `/simplify` | Framework skill, `/solid` (for architecture reviews) |
| **Security Reviewer** | `/coding-conventions` (validation/error sections) | Language skill for language-specific security patterns |
| **Style Enforcer** | Language skill, `/coding-conventions`, `/code-comments`, `/file-naming`, `/simplify` | Framework skill |
| **Architect** | `/design-patterns`, `/solid`, `/composition`, `/file-naming` | Language/framework skills |

### Step 5: Write the Plan

Output format:

```markdown
## Execution Plan: <task name>

### Analysis
- Languages: <list>
- Frameworks: <list>
- Architectural patterns needed: <list>
- New files/folders: <yes/no — if yes, /file-naming applies>

### Agents

#### Wave 1

**Agent 1: <name> (<role>)**
- Task: <specific description>
- Skills: `/typescript`, `/react`, `/coding-conventions`, `/file-naming`
- Rationale: <why these skills — e.g., "Creating new React components with new file structure">

**Agent 2: <name> (<role>)**
- Task: <specific description>
- Skills: `/typescript`, `/coding-conventions`, `/strategy-pattern`
- Rationale: <why these skills — e.g., "Refactoring handler to use strategy pattern for export formats">

#### Wave 1 — Reviewers

**Agent 3: <name> Reviewer**
- Reviews: Agent 1
- Skills: `/typescript`, `/react`, `/coding-conventions`, `/code-comments`, `/simplify`

**Agent 4: <name> Reviewer**
- Reviews: Agent 2
- Skills: `/typescript`, `/coding-conventions`, `/code-comments`, `/solid`, `/simplify`

#### Wave 2 (depends on Wave 1)
...
```

## Skill Assignment Rules

### Be Specific, Not Exhaustive

Assign only skills that are **relevant** to the agent's task. Don't give every agent every skill — that dilutes focus.

```
# BAD — kitchen sink
Skills: /typescript, /rust, /react, /solid, /composition, /strategy-pattern,
        /factory-pattern, /observer-pattern, /file-naming, /coding-conventions,
        /code-comments, /simplify

# GOOD — targeted
Skills: /typescript, /react, /coding-conventions, /file-naming
Rationale: Building new React components in TypeScript, creating new file structure
```

### Match Patterns to Problems

Only assign design pattern skills when there's a **concrete signal**:

| Signal in the Code | Pattern to Assign |
|--------------------|------------------|
| Growing switch/if chain on a type | `/strategy-pattern` |
| Complex object construction | `/factory-pattern` |
| Events, notifications, reactive updates | `/observer-pattern` |
| Deep inheritance or "extends" chains | `/composition` |
| Module doing too many things | `/solid` |
| General architecture review | `/design-patterns` (the index) |

### Reviewers Get Style Skills, Workers Get Implementation Skills

- Workers focus on: language, framework, design patterns, `/coding-conventions`, `/file-naming`
- Reviewers focus on: language, framework, `/coding-conventions`, `/code-comments`, `/simplify`, `/solid`

### Always Include Language Skill

Every agent that touches code gets the appropriate language skill. No exceptions.
