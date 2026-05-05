# Self-Improvement Loop

After all PRs are processed, launch a **background subagent** (`run_in_background: true`) that analyzes the entire session and proposes improvements to prevent recurring issues. This runs in parallel with the user reviewing the report.

## Prompt Inputs

The subagent's prompt must include:
- The full list of PR comments that were triaged (actionable, acknowledged, noise)
- The categories of fixes that were made
- The number of review loop iterations per PR
- What the security reviewer and style enforcer caught
- Any no-ops that were detected

## Analysis Tasks

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
