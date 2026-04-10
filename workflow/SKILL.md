---
name: workflow
description: Eric's preferred AI-assisted development workflow. Use when starting any non-trivial coding task, feature implementation, or project work. Triggers on new features, tasks, implementations, or when the user says "let's build", "implement", "create", "add feature". This skill defines the end-to-end process from instructions to PRs.
---

# Workflow — AI-Assisted Development

Eric's preferred workflow for AI-assisted development. Follow this process for any non-trivial task.

## Phase 1: Instructions Document

Before writing any code, create an instructions document.

### Location
1. If the repo has a `.ignore.me/` folder, put it there
2. If not, create it at `~/Desktop/repos/ai-instructions/`

### Self-Contained Principle

**The instructions document must be fully self-contained.** A worker agent with zero prior context should be able to read it and complete the task without exploring the codebase.

Before writing the instructions, the agent creating them must:
1. **Explore the codebase** — read relevant files, understand existing patterns, find reusable utilities
2. **Gather all context** — current source code of files to modify, design system tokens, type definitions, data structures, API contracts
3. **Include only what's relevant** — don't dump the entire codebase, but include enough that a fresh agent can work autonomously

What to embed in the instructions:
- **Current source code** of files being modified (full or key sections)
- **Type definitions** and interfaces the worker needs
- **Design system tokens** (colors, spacing, component patterns) if doing UI work
- **API contracts** if integrating with endpoints
- **Existing patterns** to follow (with file paths and code examples)
- **Monorepo structure** if relevant (which app, where files live)
- **Naming review** for any new files/folders — apply `/file-naming` to the proposed layout and call out vague names (`utils.ts`, `helpers.ts`) or redundant domain repetition (`health-check/run-health-check-audit.ts`) before implementation

### Format
- **Lightweight top-level file** — objective, tasks, constraints, execution plan
- If more detail is needed, use a **composed structure**: top-level file points to sub-files with details (see example below)
- Name: `<feature-name>-instructions.md` (e.g., `contact-form-instructions.md`)

### Composed Structure Example
```
.ignore.me/
├── feature-name-instructions.md    ← lightweight overview (with embedded context)
└── feature-name/
    ├── task-1.md                   ← detailed task spec (with relevant source code)
    ├── task-2.md
    └── knowledge-base.md           ← reference material, existing code, type defs
```

### Required Sections
- **Objective** — what and why
- **Base Branch** — which branch to branch off of
- **Monorepo/Project Structure** — where the app lives, key directories
- **Current State** — actual source code of files to modify (not just file paths)
- **Tasks** — numbered, with file paths, clear deliverables, and enough context to execute
- **Design/Constraints** — design system tokens, patterns to follow, things to avoid
- **Naming Decisions** — proposed file and folder names for new structure, with any deviations from `/file-naming` called out explicitly
- **Multi-Agent Setup** — workers, reviewers, waves, branch names, and the specific skills each individual agent should leverage (use `/plan` and `/code-styles` to determine these)

### Wait for Approval
Present the instructions to Eric. **Do NOT proceed until he says OK.**

If a user-supplied plan or prior instructions contain weak naming, surface that before approval instead of copying the names through unchanged.

## Phase 2: Branch & Worktree Setup

After approval:

1. Create a branch: `eh/<type>/<name>` (e.g., `eh/feat/contact-form`, `eh/fix/scroll-bug`)
2. Create a worktree for the agent(s) to work in
3. Branch types: `feat`, `fix`, `chore`, `refactor`

## Phase 3: Execution with Multi-Agents

Use `/multi-agent` for all non-trivial work. Even single-task work benefits from a reviewer.

**Before dispatching agents**, use `/plan` to create the execution plan:
1. Read `/code-styles` to understand the full skill hierarchy
2. Identify languages and frameworks in the task
3. Assign **specific skills per agent** based on their role and the code they'll touch

