# Iterative Bot Mode

Use this mode when a bot reviewer with a confidence/quality score is the gating reviewer.

## Triggers

Use iterative bot mode when any of these are true:
- User says "iterate until Greptile is happy", "loop until 5/5", "until the bot is clean"
- The PR is gated on a bot reviewer with a confidence/quality score
- The user names a bot reviewer explicitly (Greptile, Vercel Agent, etc.)

## Loop Protocol

If running in **iterative bot mode**, wrap the entire standard process in a loop until the bot reviewer reports a clean state.

```
┌──────────────────────────────────────────────────────────────────────┐
│ A. Push & wait for bot review                                        │
│    git push                                                          │
│    gh pr checks <PR> --watch       # block until bot check completes │
│                                                                       │
│ B. Fetch bot review                                                   │
│    gh api repos/{owner}/{repo}/pulls/<PR>/reviews                    │
│    → find latest review from greptile-apps[bot] (or other bot)       │
│    → parse confidence score (e.g., "3/5", "5/5") from review body    │
│    → fetch unresolved inline comments via reviewThreads GraphQL      │
│                                                                       │
│ C. Exit conditions (stop if ANY)                                      │
│    - Confidence == max (e.g., 5/5) AND zero unresolved comments      │
│    - Iteration count >= 5 (cap to avoid runaway loops)                │
│    - User intervention requested                                      │
│                                                                       │
│ D. Otherwise, run steps 1-6 of the standard process                   │
│    → triage, no-op check, multi-agent fix, resolve threads           │
│                                                                       │
│ E. Increment iteration; go back to A.                                 │
└──────────────────────────────────────────────────────────────────────┘
```

## Parsing Confidence Scores

Bot reviews typically include a score in the review body. Use a regex search like `/(\d)\s*\/\s*5/` to extract `N/5` patterns. If multiple appear, use the last one (final summary). If no score is parseable, treat the bot's "Approved" review event as a clean signal and "Changes requested" as not-clean.

## Per-Iteration Commit Message

Use a descriptive commit message that includes the iteration number:

```
fix(<scope>): address <bot> review feedback (iteration N)
```

## Output Format

After the loop exits, replace the standard report with the iterative summary:

```markdown
## PR #<N>: <title> — Iterative <bot> mode

| Field             | Value      |
|-------------------|------------|
| Iterations        | N          |
| Final confidence  | X/5        |
| Comments resolved | M          |
| Remaining         | K          |

Status: ✓ Clean / ⚠ Stopped at iteration cap / ✗ User intervention

Remaining issues (if K > 0):
  - <path>:<line> — "<comment excerpt>"
  - ...
```

If the loop exited due to the iteration cap, list the remaining issues and surface a recommendation: a remaining comment after 5 iterations is usually a false positive, a design disagreement, or a structural change that needs human input — flag it for the user rather than churning further.
