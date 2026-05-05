---
name: address-pr-comments
description: Address and resolve open review comments on GitHub pull requests. Use when the user says "address comments", "resolve PR comments", "fix PR feedback", or wants to handle review threads on one or more PRs. Supports single PR or batch mode across multiple PRs.
---

# Address PR Comments

Read open review threads on GitHub PRs, implement requested changes, and resolve the threads. Orchestrates existing skills for research, implementation, security review, style enforcement, and self-improvement.

## Usage

```
/address-pr-comments <pr-number> [pr-number...]
```

## When to Use

Use this skill when:
- The user asks to address, resolve, or fix PR review comments
- One or more GitHub PR review threads need actionable follow-up
- A bot reviewer is gating the PR and the user wants iterative cleanup
- Multiple PRs need comments handled in parallel with separate worktrees

Do not use this for a general PR review; use `/review-pr` instead.

## Reviewer Modes

| Mode | When | Loop strategy |
|------|------|---------------|
| **Standard** (default) | Human reviewers, mixed reviewers | One pass: triage → fix → resolve → push. Stop when threads are resolved. |
| **Iterative bot mode** | A bot reviewer with a confidence/quality score is the gating reviewer | Loop until the bot reports a clean score and zero unresolved comments, capped at 5 iterations. See [iterative bot mode](references/iterative-bot-mode.md). |

If the user says "iterate until Greptile is happy" / "loop until 5/5" / "until the bot is clean", use iterative bot mode. Otherwise use standard.

## Quick Workflow

| Step | Action | Detail |
|------|--------|--------|
| 1 | Fetch threads | Fetch unresolved review threads with GitHub GraphQL. See [standard workflow](references/standard-workflow.md). |
| 2 | Triage | Classify each comment as actionable, acknowledged, or noise. Present triage before changes. |
| 3 | Plan skills | Use `/plan` and `/code-styles` to assign language/framework skills. |
| 4 | Detect no-ops | Verify each requested fix is not already satisfied before editing. |
| 5 | Implement | Use `/multi-agent` worktree orchestration and the [agent role briefs](references/agent-roles.md). |
| 6 | Review | Run security and style reviewers until both pass. |
| 7 | Resolve | Reply to acknowledged comments, resolve addressed threads, then report. |
| 8 | Improve | Launch the self-improvement analysis after all PRs are processed. See [self-improvement loop](references/self-improvement.md). |

## Triage Classes

| Class | Meaning | Action |
|-------|---------|--------|
| **actionable** | Requires code change: fix, refactor, rename, attribute, validation, etc. | Implement, verify, push, resolve. |
| **acknowledged** | Valid observation intentionally skipped, or already addressed. | Reply with a brief explanation, then resolve. |
| **noise** | Bot noise, deployment notifications, non-review comments. | Skip entirely. |

## No-Op Detection

Before implementing any fix, verify it is **not a no-op**. A no-op has no functional effect: renaming to the same name, adding code already present, or restructuring without the requested behavior/style improvement.

For each actionable comment:
- Read the current code at the commented location
- Compare the request against what already exists
- If already satisfied, reclassify as **acknowledged** and resolve with an explanation

After implementing fixes:
- Run `git diff` and review every hunk
- Revert whitespace-only or effect-free hunks
- If all changes are no-ops, do not commit; report that all comments were already addressed

## Agent Roles

| Role | Skills Used |
|------|-------------|
| Planning | `/plan`, `/code-styles` |
| Research | `/explore` |
| Implementation | `/multi-agent` worker pattern, `/coding-conventions`, `/file-naming`, language skill, framework skill |
| Security Review | `/review-pr` checklists, `/coding-conventions`, language skill |
| Style Enforcement | Language skill, `/coding-conventions`, `/code-comments`, `/simplify`, `/file-naming` |
| Orchestration | `/multi-agent` worktrees, wave dispatch, review loop |
| Bot-reviewer Loop | `gh pr checks --watch`, confidence-score parsing, iteration cap; see [iterative bot mode](references/iterative-bot-mode.md) |
| Self-Improvement | `/update-skills`; see [self-improvement loop](references/self-improvement.md) |

## Commit and Push Rules

- Commit only if the final diff has functional or meaningful style changes
- Use `fix(<scope>): address PR review comments` for standard mode
- Use `fix(<scope>): address <bot> review feedback (iteration N)` for iterative bot mode
- Push the updated branch after reviewers pass
- For multiple PRs, run each PR pipeline in parallel with separate worktrees

## Resolution and Reporting

Resolve addressed threads with GitHub GraphQL. For acknowledged comments, reply before resolving. Use the report templates in [standard workflow](references/standard-workflow.md) and [iterative bot mode](references/iterative-bot-mode.md).

## Reference Files

- [Standard workflow](references/standard-workflow.md) — detailed GitHub API calls, triage, resolution, and report format
- [Agent roles](references/agent-roles.md) — researcher, implementer, security reviewer, style enforcer, and review loop prompts/checks
- [Iterative bot mode](references/iterative-bot-mode.md) — bot-reviewer loop, score parsing, iteration commits, and summary format
- [Self-improvement loop](references/self-improvement.md) — post-run analysis and `/update-skills` proposal format
