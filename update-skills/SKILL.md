---
name: update-skills
description: Review a conversation or session and identify improvements to agent skills. Analyzes what went well, what went wrong, and proposes skill updates. Use when the user says "update skills", "improve skills", "what can we learn", or wants to refine the agent system based on experience.
---

# Update Skills — Continuous Improvement

Review a conversation or work session, identify gaps or failures in the agent skill system, and propose targeted improvements. Interview the user on which improvements to implement.

## Usage

```
/update-skills
```

Optionally provide context: `/update-skills <conversation summary or link>`

## Process

### Step 1: Analyze the Session

Review the current conversation (or referenced session) and extract:

1. **What went well** — which skills were applied correctly and produced good results?
2. **What went wrong** — where did agents make mistakes, produce poor output, or miss something?
3. **What was missing** — did the user have to correct the agent on something not covered by any skill?
4. **What was redundant** — were multiple skills saying conflicting things?
5. **What was vague** — did agents misinterpret a skill because it was unclear?

### Step 2: Categorize Findings

For each finding, classify as:

| Category | Description | Example |
|----------|-------------|---------|
| **New skill needed** | A gap — no skill covers this | "Agents don't know our API versioning conventions" |
| **Skill update needed** | An existing skill is incomplete or unclear | "/typescript doesn't mention our enum naming convention" |
| **Skill conflict** | Two skills give contradictory guidance | "/react says X but /coding-conventions says Y" |
| **Skill too broad** | A skill tries to cover too much and gets ignored | "/coding-conventions has 200 rules and agents skip half" |
| **Skill too narrow** | A skill is so specific it's rarely useful | A pattern skill that only applies to one use case |
| **Workflow gap** | The orchestration (plan/multi-agent/workflow) failed | "Planner didn't assign /file-naming to the worker creating new files" |

### Step 3: Propose Improvements

For each finding, draft a specific proposal:

```markdown
## Proposed Improvements

### 1. [Category] <title>
- **Problem**: <what went wrong and why>
- **Affected skill(s)**: <skill name(s)>
- **Proposed change**: <specific addition, modification, or new skill>
- **Impact**: <what this fixes going forward>
- **Priority**: HIGH / MEDIUM / LOW
```

### Step 4: Interview the User

Present all proposals grouped by priority (HIGH first). For each one, ask:

> **Proposal N: <title>**
> <brief description>
>
> Implement this? (yes / no / modify)

Wait for the user's response on each proposal before implementing. Accept batch responses like "yes to all HIGH priority" or "implement 1, 3, 5, skip 2 and 4".

### Step 5: Implement Approved Changes

For each approved proposal:
1. Read the current skill file
2. Make the targeted change (don't rewrite the whole file)
3. If it's a new skill, follow the standard skill format with frontmatter
4. Update `/code-styles` index if a new style skill was added
5. Update `/design-patterns` index if a new pattern skill was added
6. Show the diff to the user for confirmation

### Step 6: Summary

```markdown
## Skills Updated

| Skill | Change | Status |
|-------|--------|--------|
| /typescript | Added enum naming convention | Done |
| /api-versioning | New skill created | Done |
| /plan | Updated assignment rules | Done |
| /react | Clarified server component rules | Skipped (user declined) |

### Remaining Gaps
<anything the user deferred or that needs more thought>

### Docker Containers
Run `/sync-containers` to propagate these changes to your Docker sandboxes.
```

## What to Look For

When analyzing a session, pay special attention to:

- **User corrections** — "no, don't do it that way" = missing or unclear skill
- **Repeated mistakes** — agent made the same error multiple times = skill not strong enough
- **Manual overrides** — user had to manually fix something the agent should have handled
- **Wasted iterations** — reviewer caught something the worker's skill should have prevented
- **Style inconsistencies** — code didn't match project patterns = missing convention
- **Naming issues** — files/folders named vaguely = `/file-naming` not assigned or too weak
- **Architecture issues** — wrong pattern used = `/design-patterns` or `/plan` gap
