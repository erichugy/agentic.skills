# Review Loop

Detailed Phase 3 protocol for `/multi-agent`.

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
