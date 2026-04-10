---
name: create-claude-playground
description: Create an isolated Docker sandbox for running Claude Code with --dangerously-skip-permissions. Use when the user wants to set up a safe autonomous Claude environment for a specific project folder. Triggers on "playground", "docker sandbox", "dangerously-skip-permissions", "autonomous claude".
---

# Create Claude Playground

Spins up an isolated Docker container pre-configured for autonomous Claude Code usage with `--dangerously-skip-permissions`. The container runs Claude as a non-root user with immutable branch protection hooks.

**Each folder gets its own container** (`claude-sandbox-<folderName>`) so multiple playgrounds can coexist without interfering with each other.

## Usage

```
/create-claude-playground <folderName>
```

`<folderName>` is a directory name inside `~/Desktop/repos/`.

## Execution Steps

Follow these steps in order. Do NOT skip any step.

### Step 1: Validate Prerequisites

1. Check Docker is installed: `which docker`
2. Check Docker daemon is running: `docker info`
3. If Docker is not running, tell the user: **"Please open Docker Desktop and run this command again."** Then stop.
4. Validate `<folderName>` exists in `~/Desktop/repos/`
5. Check if `claude-sandbox-<folderName>` container already exists and is running. If so, tell the user it's already up and show them the connect command. Don't recreate.

### Step 2: Ask User Preferences

Ask the user (all at once, single prompt):

1. **Copy `.bashrc` and `.bash_profile`?** — "Should I copy your `~/.bashrc` and `~/.bash_profile` into the container? This brings over aliases and shell customizations."
2. **Additional folders?** — "Any other folders from `~/Desktop/repos/` to include? (comma-separated, or 'no')"
3. **Bind-mount the primary folder?** — "Should I bind-mount `<folderName>` so changes sync live between the container and your host? (Recommended for iterative work — files appear on both sides instantly, no manual copying needed)"

### Step 3: Generate Dockerfile

Reuse `~/Desktop/repos/scripts/claude-docker/Dockerfile` if it already exists. Only regenerate if missing.

See `references/dockerfile-template.md` for the full template.

### Step 4: Run the Setup Script

The setup script lives at `~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh`. Reuse if it exists, only regenerate if missing.

Build the flags based on user preferences:

```bash
FLAGS=""
# Add --copy-shell if user said yes to .bashrc/.bash_profile
# Add --mount <folders> if user wants bind-mounting (comma-separated folder names)
# Add extra folder names after the primary folder

~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh $FLAGS <folderName> [extra folders...]
```

The script handles everything: building the image, staging files, copying/mounting into container, locking hooks, verifying security, and generating the in-container `~/setup.sh`.

**Bind-mount flag**: If the user wants live sync between host and container (recommended for iterative work), add `--mount <folder1>,<folder2>` to bind-mount those folders instead of copying. Changes on either side appear instantly.

**IMPORTANT**: The script will output to stdout but the interactive `docker exec` for login won't work from within Claude. Instead, skip `--login` and just tell the user to run auth manually (Step 5).

So actually run:

```bash
~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh <folderName> [extra folders...] [--copy-shell] [--mount <folders>]
```

Without `--login` and without `--key` — let it fall through to interactive mode, which will fail silently. The important thing is the container gets created, files copied/mounted, and hooks locked.

### Step 5: Tell User to Authenticate

Since we can't do interactive auth from within Claude, tell the user to open their terminal and run:

```
docker exec -it --user claude claude-sandbox-<folderName> bash -l
bash ~/setup.sh
```

### Step 6: Print Usage Instructions

Always end with this full block — replace `<folderName>` with actual name, do NOT abbreviate:

```
============================================
Claude Playground 'claude-sandbox-<folderName>' Ready!
============================================

Available tools: node, npm, pnpm, bun (via bash -l), git, gh, jq,
  python3, curl, ripgrep, vim, nano, build-essential

Workspace: /workspace/<folderName>

Hooks: locked, root-owned, read-only

---

First-time setup — open your terminal and run:

  # Shell in
  docker exec -it --user claude claude-sandbox-<folderName> bash -l

  # Then inside the container:
  bash ~/setup.sh
  # (runs gh auth login + claude login)
  exit

---

Run Claude with autonomous permissions:

  docker exec -it --user claude claude-sandbox-<folderName> bash -lc \
    "cd /workspace/<folderName> && claude --dangerously-skip-permissions"

---

Security:
  - Claude runs as non-root user "claude" (no sudo)
  - Branch protection hooks are root-owned and read-only
  - Claude CANNOT push/merge to main, master, production, or release
  - Claude CANNOT modify the hooks or settings to bypass protection

Manage container:
  docker stop claude-sandbox-<folderName>
  docker start claude-sandbox-<folderName>
  ~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh --clean <folderName>
  ~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh --list
```

## Security Model

| Layer | What it protects | How |
|-------|-----------------|-----|
| Non-root user | Hooks & settings | Claude runs as `claude` user, cannot `chmod`/`chown` root-owned files |
| Read-only hooks | Branch protection | `protect-branches.sh` is `444` root-owned |
| Read-only settings | Hook config | `settings.json` is `444` root-owned, Claude can't remove hook references |
| PreToolUse hook | Protected branches | Blocks `git push` to main/master/production/release, blocks `gh pr merge` into those branches |

## Quick Reference

| What | Where (container) |
|------|-------------------|
| Container name | `claude-sandbox-<folderName>` |
| Workspace | `/workspace/<folderName>` |
| Claude config | `/home/claude/.claude/` |
| Agent config | `/home/claude/.agents/` |
| Hooks | `/home/claude/.claude/hooks/` (root-owned, read-only) |
| Settings | `/home/claude/.claude/settings.json` (root-owned, read-only) |
| Auth setup | `/home/claude/setup.sh` |
