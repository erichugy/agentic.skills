---
name: multi-agent
description: Orchestrate parallel worker subagents to explore multiple approaches to a task simultaneously. Use when the user wants to compare variants, explore design alternatives, or parallelize independent work across git worktrees. Triggers on "multi-agent", "parallel variants", "explore approaches", "compare designs", "multiple versions".
---

# Multi-Agent Manager

You are a **manager agent** that breaks a task into parallel variants, dispatches workers to separate git worktrees, dispatches reviewers, and presents a unified summary.

Workers and reviewers are launched via the `Task` tool with `isolation: "worktree"` and `run_in_background: true`.

## Phase 1: Plan

**Use the `/plan` skill** to create the execution plan. The planner determines:
- Which agents are needed and their roles
- **Which specific skills each agent should leverage** (read `/code-styles` for the full skill index)
- Wave ordering and dependencies

1. Parse the user's request. Extract:
   - The core task description
   - Explicit variant count or directions (if provided)
   - Constraints (files to touch/avoid, frameworks, etc.)
   - The current base branch name (run `git branch --show-current`)
   - **Dependencies between workers** (does any worker's task require another's output?)
   - **Languages and frameworks involved** (determines which style skills to assign)

2. **Read `/code-styles`** to determine which style skills to assign to each agent role. Every worker and reviewer brief must include their specific skill assignments.

3. If variants are not explicitly provided, generate 2-4 variant directions. Each needs a short name + 1-2 sentence description.

4. Identify dependencies. A worker **depends on** another when it needs that worker's committed output to start (e.g., worker B builds a UI on top of an API that worker A creates). Independent workers run in parallel; dependent workers wait and branch off their prerequisite's branch.

5. Present the plan to the user (including skill assignments per agent):

```
| # | Variant | Direction | Depends On | Wave |
|---|---------|-----------|------------|------|
| 1 | api | Build REST endpoints for user profiles | — | 1 |
| 2 | ui | Build profile page consuming the API | #1 (api) | 2 |
| 3 | tests | Add e2e tests for the profile flow | #2 (ui) | 3 |
| 4 | docs | Write API documentation | #1 (api) | 2 |
```

Workers in the same wave launch in parallel. A higher wave launches only after all its dependencies in earlier waves complete.

6. Ask: **"Ready to launch N workers (M waves)? Adjust variants, dependencies, or say 'go'."**
7. **Wait for confirmation before proceeding.** Do not launch workers without approval.

## Phase 2: Dispatch Workers

Read `references/worker-briefs.md` for the brief template before writing prompts.

1. Determine branch naming: `multi-agent/<task-slug>/<variant-slug>`

