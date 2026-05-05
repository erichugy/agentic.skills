---
name: remind
description: Quick reference of all available workflows and when to use them. Use when the user says "remind", "what's the workflow", "how should I", or needs a refresher on the right process for a task.
---

# Remind — Workflow Quick Reference

Here are your workflows and when to use each one.

---

## Explore & Answer Questions → `/explore`

**When**: "How does X work?", "Where is Y?", "Explain this code", onboarding to a new repo.

```
1. Check for existing blueprint at <package>/.claude/blueprint.md
2. Deploy 2-4 parallel Explore agents with distinct focus areas
3. Update/create the blueprint with discoveries
4. Synthesize findings into a clear answer
```

---

## Address PR Review Comments → `/address-pr-comments <pr-number>`

**When**: "Address comments", "resolve PR feedback", "fix PR comments".

```
1. Fetch unresolved review threads via GraphQL
2. Triage: actionable / acknowledged / noise → present to you
3. Plan skill assignments via /plan + /code-styles
4. No-op check: skip comments already addressed
5. Multi-agent loop:
   - Researcher (via /explore) investigates each comment
   - Implementer applies fixes in worktree
   - Security Reviewer + Style Enforcer review in parallel
   - Loop until both pass with zero issues
6. Resolve threads via GraphQL
7. Report results
8. Self-improvement agent proposes skill/config updates → waits for your approval
```

---

## Build Features / Refactor Code → `/workflow`

**When**: "Let's build", "implement", "create", "add feature", any non-trivial coding task.

```
Phase 1: Instructions Document
   - Explore codebase, gather all context
   - Write self-contained instructions (worker can execute with zero prior context)
   - Include skill assignments per agent (via /plan + /code-styles)
   - Present to you → WAIT for approval

Phase 2: Branch & Worktree
   - Create branch: eh/<type>/<name>
   - Set up worktrees

Phase 3: Execution (via /multi-agent)
   - Assign specific skills per agent (language + framework + patterns + conventions)
   - Workers implement in isolated worktrees, self-check against hardening checklist
   - No-op check: revert any changes with no functional effect
   - Reviewers review against full checklist + assigned skills
   - Review loop: fix → re-review → repeat until PASS with zero issues

Phase 4: PR Creation
   - Push + create PR(s)
   - Present PR link(s) to you → NEVER merges
```

---

## Update & Improve Skills → `/update-skills`

**When**: "Update skills", "improve skills", "what can we learn", after a session that revealed gaps.

```
1. Analyze the session:
   - What went well / wrong / was missing / was redundant / was vague
2. Categorize: new skill needed / skill update / conflict / too broad / too narrow / workflow gap
3. Draft proposals with problem, affected skill, proposed change, priority
4. Interview you on each proposal (yes / no / modify)
5. Implement approved changes
6. Summary of what was updated
```

---

## Planning (used by other workflows) → `/plan`

**When**: Before `/workflow` Phase 3 or `/multi-agent` Phase 2 — determines skill assignments.

```
1. Analyze task: languages, frameworks, architectural concerns
2. Read /code-styles for the full skill index
3. Assign specific skills per agent based on role and task
4. Output: execution plan with agents, waves, and skill lists
```

---

## Skill Index → `/code-styles`

**When**: Need to know what style skills exist and which to assign.

| Category | Skills |
|----------|--------|
| Languages | `/typescript`, `/rust` |
| Frameworks | `/react` |
| Patterns | `/solid`, `/composition`, `/strategy-pattern`, `/factory-pattern`, `/observer-pattern` |
| Structure | `/coding-conventions`, `/file-naming`, `/code-comments`, `/simplify` |
| Index | `/design-patterns` (pattern selection guide) |

---

## Other Useful Commands

| Command | What it does |
|---------|-------------|
| `/review-pr <number>` | Review a PR against full checklists |
| `/commit` | Commit staged changes with conventional commits |
| `/sync` | Fetch, prune, clean up branches, pull latest |
| `/knip` | Find and remove unused code/dependencies |
| `/rams` | Accessibility and visual design review |
| `/simplify` | Simplify recently modified code |
| `/sync-containers` | Propagate skill/config changes to Docker sandboxes |
| `/create-claude-playground` | Create an isolated Docker sandbox for autonomous Claude Code |
| `/find-skills` | Discover installable skills |
