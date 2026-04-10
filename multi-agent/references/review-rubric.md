# Review Rubric

Use this rubric when constructing prompts for reviewer subagents.

## Instructions for Reviewer

Include these instructions in every reviewer prompt:

1. Fetch the worker's branch: `git fetch origin {branch_name}` (if needed)
2. View all changes: `git diff {base_branch}...{worker_branch}`
3. Read modified/created files in full to understand the complete picture
4. Run through ALL applicable checklists below — every item, every file
5. Evaluate against the scoring rubric
6. Output the structured review with specific **Issues to Fix** (file, line, what's wrong, how to fix)

**Quality bar**: A subsequent automated reviewer (e.g., GitHub Copilot) should find **zero** new issues after your review. If you think Copilot would flag something, flag it first.

## Checklists

### General (applies to ALL code)
See `/review-pr/references/general-checklist.md` — or include inline:

- Dead code: unused functions/variables/types/imports, dead dataset assignments, stale comments
- Type safety: `type` over `interface`, no `as` casts, explicit return types, explicit React type imports
- Comments: only "the why", no stale comments contradicting code
- Input validation: trim, max lengths, safeParse, response.json try/catch, deep validation of deserialized data
- Network: AbortController timeouts, clearTimeout in finally, cleanup on unmount
- Resources: timer unref, size caps with eviction, opportunistic cleanup
- Race conditions: double-submit guards, isMountedRef, ALL state setters guarded, stale request detection
- Errors: user-friendly, logged with context, no silent catch blocks
- Config: try/except fallbacks, clamped ranges, startup warnings
- Consistency: UTC, `??` vs `||`, case sensitivity across operators, formatting
- Functional completeness: features fully wired, IDs passed through, return values used
- Security: server-only secrets, no sensitive info in errors

### UI/Frontend (applies to React/CSS changes)
See `/review-pr/references/ui-checklist.md` — or include inline:

- Event listeners have cleanup in useEffect returns
- Drag handlers cleaned up on unmount via ref
- Dropdowns: close on outside click, handle viewport overflow, sub-menus don't escape viewport
- State wiring: features fully connected, IDs passed to handlers, return values used
- localStorage: try/catch, deep validation, graceful fallbacks
- Accessibility: aria-expanded, type=button, aria-label, keyboard navigation
- CSS: prefix convention, consistent active states, smooth transitions
- React: complete useCallback/useMemo/useEffect deps, no state updates in render, stable keys

### API/Backend (applies to route/server changes)
See `/review-pr/references/api-checklist.md` — or include inline:

- Route config: runtime, dynamic, return types, CORS
- HTTP: OPTIONS returns 204, all methods handled
- Validation: try/catch on body parsing, safeParse, trim + length cap
- Rate limiting: `||` for IP, fallback key, size caps, unref, opportunistic cleanup
- Errors: no sensitive info, user-friendly, consistent shape, correct status codes
- Env vars: server-only, 503 on missing, try/catch parsing, startup warnings

## Scoring Categories (1-5 each)

| Category | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|----------|----------|--------------|---------------|
| **Completeness** | Major pieces missing | Core task done, some gaps | Fully addresses the task |
| **Direction** | Ignores the assigned approach | Partially follows | Strongly embodies the variant |
| **Code Quality** | Bugs, broken code | Works but messy | Clean, idiomatic, no issues |
| **Consistency** | Mixed styles | Mostly consistent | Cohesive throughout |
| **Polish** | Rough, unfinished | Acceptable | Attention to detail, edge cases |
| **Hardening** | Missing timeouts, no sanitization | Some hardening | All checklist items addressed |

## Review Output Format

```markdown
### Review: {variant_name}

**Scores**:
| Category | Score |
|----------|-------|
| Completeness | X/5 |
| Direction | X/5 |
| Code Quality | X/5 |
| Consistency | X/5 |
| Polish | X/5 |
| Hardening | X/5 |
| **Total** | **X/30** |

**Checklist Results**:
- General: PASS or [list failures with file:line]
- UI: PASS or [list failures] (if applicable)
- API: PASS or [list failures] (if applicable)

**Strengths**:
- {bullet points}

**Issues to Fix** (MUST be resolved):
1. {file:line — what's wrong — how to fix}

**Verdict**: PASS (0 issues) | FAIL (issues listed above)
```