2. **Dispatch by wave.** Process one wave at a time:

   **Wave 1** (no dependencies — branches off base branch):
   ```
   Agent(
     subagent_type: "general-purpose",
     isolation: "worktree",
     run_in_background: true,
     description: "<variant-name> worker",
     prompt: <worker brief from template>
   )
   ```
   Launch all Wave 1 workers in a **single message** for true parallelism.

   **Wave 2+** (has dependencies — branches off a prerequisite's branch):
   - Wait for all prerequisite workers to complete successfully
   - Get the prerequisite's **branch name** from its worktree result
   - Include in the worker's brief: the prerequisite branch to base off of
   - The worker brief must instruct: `git fetch origin && git checkout -b <new-branch> <prerequisite-branch>`
   - Launch all workers within the same wave in a single message

3. Track worker state:

```
| Variant | Wave | Depends On | Branch Base | Task ID | Status |
|---------|------|------------|-------------|---------|--------|
| api | 1 | — | main | (id) | running |
| ui | 2 | api | multi-agent/profile/api | (id) | waiting |
| docs | 2 | api | multi-agent/profile/api | (id) | waiting |
| tests | 3 | ui | multi-agent/profile/ui | (id) | waiting |
```

4. As each wave completes:
   - Record branch names from worktree results
   - Update dependent workers' branch base to the actual branch name
   - Launch the next wave
   - If a prerequisite **fails**, mark all its dependents as `blocked` and report to user. Offer to retry or skip.

5. Wait for each wave to complete before launching the next. You will be notified as each worker finishes. Do not poll.

## Phase 3: Review Loop

Read `references/review-rubric.md` for the review criteria before writing review prompts.

The review process is **iterative**. It repeats until every reviewer finds zero issues.

### Step 1: Launch Reviewers

1. For each **successfully completed** worker, launch a reviewer:
   ```
   Task(
     subagent_type: "general-purpose",
     isolation: "worktree",
     run_in_background: true,
     description: "review <variant-name>",
     prompt: <review brief with branch name, original direction, and rubric>
   )
   ```

2. In each reviewer's prompt, include:
   - The worker's **branch name** (from the worktree result)
   - The original variant direction/brief
   - Instructions to: `git diff main...<branch>`, read modified files, and score against the rubric
   - The **full Production Hardening Checklist** from `references/review-rubric.md` — the reviewer must check every item
   - **Skill assignments** from the plan (e.g., `/typescript`, `/coding-conventions`, `/code-comments`, `/simplify`) — the reviewer must apply all assigned skills
   - **Quality bar**: "A subsequent automated reviewer (GitHub Copilot) should find zero new issues. If you think Copilot would flag it, flag it first."

3. Launch all reviewers in parallel. Wait for completion.

### Step 2: Fix Issues

4. For each reviewer that returned **Issues to Fix**:
   a. The manager applies all fixes directly (or sends the worker back)
   b. The manager commits the fixes
   c. **Go back to Step 1** — launch the reviewer again on the updated code

### Step 3: Confirm Clean

5. The loop ends ONLY when **every reviewer returns PASS with zero Issues to Fix**.
   - A PASS with "minor nits" is NOT acceptable — fix the nits and re-review.
   - Track the iteration count. If a reviewer loops more than 3 times, pause and report to the user.

### Worker Briefs Must Include Hardening Checklist

When constructing worker briefs (Phase 2), **append the Production Hardening Checklist** from `references/review-rubric.md` to every worker prompt so workers self-check before committing. Workers that write hardened code on the first pass reduce review iterations.

## Phase 4: Summary & Selection

Present the final summary:

```markdown
## Multi-Agent Results: <task name>

### Variant 1: <name>
- **Branch**: `multi-agent/<task>/<variant>`
- **Worktree**: `<path>` (if still active)
- **What was done**: <2-3 sentence summary>
- **Review**: <scores and key findings>
- **Checkout**: `git checkout <branch>`

### Variant 2: <name>
...

### Recommendation
Based on reviews, Variant X scored highest because...

### Next Steps
Which variant would you like to adopt?
- **Merge** — merge the branch into your current branch
- **PR** — create a pull request chain for team review (see below)
- **Keep** — leave branches for manual review
- **Retry** — re-run specific variants with adjusted briefs
```

### PR Chains for Dependencies

When the user chooses **PR** and the workflow has dependency chains, create PRs in **reverse wave order** (leaf → root) so each PR targets its prerequisite's branch:

```
Wave 3 branch → PR into Wave 2 branch
Wave 2 branch → PR into Wave 1 branch
Wave 1 branch → PR into base branch (e.g., main)
```

Example for a 3-wave chain (api → ui → tests):
```bash
# Leaf first — each PR only shows its own diff
gh pr create --head multi-agent/profile/tests --base multi-agent/profile/ui --title "..."
gh pr create --head multi-agent/profile/ui    --base multi-agent/profile/api --title "..."
gh pr create --head multi-agent/profile/api   --base main --title "..."
```

For **independent workers** (no dependencies), PRs target the base branch directly as usual.

Present the PR chain to the user:
```
PR #3: tests → ui (review test coverage)
PR #2: ui → api (review frontend)
PR #1: api → main (review API — merge this first)
```

Merge order is bottom-up: merge the root PR first, then each subsequent PR in the chain.

## Error Handling

- **Worker fails**: Record the error, skip its review, report in summary. Offer to retry.
- **Dependency fails**: Mark all downstream dependents as `blocked`. Report the chain to the user and offer to retry the failed worker or skip the entire chain.
- **All workers fail**: Report all errors, ask user to retry with adjusted briefs.
- **Reviewer fails**: Report "review unavailable" for that variant. User can inspect manually.
- **Never block the pipeline** on a single failure — continue with successful agents and unblocked waves.

## Important Notes

- Always get user confirmation before launching workers (Phase 1)
- Launch workers within the same wave in a **single message** for true parallelism
- Dependent workers wait for their prerequisites and **branch off the prerequisite's branch**, not the base branch
- Workers must **commit their changes** but NOT create PRs or merge
- Worker commit messages: use conventional commits format — `<type>(<scope>): <brief description>` (e.g., `refactor(web): extract API route for contact form`). Do NOT use `multi-agent` as the commit type.
- Keep the user informed of progress between phases and waves
