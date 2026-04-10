---
name: review-pr
description: Review a GitHub pull request for code quality, security, performance, and consistency with project conventions.
allowed-tools: Bash(gh:*) Read Glob Grep
---

# Review Pull Request

Review a GitHub pull request with production-grade rigor. The benchmark: a subsequent automated reviewer (e.g., GitHub Copilot) should find **zero** new issues after your review.

## Usage

```
/review-pr <pr-number>
```

## Process

1. **Fetch PR context** using GitHub CLI:
   ```bash
   gh pr view $ARGUMENTS --json title,body,author,baseRefName,headRefName,files
   gh pr diff $ARGUMENTS
   gh pr view $ARGUMENTS --comments
   ```

2. **Read project conventions** (MANDATORY — do this before reviewing any code):
   - Check for `AGENTS.md` in the workspace/app directory (e.g., `apps/web/AGENTS.md`)
   - Check for `CLAUDE.md` in the repo root
   - These define project-specific conventions (directory structure, type patterns, import ordering, etc.) that **override** generic best practices
   - All review findings must be validated against these project conventions

3. **Read related local files** to understand the broader context:
   - Read files that are modified in the PR
   - Read files that import/are imported by modified files
   - Check for related tests

4. **Review against ALL checklists** — load and check every item in:
   - `references/general-checklist.md` — applies to ALL code
   - `references/ui-checklist.md` — applies to frontend/React/CSS changes
   - `references/api-checklist.md` — applies to API routes and backend changes

5. **Report findings** in the conversation organized by severity

6. **Ask before posting** any comments to GitHub

## Output Format

Present findings grouped by file, then by severity:

```
## PR #<number>: <title>

### Summary
<1-2 sentence overview of the PR and overall assessment>

### Checklist Results
<For each applicable checklist, mark PASS or list specific failures>

### Findings

#### <filename>

- **issue**: <description of blocking problem>
- **suggestion**: <improvement that would be nice to have>
- **question**: <something needing clarification>
- **praise**: <something done well>

### Verdict
<APPROVE | REQUEST_CHANGES | COMMENT>
<brief justification>
```

## Conventional Comments

When preparing comments to post on the PR, use the conventional comment format:

```
<type>: <description>
```

| Type | Purpose | Blocking? |
|------|---------|-----------|
| **issue** | Problem that MUST be addressed before merging | Yes |
| **suggestion** | Improvement, not required but good to have | No |
| **question** | Ask for clarification on code intent | No |
| **praise** | Highlight something done well | No |
| **nitpick** | Personal opinion, completely optional | No |
| **note** | Non-blocking observation | No |

## Posting Comments

After presenting findings, ask:

> Would you like me to post these comments to the PR?

If yes, use `gh pr review $ARGUMENTS` with the appropriate flag:
- `--approve` for APPROVE
- `--request-changes` for REQUEST_CHANGES
- `--comment` for COMMENT
