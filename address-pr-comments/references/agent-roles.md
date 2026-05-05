# Agent Roles and Review Loop

Detailed multi-agent implementation flow for addressing actionable PR comments.

## Multi-Agent Implementation Loop

Use the `/multi-agent` orchestration pattern: work in **git worktrees**, dispatch all agents as **subagents** (`run_in_background: true` where independent, foreground where sequential), and run an iterative review loop until clean.

All agents below are launched as subagents via the `Agent` tool. The manager orchestrates them — it does not do the work itself.

For each PR with actionable comments, dispatch the following subagents.

## Subagent 1: Researcher — via `/explore`

Launch as background subagent(s) (`run_in_background: true`, `subagent_type: "Explore"`). Deploy parallel Explore agents to investigate each actionable comment. Each agent:
- Reads the commented file and surrounding code
- Checks for existing blueprints at `<package-root>/.claude/blueprint.md`
- Traces imports, types, and related files to understand the full context
- Determines the best solution approach
- **Verifies the comment isn't already addressed** (no-op check) — if it is, report back so the comment can be resolved without changes
- Documents what files need changing and why

Feed the researcher's findings into the implementer's brief.

## Subagent 2: Implementer — via `/multi-agent` worker pattern

Launch as a subagent in a worktree (`isolation: "worktree"`, `subagent_type: "general-purpose"`). Follows `/multi-agent` Phase 2 conventions:
- Check out the PR branch in a worktree
- Apply all fixes based on the researcher's findings (skip any the researcher flagged as already addressed)
- Self-check against the Production Hardening Checklist from `references/review-rubric.md` (as specified in `/multi-agent` worker briefs)
- **Skills**: assign per `/plan` output — always includes `/coding-conventions` + `/file-naming` + the appropriate language skill (e.g., `/typescript`, `/rust`) + framework skill if applicable (e.g., `/react`)
- **No-op check**: after making changes, run `git diff` and verify every hunk has a functional effect. Revert any no-op hunks.
- Run the project's build command to verify nothing is broken
- Commit with message: `fix(<scope>): address PR review comments`

## Subagent 3: Security Reviewer — via `/review-pr` checklists

Launch as a background subagent (`run_in_background: true`, `isolation: "worktree"`, `subagent_type: "general-purpose"`). Follows `/multi-agent` Phase 3 review loop conventions:
- Runs `git diff` on the implementer's changes
- Applies the **full** `/review-pr` checklist suite:
  - `references/general-checklist.md` — all code
  - `references/api-checklist.md` — if API/backend code was touched
  - `references/ui-checklist.md` — if frontend code was touched
- **Skills**: `/coding-conventions` (validation/error handling sections) + the appropriate language skill
- Reads project conventions from `AGENTS.md` / `CLAUDE.md` (as `/review-pr` requires)
- Performs an OWASP Top 10 audit on all changed code
- Verifies fixes don't introduce new attack surfaces
- Flags unsafe patterns: unescaped HTML rendering, raw SQL queries, dynamic code evaluation, unsanitized user input, hardcoded secrets, overly permissive CORS/permissions
- **Flags no-ops**: if any change has no functional effect, flag it as an issue to revert
- Returns findings using the `/review-pr` output format (severity, conventional comments)

Launch in parallel with Subagent 4 (in a single message for true parallelism).

## Subagent 4: Style Enforcer — via language skill + `/coding-conventions` + `/code-comments` + `/simplify`

Launch as a background subagent (`run_in_background: true`, `isolation: "worktree"`, `subagent_type: "general-purpose"`). Runs in parallel with Subagent 3:
- **Skills**: assign per `/plan` output — always includes the language skill + `/coding-conventions` + `/code-comments` + `/simplify` + `/file-naming` (if new files were created)
- Applies the language skill (e.g., `/typescript` for naming, imports, type safety; `/rust` for ownership, clippy)
- Applies `/coding-conventions` — language-agnostic rules (validation, error handling, resource lifecycle)
- Applies `/code-comments` — conventional comment format, "only the why" rule
- Applies `/simplify` — reduce unnecessary complexity, eliminate redundancy, improve clarity without changing behavior
- Detects the project's linting setup (`.eslintrc*`, `biome.json`, `.prettierrc*`, `ruff.toml`, etc.)
- Runs the project's lint/format commands (e.g., `pnpm lint --fix`, `pnpm format`)
- If no configured linter is found, reviews code style manually against existing repo patterns
- **Flags no-ops**: if any change is purely cosmetic with no functional or style improvement, flag it for removal
- Fixes any style violations

## Review Loop — per `/multi-agent` Phase 3

Follow the `/multi-agent` review loop protocol exactly:

1. Subagents 3 and 4 review the implementer's changes (launched in parallel in a single message)
2. If either finds issues (including no-ops) → manager applies fixes / reverts no-ops → commits → re-launch reviewers
3. Loop until **both** reviewers return PASS with zero issues
4. Maximum 3 iterations — if issues persist after 3 rounds, present remaining concerns to the user
5. Quality bar (from `/multi-agent`): "A subsequent automated reviewer (GitHub Copilot) should find zero new issues."

## Commit and Push

Once both reviewers pass:
1. Run a final `git diff` — if the diff is empty (all changes were no-ops that got reverted), do NOT commit or push. Report that all comments were already addressed.
2. Otherwise, commit with message: `fix(<scope>): address PR review comments`
3. Push the updated branch

When multiple PRs need fixes, run each PR's agent pipeline in parallel using separate worktrees.
