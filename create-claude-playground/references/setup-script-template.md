# Setup Script Template

This is the reference template for `setup-claude-docker.sh`. The skill should generate this script (or reuse it if it already exists) at `~/Desktop/repos/scripts/claude-docker/setup-claude-docker.sh`.

## Key Design Decisions

1. **Container runs `sleep infinity` as root** — keeps it alive and allows permission setup
2. **All interactive use is `docker exec --user claude`** — Claude never has root
3. **Files are staged locally first**, then `docker cp`'d in — avoids volume mount issues
4. **Hooks are locked after copy** — `chown root:root` + `chmod 444`
5. **Security verification** — script tests that claude user cannot write to hooks/settings before declaring success

## Script Behavior

```
Usage: setup-claude-docker.sh [OPTIONS] [FOLDER_NAMES...]

Options:
  --key KEY       Anthropic API key (or set ANTHROPIC_API_KEY env var)
  --login         Use interactive 'claude login' inside the container
  --prompt TEXT   Run claude with this prompt non-interactively
  --rebuild       Force rebuild the Docker image
  --clean         Remove container and image, then exit
  --copy-shell    Copy ~/.bashrc and ~/.bash_profile into the container
  --help          Show this help
```

## Staging Logic

When staging files, the script should:

### Repos/Folders
- Copy each specified folder from `~/Desktop/repos/<name>` into staging
- **SKIP `node_modules/`** directories (use `rsync --exclude node_modules` or find+exclude)
- If no folders specified, copy ALL (excluding hidden dirs and node_modules)

### .agents/ Directory
- Copy only these items: `AGENTS.md`, `CLAUDE.md`, `hooks/`, `skills/`, `commands/`, `.claude/`
- **Skip `backup/` directory** — it contains broken symlinks
- Use `cp -aL` to resolve symlinks, fall back to `cp -a` if symlinks are broken

### .claude/ Directory
- `settings.json` and `settings.local.json`
- `hooks/` (from `~/.agents/hooks/` which is the source of truth)
- `CLAUDE.md` (resolve symlink)
- `commands/` (resolve symlink)
- `skills/` (resolve symlink)

### .gitconfig
- Always copy `~/.gitconfig` if it exists

### .bashrc / .bash_profile (opt-in)
- Only copy if user opted in (--copy-shell flag or user said yes when asked)

### Path Rewriting
After staging, rewrite paths in `settings.json` and `CLAUDE.md`:
- `~/.claude/` → `/home/claude/.claude/`
- `~/.agents/` → `/home/claude/.agents/`
- `$HOME/.claude/` → `/home/claude/.claude/`
- `$HOME/.agents/` → `/home/claude/.agents/`

## Permission Setup

After copying files into the container:

```bash
# Everything owned by claude user
chown -R claude:claude /workspace
chown -R claude:claude /home/claude/.agents
chown -R claude:claude /home/claude/.claude
chown -R claude:claude /home/claude/.bun
[ -f /home/claude/.gitconfig ] && chown claude:claude /home/claude/.gitconfig

# Lock hooks — root-owned, read-only
chown -R root:root /home/claude/.claude/hooks/
chmod 555 /home/claude/.claude/hooks/
find /home/claude/.claude/hooks/ -type f -exec chmod 444 {} +

# Lock settings.json
chown root:root /home/claude/.claude/settings.json
chmod 444 /home/claude/.claude/settings.json

# Lock .agents/hooks too
chown -R root:root /home/claude/.agents/hooks/
chmod 555 /home/claude/.agents/hooks/
find /home/claude/.agents/hooks/ -type f -exec chmod 444 {} +
```

## In-Container Setup Script

Generate `/home/claude/setup.sh` inside the container:

```bash
#!/bin/bash
echo "=========================================="
echo "  Claude Playground — First-Time Setup"
echo "=========================================="
echo ""

# GitHub CLI login
echo "Step 1: Authenticate GitHub CLI"
echo "---------------------------------"
gh auth login
echo ""

# Claude login
echo "Step 2: Authenticate Claude Code"
echo "----------------------------------"
claude login
echo ""

# Done
echo "=========================================="
echo "  Setup complete!"
echo "=========================================="
echo ""
echo "Run Claude with autonomous permissions:"
echo ""
echo "  claude --dangerously-skip-permissions"
echo ""
echo "REMINDER: Claude runs as a non-root user."
echo "Branch protection hooks prevent pushing to"
echo "main, master, production, and release branches."
echo "Claude cannot modify these hooks."
echo "=========================================="
```

## Security Verification

Before declaring success, the script MUST verify:

```bash
# Test 1: claude user cannot write to hooks
docker exec --user claude CONTAINER bash -c \
  "echo test >> /home/claude/.claude/hooks/protect-branches.sh" 2>/dev/null
# Must fail (exit code != 0)

# Test 2: claude user cannot write to settings
docker exec --user claude CONTAINER bash -c \
  "echo test >> /home/claude/.claude/settings.json" 2>/dev/null
# Must fail (exit code != 0)
```

If either test passes (claude CAN write), the script must abort with a security error.
