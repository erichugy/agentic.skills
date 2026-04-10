---
name: sync-containers
description: Propagate skill, config, and agent changes to Docker containers. Use after updating skills, CLAUDE.md, AGENTS.md, or any .claude/.agents config to sync changes into running or stopped claude-sandbox containers. Triggers on "sync containers", "propagate", "update containers", "push to docker".
---

# Sync Containers — Propagate Config to Docker

After updating skills, CLAUDE.md, AGENTS.md, hooks, or other agent config, propagate those changes to all claude-sandbox Docker containers so they stay in sync with the host.

## Usage

```
/sync-containers
```

Optionally target specific containers: `/sync-containers claude-sandbox-hq claude-sandbox-botpress`

## Process

### 1. Discover containers

List all claude-sandbox containers (running or stopped):

```bash
docker ps -a --filter "name=claude-sandbox" --format "{{.Names}}\t{{.Status}}"
```

Present the list to the user and confirm which containers to sync. Default: all claude-sandbox containers.

### 2. Determine what to sync

The following paths are synced from host to container:

| Host Path | Container Path | Permissions | Notes |
|-----------|---------------|-------------|-------|
| `~/.claude/skills/` | `/home/claude/.claude/skills/` | `claude:claude` | All skill files |
| `~/.claude/commands/` | `/home/claude/.claude/commands/` | `claude:claude` | Custom commands |
| `~/.claude/CLAUDE.md` | `/home/claude/.claude/CLAUDE.md` | `claude:claude` | Global instructions |
| `~/.claude/settings.json` | `/home/claude/.claude/settings.json` | `root:root 444` | Locked, read-only |
| `~/.claude/settings.local.json` | `/home/claude/.claude/settings.local.json` | `claude:claude` | Local overrides |
| `~/.agents/AGENTS.md` | `/home/claude/.agents/AGENTS.md` | `claude:claude` | Agent instructions |
| `~/.agents/skills/` | `/home/claude/.agents/skills/` | `claude:claude` | Agent skills |
| `~/.agents/hooks/` | `/home/claude/.agents/hooks/` | `root:root 444` | Locked, read-only |
| `~/.agents/.claude/` | `/home/claude/.agents/.claude/` | `claude:claude` | Agent claude config |

### 3. Sync each container

For each target container, launch a **background subagent** (`run_in_background: true`) that:

1. **Starts the container** if not running (track whether we started it so we can stop it after)

2. **Copies files** using `docker cp`:
   ```bash
   # Skills and commands
   docker cp ~/.claude/skills/. <container>:/home/claude/.claude/skills/
   docker cp ~/.claude/commands/. <container>:/home/claude/.claude/commands/ 2>/dev/null || true

   # CLAUDE.md (resolve symlinks)
   cp -L ~/.claude/CLAUDE.md /tmp/claude-sync-CLAUDE.md 2>/dev/null && \
     docker cp /tmp/claude-sync-CLAUDE.md <container>:/home/claude/.claude/CLAUDE.md && \
     rm /tmp/claude-sync-CLAUDE.md

   # Settings
   docker cp ~/.claude/settings.json <container>:/home/claude/.claude/settings.json
   docker cp ~/.claude/settings.local.json <container>:/home/claude/.claude/settings.local.json 2>/dev/null || true

   # Agents
   docker cp ~/.agents/AGENTS.md <container>:/home/claude/.agents/AGENTS.md 2>/dev/null || true
   docker cp ~/.agents/skills/. <container>:/home/claude/.agents/skills/ 2>/dev/null || true
   docker cp ~/.agents/hooks/. <container>:/home/claude/.agents/hooks/ 2>/dev/null || true
   docker cp ~/.agents/.claude/. <container>:/home/claude/.agents/.claude/ 2>/dev/null || true
   ```

3. **Rewrites paths** for container environment:
   ```bash
   docker exec <container> bash -c "
     for f in /home/claude/.claude/settings.json /home/claude/.claude/CLAUDE.md; do
       [ -f \"\$f\" ] || continue
       sed -i 's|~/.claude/|/home/claude/.claude/|g' \"\$f\"
       sed -i 's|~/.agents/|/home/claude/.agents/|g' \"\$f\"
       sed -i 's|$HOME/.claude/|/home/claude/.claude/|g' \"\$f\"
       sed -i 's|$HOME/.agents/|/home/claude/.agents/|g' \"\$f\"
     done
   "
   ```

4. **Sets permissions**:
   ```bash
   docker exec <container> bash -c "
     chown -R claude:claude /home/claude/.claude/skills/ 2>/dev/null || true
     chown -R claude:claude /home/claude/.claude/commands/ 2>/dev/null || true
     [ -f /home/claude/.claude/CLAUDE.md ] && chown claude:claude /home/claude/.claude/CLAUDE.md
     [ -f /home/claude/.claude/settings.local.json ] && chown claude:claude /home/claude/.claude/settings.local.json
     chown -R claude:claude /home/claude/.agents/AGENTS.md 2>/dev/null || true
     chown -R claude:claude /home/claude/.agents/skills/ 2>/dev/null || true
     chown -R claude:claude /home/claude/.agents/.claude/ 2>/dev/null || true

     # Lock hooks and settings (root-owned, read-only)
     chown root:root /home/claude/.claude/settings.json
     chmod 444 /home/claude/.claude/settings.json
     if [ -d /home/claude/.agents/hooks ]; then
       chown -R root:root /home/claude/.agents/hooks/
       chmod 555 /home/claude/.agents/hooks/
       find /home/claude/.agents/hooks/ -type f -exec chmod 444 {} +
     fi
     if [ -d /home/claude/.claude/hooks ]; then
       chown -R root:root /home/claude/.claude/hooks/
       chmod 555 /home/claude/.claude/hooks/
       find /home/claude/.claude/hooks/ -type f -exec chmod 444 {} +
     fi
   "
   ```

5. **Stops the container** if we started it (restore to previous state)

### 4. Report

```
## Container Sync Results

| Container | Status | Synced | Notes |
|-----------|--------|--------|-------|
| claude-sandbox-hq | was running | Yes | — |
| claude-sandbox-botpress | was stopped → started → synced → stopped | Yes | — |
| claude-sandbox | was stopped → started → synced → stopped | Yes | — |

Files synced: skills/, commands/, CLAUDE.md, settings.json, AGENTS.md, hooks/
```

## Integration with Other Skills

This skill should be run after:
- `/update-skills` — when skills are modified based on session analysis
- Manual edits to `~/.claude/skills/`, `~/.agents/`, or `CLAUDE.md`
- The self-improvement agent in `/address-pr-comments` (Step 8) makes approved changes

The `/update-skills` skill should suggest running `/sync-containers` after implementing approved changes if Docker containers exist.
