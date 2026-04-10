# Dockerfile Template

Use this as the base Dockerfile for the Claude playground container.

```dockerfile
FROM node:22-slim

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    bash \
    curl \
    wget \
    git \
    jq \
    gh \
    python3 \
    python3-pip \
    python3-venv \
    build-essential \
    vim \
    nano \
    less \
    tree \
    ripgrep \
    fd-find \
    fzf \
    zip \
    unzip \
    openssh-client \
    ca-certificates \
    gnupg \
    lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Install pnpm globally
RUN npm install -g pnpm

# Install Claude Code CLI globally
RUN npm install -g @anthropic-ai/claude-code

# Create non-root user "claude" (no sudo access)
RUN useradd -m -s /bin/bash claude

# Install bun for the claude user
USER claude
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/home/claude/.bun/bin:$PATH"
RUN echo 'export PATH="/home/claude/.bun/bin:$PATH"' >> /home/claude/.bashrc
USER root

# Create workspace and config directories
RUN mkdir -p /home/claude/.claude /home/claude/.agents /workspace

WORKDIR /workspace

# Container runs as root (entrypoint) so setup script can set permissions.
# All interactive use goes through: docker exec --user claude
CMD ["sleep", "infinity"]
```

## Notes

- The container entrypoint is `sleep infinity` running as root. This keeps the container alive and allows the setup script to set file permissions (chown/chmod) before Claude uses it.
- All interactive sessions use `docker exec --user claude` so Claude never has root access.
- The `claude` user has NO sudo access — this is critical for the security model.
- Bun is installed per-user, not globally, so it lives in `/home/claude/.bun/bin/`.
- The PATH export is added to `.bashrc` so it's available in interactive shells.

## Package Manager Detection

After copying the target folder, check which package manager it uses:

| Lockfile | Package Manager |
|----------|----------------|
| `bun.lockb` or `bun.lock` | bun |
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn (add to Dockerfile: `npm install -g yarn`) |
| `package-lock.json` | npm |

All of bun, npm, and pnpm are pre-installed. Only yarn needs to be added if detected.
