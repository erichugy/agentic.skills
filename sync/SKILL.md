---
name: sync
description: Full repo sync — fetch, prune remotes, remove worktrees and branches with no remote, pull all remaining branches. Use when the user says "sync", "clean up", "pull latest", or after merging PRs.
---

# Sync — Full Repo Cleanup & Pull

Sync the local repo with remote. Removes all stale worktrees and branches, pulls latest on everything that remains.

## Process

### 1. Fetch and prune remote tracking refs

```bash
git fetch --prune origin
```

### 2. Identify local branches with no remote

```bash
git branch -vv
git branch -r
```

Compare local branches against remote branches. A local branch is stale if:
- Its tracked remote branch was deleted (shows `[origin/...: gone]`)
- It has no tracking branch AND no matching `origin/<name>` exists

### 3. Remove worktrees for stale branches

```bash
git worktree list
```

For each stale branch that has a worktree, remove nested worktrees first (deepest path first), then outer ones:
```bash
git worktree remove <path> --force
```

After all removals: `git worktree prune`

### 4. Delete stale local branches

If currently on a stale branch, switch to main first:
```bash
git checkout main
```

Then delete all stale branches:
```bash
git branch -D <branch1> <branch2> ...
```

### 5. Pull latest on all remaining branches

For each remaining local branch (including main):
```bash
git checkout <branch> && git pull origin <branch>
```

Switch back to main when done.

### 6. Report final state

Show the final `git branch -vv` and `git worktree list` output so the user can verify.

## Rules

- **Auto-clean all branches/worktrees with no remote** — no confirmation needed
- **Never delete branches that have an active remote** — only pull them
- **Always fetch --prune first** to get accurate remote state
- **Remove nested worktrees before parent worktrees** to avoid errors
- **Always end on main branch**
