# Summary and PR Chains

Detailed Phase 4 protocol, dependent PR chains, merge order, and error handling.

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

## PR Chains for Dependencies

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
