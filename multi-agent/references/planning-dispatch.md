# Planning and Dispatch

Detailed Phase 1 and Phase 2 protocol for `/multi-agent`.

## Phase 1: Plan

**Use the `/plan` skill** to create the execution plan. The planner determines:
- Which agents are needed and their roles
- **Which specific skills each agent should leverage** (read `/code-styles` for the full skill index)
- Wave ordering and dependencies

1. Parse the user's request. Extract:
   - The core task description
   - Explicit variant count or directions (if provided)
   - Constraints (files to touch/avoid, frameworks, etc.)
   - The current base branch name (run `git branch --show-current`)
   - **Dependencies between workers** (does any worker's task require another's output?)
   - **Languages and frameworks involved** (determines which style skills to assign)

2. **Read `/code-styles`** to determine which style skills to assign to each agent role. Every worker and reviewer brief must include their specific skill assignments.

3. If variants are not explicitly provided, generate 2-4 variant directions. Each needs a short name + 1-2 sentence description.

4. Identify dependencies. A worker **depends on** another when it needs that worker's committed output to start (e.g., worker B builds a UI on top of an API that worker A creates). Independent workers run in parallel; dependent workers wait and branch off their prerequisite's branch.

5. Present the plan to the user (including skill assignments per agent):

```
| # | Variant | Direction | Depends On | Wave |
|---|---------|-----------|------------|------|
| 1 | api | Build REST endpoints for user profiles | — | 1 |
| 2 | ui | Build profile page consuming the API | #1 (api) | 2 |
| 3 | tests | Add e2e tests for the profile flow | #2 (ui) | 3 |
| 4 | docs | Write API documentation | #1 (api) | 2 |
```

Workers in the same wave launch in parallel. A higher wave launches only after all its dependencies in earlier waves complete.

6. Ask: **"Ready to launch N workers (M waves)? Adjust variants, dependencies, or say 'go'."**
7. **Wait for confirmation before proceeding.** Do not launch workers without approval.

## Phase 2: Dispatch Workers

Read `references/worker-briefs.md` for the brief template before writing prompts.

1. Determine branch naming: `multi-agent/<task-slug>/<variant-slug>`

2. **Dispatch by wave.** Process one wave at a time:

   **Wave 1** (no dependencies — branches off base branch):
   ```
   Agent(
     subagent_type: "general-purpose",
     isolation: "worktree",
     run_in_background: true,
     description: "<variant-name> worker",
     prompt: <worker brief from template>
   )
   ```
   Launch all Wave 1 workers in a **single message** for true parallelism.

   **Wave 2+** (has dependencies — branches off a prerequisite's branch):
   - Wait for all prerequisite workers to complete successfully
   - Get the prerequisite's **branch name** from its worktree result
   - Include in the worker's brief: the prerequisite branch to base off of
   - The worker brief must instruct: `git fetch origin && git checkout -b <new-branch> <prerequisite-branch>`
   - Launch all workers within the same wave in a single message

3. Track worker state:

```
| Variant | Wave | Depends On | Branch Base | Task ID | Status |
|---------|------|------------|-------------|---------|--------|
| api | 1 | — | main | (id) | running |
| ui | 2 | api | multi-agent/profile/api | (id) | waiting |
| docs | 2 | api | multi-agent/profile/api | (id) | waiting |
| tests | 3 | ui | multi-agent/profile/ui | (id) | waiting |
```

4. As each wave completes:
   - Record branch names from worktree results
   - Update dependent workers' branch base to the actual branch name
   - Launch the next wave
   - If a prerequisite **fails**, mark all its dependents as `blocked` and report to user. Offer to retry or skip.

5. Wait for each wave to complete before launching the next. You will be notified as each worker finishes. Do not poll.
