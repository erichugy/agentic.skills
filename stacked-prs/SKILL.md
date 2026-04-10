---
name: stacked-prs
description: Create and manage stacked pull requests. Use when work should be split into dependent PRs, when a branch should target another feature branch instead of main, or when the user says "stacked PRs", "stacked diffs", "branch 2 into branch 1", or wants smaller reviewable increments.
---

# Stacked PRs

Use this workflow when one feature is easier to review as a sequence of dependent pull requests instead of one large branch.

## Core Model

Stacked PRs are just normal Git branches and PRs with non-`main` base branches.

Example:

- `branch-1` targets `main`
- `branch-2` is created from `branch-1` and its PR targets `branch-1`
- `branch-3` is created from `branch-2` and its PR targets `branch-2`

Merge from the bottom of the stack upward.

## When To Use

Use stacked PRs when:

- one feature has clear incremental layers
- reviewers would benefit from smaller diffs
- later work depends on earlier refactors
- you want each PR to have one main concern
- one branch currently mixes behavior changes with structural moves, renames, or cleanup that could be reviewed separately

Do not use them when:

- the work is small enough for one PR
- the branches would be tightly entangled and impossible to review independently
- each layer is not buildable or reviewable on its own

## Workflow

### 1. Split the work first

Before creating branches, define the stack:

- PR 1: structural groundwork or refactor layer
- PR 2: behavior layer that builds on the groundwork
- PR 3: cleanup or follow-through layer if needed

Each PR should have one clear purpose and be mergeable on its own.

When the work started as one broad branch, actively look for this decomposition:

- behavior PR: user-facing or system-behavior change
- structural PR: renames, moves, extracted modules, dependency inversion, boundary cleanup
- cleanup PR: dead code removal, naming polish, follow-up simplification

### 2. Create the first branch from the real base

Usually:

- branch 1 from `main`

Open PR 1 against `main`.

### 3. Create each next branch from the previous branch

Example:

- branch 2 from branch 1
- branch 3 from branch 2

Open each PR against the branch directly below it in the stack.

### 4. Keep the PRs explicitly labeled

Use a stack marker in the PR title or body:

- `[1/4]`
- `[2/4]`
- `[3/4]`

State the base branch clearly in the PR body.

### 5. Merge from the bottom up

After PR 1 merges:

- GitHub can automatically retarget PR 2 to PR 1's base branch if the merged branch is deleted
- otherwise, change the base branch manually

Then merge PR 2, then PR 3, and so on.

## Rules

- Keep each PR focused on one concern.
- Do not mix behavior changes with broad renames, moves, or architectural cleanup unless they are inseparable.
- Do not mix schema, refactor, orchestration, and persistence changes in the same PR unless they are inseparable.
- Keep each PR buildable and testable.
- Rebase or merge carefully; do not rewrite the stack casually once reviews are active.
- Review comments on higher PRs may go stale when the base changes. Expect this.
- If a later PR exists only because the first PR was too broad, consider whether the stack should have existed from the start.

## GitHub Support

GitHub supports the mechanics needed for stacked PRs, but not as a special first-class “stacked PR” feature.

What GitHub natively supports:

- opening a PR against any branch
- changing the base branch of an open PR
- automatically retargeting open PRs whose base branch was merged and deleted

What GitHub does not give you natively:

- a dedicated stack UI
- stack-wide review tools
- explicit “depends on PR X” workflow semantics

## Quick Review Checklist

Before opening each PR, verify:

- the PR has one main concern
- the diff is reviewable by itself
- the target/base branch is correct
- the PR description explains where it sits in the stack
- validation has been run for that layer
- structural churn is separated from behavior whenever practical
