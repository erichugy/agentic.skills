---
name: address-pr-comments
description: Address and resolve open review comments on GitHub pull requests. Use when the user says "address comments", "resolve PR comments", "fix PR feedback", or wants to handle review threads on one or more PRs. Supports single PR or batch mode across multiple PRs.
---

# Address PR Comments

Read open review threads on GitHub PRs, implement the requested changes, and resolve the threads. Orchestrates existing skills for research, implementation, security review, and style enforcement.

## Usage

```
/address-pr-comments <pr-number> [pr-number...]
```

## Reviewer modes

Two operating modes depending on who is reviewing the PR:

| Mode | When | Loop strategy |
|------|------|---------------|
| **Standard** (default) | Human reviewers, mixed reviewers | One pass: triage → fix → resolve → push. Stop when threads are resolved. |
| **Iterative bot mode** | A bot reviewer with a confidence/quality score (e.g., Greptile, Vercel Agent) is the gating reviewer | Loop the full process until the bot reports a clean score (e.g., 5/5) **and** zero unresolved comments, capped at 5 iterations. See the [Iterative bot mode](#iterative-bot-mode-eg-greptile) section. |

If the user says "iterate until Greptile is happy" / "loop until 5/5" / "until the bot is clean", use iterative bot mode. Otherwise use standard.

## Process

### 1. Fetch open review threads

For each PR, fetch all unresolved review threads:

```bash
gh api graphql -f query='query {
  repository(owner: "<owner>", name: "<repo>") {
    pullRequest(number: <N>) {
      headRefName
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 5) {
            nodes { body path line originalLine diffHunk }
          }
        }
      }
    }
  }
}'
```

Filter to `isResolved: false` threads only. Extract the repo owner/name from `gh repo view --json owner,name`.

### 2. Triage comments

For each unresolved thread, classify as:
- **actionable** — requires a code change (fix, refactor, rename, add attribute, etc.)
- **acknowledged** — valid observation but intentionally skipped (e.g., over-engineering concern, build-time year is fine). Resolve with a brief reply explaining why.
- **noise** — bot comments, deployment notifications, etc. Skip entirely.

Present the triage to the user before making changes. Group by PR.

### 3. Plan skill assignments

Before dispatching agents, use `/plan` and `/code-styles` to determine skill assignments:

1. **Read `/code-styles`** to understand the full skill hierarchy
2. **Identify languages and frameworks** in the PR's changed files (`.ts` → `/typescript`, `.rs` → `/rust`, React components → `/react`, etc.)
3. **Assign skills per agent** using the `/code-styles` assignment guide

### 4. No-Op Detection (CRITICAL)

Before implementing any fix, verify it is **not a no-op**. A no-op is a change that has no functional effect — renaming to the same name, adding code that's already there, restructuring without behavior change when the comment didn't ask for it, etc.

For each actionable comment:
- Read the current state of the code at the commented location
- Compare what the comment asks for against what already exists
- If the code already satisfies the comment (e.g., the fix was already applied in a subsequent commit), reclassify the comment as **acknowledged** and resolve with a reply explaining it's already addressed

**After implementing all fixes**, before committing:
- Run `git diff` and review every hunk
- If any hunk is a no-op (whitespace-only change, reordering with no effect, adding something already present), revert that hunk
- If ALL changes are no-ops, do NOT commit. Report that all comments were already addressed.

### 5. Multi-agent implementation loop

Use the `/multi-agent` orchestration pattern: work in **git worktrees**, dispatch all agents as **subagents** (`run_in_background: true` where independent, foreground where sequential), and run an iterative review loop until clean.

All agents below are launched as subagents via the `Agent` tool. The manager orchestrates them — it does not do the work itself.

For each PR with actionable comments, dispatch the following subagents:

#### Subagent 1: Researcher — via `/explore`

Launch as background subagent(s) (`run_in_background: true`, `subagent_type: "Explore"`). Deploy parallel Explore agents to investigate each actionable comment. Each agent:
- Reads the commented file and surrounding code
- Checks for existing blueprints at `<package-root>/.claude/blueprint.md`
- Traces imports, types, and related files to understand the full context
- Determines the best solution approach
- **Verifies the comment isn't already addressed** (no-op check) — if it is, report back so the comment can be resolved without changes
- Documents what files need changing and why

Feed the researcher's findings into the implementer's brief.

#### Subagent 2: Implementer — via `/multi-agent` worker pattern

Launch as a subagent in a worktree (`isolation: "worktree"`, `subagent_type: "general-purpose"`). Follows `/multi-agent` Phase 2 conventions:
- Check out the PR branch in a worktree
- Apply all fixes based on the researcher's findings (skip any the researcher flagged as already addressed)
- Self-check against the Production Hardening Checklist from `references/review-rubric.md` (as specified in `/multi-agent` worker briefs)
- **Skills**: assign per `/plan` output — always includes `/coding-conventions` + `/file-naming` + the appropriate language skill (e.g., `/typescript`, `/rust`) + framework skill if applicable (e.g., `/react`)
- **No-op check**: after making changes, run `git diff` and verify every hunk has a functional effect. Revert any no-op hunks.
- Run the project's build command to verify nothing is broken
- Commit with message: `fix(<scope>): address PR review comments`

#### Subagent 3: Security Reviewer — via `/review-pr` checklists

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

#### Subagent 4: Style Enforcer — via language skill + `/coding-conventions` + `/code-comments` + `/simplify`

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

#### Review loop — per `/multi-agent` Phase 3

Follow the `/multi-agent` review loop protocol exactly:

1. Subagents 3 and 4 review the implementer's changes (launched in parallel in a single message)
2. If either finds issues (including no-ops) → manager applies fixes / reverts no-ops → commits → re-launch reviewers
3. Loop until **both** reviewers return PASS with zero issues
4. Maximum 3 iterations — if issues persist after 3 rounds, present remaining concerns to the user
5. Quality bar (from `/multi-agent`): "A subsequent automated reviewer (GitHub Copilot) should find zero new issues."

#### Commit and push

Once both reviewers pass:
1. Run a final `git diff` — if the diff is empty (all changes were no-ops that got reverted), do NOT commit or push. Report that all comments were already addressed.
2. Otherwise, commit with message: `fix(<scope>): address PR review comments`
3. Push the updated branch

When multiple PRs need fixes, run each PR's agent pipeline in parallel using separate worktrees.

### 6. Resolve threads

After pushing fixes, resolve all addressed threads via GraphQL:

```bash
gh api graphql -f query='mutation {
  resolveReviewThread(input: {threadId: "<THREAD_ID>"}) {
    thread { isResolved }
  }
}'
```

For "acknowledged" comments, reply before resolving:

```bash
gh api graphql -f query='mutation {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<THREAD_ID>",
    body: "<explanation>"
  }) { comment { id } }
}'
```

Then resolve the thread.

### 6b. Iterative bot mode (e.g., Greptile)

If running in **iterative bot mode**, wrap the entire process (steps 1-6) in a loop until the bot reviewer reports a clean state.

**Triggers** (any of):
- User says "iterate until Greptile is happy", "loop until 5/5", "until the bot is clean"
- The PR is gated on a bot reviewer with a confidence/quality score
- The user names a bot reviewer explicitly (Greptile, Vercel Agent, etc.)

#### Loop protocol

```
┌──────────────────────────────────────────────────────────────────────┐
│ A. Push & wait for bot review                                        │
│    git push                                                          │
│    gh pr checks <PR> --watch       # block until bot check completes │
│                                                                       │
│ B. Fetch bot review                                                   │
│    gh api repos/{owner}/{repo}/pulls/<PR>/reviews                    │
│    → find latest review from greptile-apps[bot] (or other bot)       │
│    → parse confidence score (e.g., "3/5", "5/5") from review body    │
│    → fetch unresolved inline comments via reviewThreads GraphQL      │
│                                                                       │
│ C. Exit conditions (stop if ANY)                                      │
│    - Confidence == max (e.g., 5/5) AND zero unresolved comments      │
│    - Iteration count >= 5 (cap to avoid runaway loops)                │
│    - User intervention requested                                      │
│                                                                       │
│ D. Otherwise, run steps 1-6 of the standard process                   │
│    → triage, no-op check, multi-agent fix, resolve threads           │
│                                                                       │
│ E. Increment iteration; go back to A.                                 │
└──────────────────────────────────────────────────────────────────────┘
```

#### Parsing confidence scores

Bot reviews typically include a score in the review body. Use a regex search like `/(\d)\s*\/\s*5/` to extract `N/5` patterns. If multiple appear, use the last one (final summary). If no score is parseable, treat the bot's "Approved" review event as a clean signal and "Changes requested" as not-clean.

#### Per-iteration commit message

Use a descriptive commit message that includes the iteration number:

```
fix(<scope>): address <bot> review feedback (iteration N)
```

#### Output format (iterative mode)

After the loop exits, replace the standard Report (step 7) with the iterative summary:

```
## PR #<N>: <title> — Iterative <bot> mode

| Field             | Value      |
|-------------------|------------|
| Iterations        | N          |
| Final confidence  | X/5        |
| Comments resolved | M          |
| Remaining         | K          |

Status: ✓ Clean / ⚠ Stopped at iteration cap / ✗ User intervention

Remaining issues (if K > 0):
  - <path>:<line> — "<comment excerpt>"
  - ...
```

If the loop exited due to the iteration cap, list the remaining issues and surface a recommendation: a remaining comment after 5 iterations is usually a false positive, a design disagreement, or a structural change that needs human input — flag it for the user rather than churning further.

### 7. Report

Summarize what was done:

```
## PR #<N>: <title>

### Changes
- Resolved: <count> threads
- Skipped: <count> (with reasons)
- No-ops detected: <count> (comments already addressed or changes with no functional effect)
- Pushed: <commit-sha> on <branch> (or "No push — all changes were no-ops")

### Security Review
- Checklist(s) applied: general, api, ui (as applicable)
- Vulnerabilities found: <count>
- Vulnerabilities fixed: <count>
- Remaining concerns: <list or "none">

### Style
- Lint/format violations fixed: <count>
- Linter used: <tool name>
- Coding convention issues fixed: <count>
- Comment issues fixed: <count>
- Review iterations: <count>
```

### 8. Self-Improvement Agent

After all PRs are processed, launch a **background subagent** (`run_in_background: true`) that analyzes the entire session and proposes improvements to prevent recurring issues. This runs in parallel with the user reviewing the report from Step 7.

The subagent's prompt must include:
- The full list of PR comments that were triaged (actionable, acknowledged, noise)
- The categories of fixes that were made
- The number of review loop iterations per PR
- What the security reviewer and style enforcer caught
- Any no-ops that were detected

The subagent then:

1. **Analyzes the PR comments that were addressed:**
   - What categories of issues came up? (naming, types, validation, security, style, architecture, etc.)
   - Are there patterns? (e.g., "3 out of 5 comments were about missing input validation")
   - Were any comments about things that existing skills already cover? (indicates the worker wasn't assigned that skill, or the skill is too weak)

2. **Analyzes the review loop:**
   - How many iterations did it take? More than 1 means the worker's skills or brief were insufficient.
   - What did the security reviewer and style enforcer catch? Could these have been prevented by better worker briefs or stronger skills?

3. **Proposes targeted updates to:**

   - **Skills** (`~/.claude/skills/`) — if a category of PR comment isn't covered by any skill, propose a new skill or an addition to an existing one
   - **`AGENTS.md`** (`~/.agents/AGENTS.md`) — if the PR comments reveal project-specific conventions not captured in the agents config, propose additions
   - **`CLAUDE.md`** (`~/.claude/CLAUDE.md`) — if the comments reveal global conventions or preferences that should apply across all projects
   - **Project `CLAUDE.md`** (repo root) — if the comments reveal repo-specific patterns that workers should know about

4. **Presents proposals using the `/update-skills` format:**

```markdown
## Self-Improvement Proposals

### From PR #<N>

#### 1. [Skill update] <title>
- **Pattern observed**: <what PR comments kept flagging>
- **Root cause**: <why the agent didn't get it right — missing skill, weak rule, not assigned>
- **Proposed change**: <specific update to skill/AGENTS.md/CLAUDE.md>
- **Priority**: HIGH / MEDIUM / LOW

#### 2. [New convention] <title>
...
```

5. **Wait for user approval** before making any changes. Accept batch responses ("implement all", "skip 2", etc.).

6. **Implement approved changes** — edit the relevant skill files, AGENTS.md, or CLAUDE.md files. Show diffs for confirmation.

This step is the feedback loop that makes the system get smarter over time. PR review comments are signal — if the same kind of comment keeps appearing, the agent system should learn to prevent it.

## Skills Leveraged

| Role | Skills Used |
|------|-------------|
| Planning | `/plan`, `/code-styles` (determines all skill assignments below) |
| Research | `/explore` |
| Implementation | `/multi-agent` worker pattern, `/coding-conventions`, `/file-naming`, language skill, framework skill |
| Security Review | `/review-pr` checklists, `/coding-conventions`, language skill |
| Style Enforcement | Language skill, `/coding-conventions`, `/code-comments`, `/simplify`, `/file-naming` |
| Orchestration | `/multi-agent` (worktrees, wave dispatch, review loop) |
| Bot-reviewer Loop | `gh pr checks --watch`, confidence-score parsing, iteration cap (see [Iterative bot mode](#6b-iterative-bot-mode-eg-greptile)) |
| Self-Improvement | `/update-skills` (analyze patterns, propose skill/config updates) |
