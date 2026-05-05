---
name: multi-agent
description: Orchestrate parallel worker subagents to explore multiple approaches to a task simultaneously. Use when the user wants to compare variants, explore design alternatives, or parallelize independent work across git worktrees. Triggers on "multi-agent", "parallel variants", "explore approaches", "compare designs", "multiple versions".
---

# Multi-Agent Manager

You are a **manager agent** that breaks a task into parallel variants, dispatches workers to separate git worktrees, dispatches reviewers, and presents a unified summary.

Workers and reviewers are launched via the `Task` tool with `isolation: "worktree"` and `run_in_background: true`.

## When to Use

Use this skill when:
- The user wants multiple approaches, variants, or designs explored in parallel
- Independent work can run in separate git worktrees
- A task benefits from worker/reviewer separation and an iterative review loop
- Work should be compared before selecting, merging, or turning into PRs

Do not use this for a single direct implementation unless the user explicitly asks for multi-agent orchestration.

## Quick Workflow

| Phase | Goal | Detail |
|-------|------|--------|
| 1. Plan | Use `/plan` to define variants, dependencies, waves, and skill assignments | See [planning and dispatch](references/planning-dispatch.md). |
| 2. Dispatch workers | Launch workers by wave in separate worktrees | Use [worker brief templates](references/worker-briefs.md). |
| 3. Review loop | Launch reviewers, fix issues, repeat until clean | See [review loop](references/review-loop.md) and [review rubric](references/review-rubric.md). |
| 4. Summary | Present branch, worktree, review, recommendation, and next steps | See [summary and PR chains](references/summary-pr-chains.md). |

## Planning Essentials

- Use `/plan` first; the planner determines agents, roles, skill assignments, wave ordering, and dependencies.
- Read `/code-styles` to assign language/framework skills to every worker and reviewer.
- Extract constraints, current base branch, languages/frameworks, variant count/directions, and worker dependencies.
- If variants are not explicit, generate 2-4 variant directions with short names and concise descriptions.
- Present the plan with `Variant`, `Direction`, `Depends On`, and `Wave` columns.
- Ask: **"Ready to launch N workers (M waves)? Adjust variants, dependencies, or say 'go'."**
- **Wait for confirmation before proceeding.** Do not launch workers without approval.

## Dispatch Essentials

- Branch naming: `multi-agent/<task-slug>/<variant-slug>`
- Launch all workers in the same wave in a **single message** for true parallelism.
- Wave 1 branches from the base branch.
- Wave 2+ waits for prerequisites and branches from the prerequisite branch.
- Track variant, wave, dependency, branch base, task ID, and status.
- If a prerequisite fails, mark dependents as `blocked` and report options.
- Wait for wave completion notifications; do not poll.

## Review Essentials

- Append the Production Hardening Checklist from [review rubric](references/review-rubric.md) to every worker prompt so workers self-check before committing.
- For each successful worker, launch a reviewer in a worktree with the branch name, original brief, full rubric, assigned skills, and quality bar.
- Launch all reviewers in parallel.
- If a reviewer returns Issues to Fix, apply fixes or send the worker back, commit, and re-review.
- The loop ends only when **every reviewer returns PASS with zero Issues to Fix**.
- A PASS with minor nits is not acceptable; fix nits and re-review.
- If a reviewer loops more than 3 times, pause and report to the user.

## Important Notes

- Always get user confirmation before launching workers.
- Launch workers within the same wave in a **single message** for true parallelism.
- Dependent workers wait for prerequisites and **branch off the prerequisite's branch**, not the base branch.
- Workers must **commit their changes** but NOT create PRs or merge.
- Worker commit messages use conventional commits: `<type>(<scope>): <brief description>` (for example, `refactor(web): extract API route for contact form`). Do NOT use `multi-agent` as the commit type.
- Keep the user informed of progress between phases and waves.
- Never block the pipeline on one failure; continue with successful agents and unblocked waves.

## Reference Files

- [Planning and dispatch](references/planning-dispatch.md) — detailed phase 1 and phase 2 protocol, wave tables, dependency handling
- [Worker briefs](references/worker-briefs.md) — worker prompt template and hardening checklist insertion point
- [Review loop](references/review-loop.md) — reviewer launch prompts, iterative fix loop, and clean-pass criteria
- [Review rubric](references/review-rubric.md) — Production Hardening Checklist and review criteria
- [Summary and PR chains](references/summary-pr-chains.md) — final summary format, dependent PR chains, merge order, and error handling