### Worker Rules
- Each worker gets its own worktree (via `isolation: "worktree"`)
- Workers commit but do NOT push or create PRs
- Commit format: conventional commits — `<type>(<scope>): <description>` (e.g., `refactor(web): extract API route`). Do NOT use `multi-agent` as the type.
- **Workers must write production-quality code on the first pass.** The worker brief must include the full Production Hardening Checklist from `references/review-rubric.md` so workers self-check before committing.
- The agent creating the workflow plan must explicitly list the skills for each individual worker. Do not give one shared skill list for all workers unless every worker truly needs the exact same set.
- Different workers may require different skills. Use `/code-styles` to determine the right skill set per worker:
  - Language skills: `/typescript`, `/rust` (based on files being touched)
  - Framework skills: `/react` (for React/Next.js UI work)
  - Pattern skills: `/solid`, `/composition`, `/strategy-pattern`, `/factory-pattern`, `/observer-pattern` (based on architectural needs)
  - Structure skills: `/coding-conventions`, `/file-naming` (always for workers creating new files)
  - Domain skills: `/rams` (for UI accessibility), repo-specific skills

### No-Op Detection (CRITICAL)

Every worker and reviewer must check for no-ops — changes that have no functional effect:
- Workers: after making changes, run `git diff` and verify every hunk has a functional effect. Revert any no-op hunks (whitespace-only, reordering with no effect, adding something already present).
- Reviewers: flag any no-op changes as issues to revert.
- Manager: if all changes in a wave are no-ops after reverting, skip the commit and report it.
- **Never commit a no-op.** A commit with zero functional changes wastes review cycles and pollutes git history.

### Reviewer Rules — Review Loop
- Every worker gets a dedicated reviewer
- Reviewers MUST run through the **entire** Production Hardening Checklist (from `references/review-rubric.md`) — every item, every file
- Reviewers apply style skills matching their worker: language skill + `/coding-conventions` + `/code-comments` + `/simplify` + framework skill (if applicable)
- Reviewers apply `/rams` for UI work, verify light/dark mode and responsiveness
- Reviewers flag no-op changes as issues to revert
- **Quality bar**: The reviewer's goal is to catch everything an automated reviewer (GitHub Copilot) would flag. If the reviewer thinks Copilot would comment on something, the reviewer must flag it first.

### Review Loop (MANDATORY)
The review process is iterative. It repeats until the reviewer finds no more issues:

```
1. Worker completes code → commits
2. Reviewer reviews against full checklist → outputs Issues to Fix
3. IF issues found:
   a. Manager applies fixes (or sends worker back)
   b. Manager commits fixes
   c. Reviewer reviews AGAIN (go to step 2)
4. IF no issues found (reviewer verdict = PASS with 0 issues):
   → Move to next wave or Phase 4
```

**The loop MUST continue until the reviewer's verdict is PASS with zero Issues to Fix.** A PASS with "minor nits" is not acceptable — fix the nits and re-review.

### Manager Rules
- The manager orchestrates workers and reviewers
- The manager CAN create worktrees, push branches, create PRs between worktree branches, and merge worktree PRs
- The manager MUST NOT merge the final PR back into the original base branch — that is Eric's job
- The manager resolves merge conflicts between worktree branches
- If a reviewer finds issues, the manager fixes them and **sends the code back to the reviewer** — not to Eric

### Wave-Based Dispatch
- Independent tasks run in parallel (same wave)
- Dependent tasks wait for prerequisites (later wave)
- Workers in later waves branch off their prerequisite's branch

## Phase 4: PR Creation

After all workers and reviewers are done (all review loops resolved):

1. Push the branch(es) to remote
2. Create PR(s) targeting the base branch specified in the instructions
3. **Do NOT merge** — present the PR link(s) to Eric for review
4. For multi-branch work: create PRs in dependency order (leaf → root)

## Quick Reference

| Step | Action | Who |
|------|--------|-----|
| 1 | Create instructions doc | Agent |
| 2 | Review instructions | Eric |
| 3 | Create branch + worktree | Agent |
| 4 | Dispatch workers | Agent (manager) |
| 5 | Review loop (review → fix → re-review until clean) | Agent (manager + reviewer) |
| 6 | Push + create PR | Agent (manager) |
| 7 | Merge PR | Eric (always) |
