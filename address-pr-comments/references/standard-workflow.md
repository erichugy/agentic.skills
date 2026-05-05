# Standard Workflow

Detailed flow for the default `/address-pr-comments` mode.

## 1. Fetch Open Review Threads

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

## 2. Triage Comments

For each unresolved thread, classify as:
- **actionable** — requires a code change (fix, refactor, rename, add attribute, etc.)
- **acknowledged** — valid observation but intentionally skipped (e.g., over-engineering concern, build-time year is fine). Resolve with a brief reply explaining why.
- **noise** — bot comments, deployment notifications, etc. Skip entirely.

Present the triage to the user before making changes. Group by PR.

## 3. Plan Skill Assignments

Before dispatching agents, use `/plan` and `/code-styles` to determine skill assignments:

1. **Read `/code-styles`** to understand the full skill hierarchy
2. **Identify languages and frameworks** in the PR's changed files (`.ts` → `/typescript`, `.rs` → `/rust`, React components → `/react`, etc.)
3. **Assign skills per agent** using the `/code-styles` assignment guide

## 4. No-Op Detection

Before implementing any fix, verify it is **not a no-op**. A no-op is a change that has no functional effect — renaming to the same name, adding code that's already there, restructuring without behavior change when the comment didn't ask for it, etc.

For each actionable comment:
- Read the current state of the code at the commented location
- Compare what the comment asks for against what already exists
- If the code already satisfies the comment (e.g., the fix was already applied in a subsequent commit), reclassify the comment as **acknowledged** and resolve with a reply explaining it's already addressed

**After implementing all fixes**, before committing:
- Run `git diff` and review every hunk
- If any hunk is a no-op (whitespace-only change, reordering with no effect, adding something already present), revert that hunk
- If ALL changes are no-ops, do NOT commit. Report that all comments were already addressed.

## 5. Resolve Threads

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

## 6. Report

Summarize what was done:

```markdown
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
