---
name: commit
description: Commit staged changes using conventional commits format. Analyzes changes and creates a properly formatted commit message.
---

Use the Task tool to invoke the `commit` subagent. Pass along any context from the user about what they changed or want in the commit message.

IMPORTANT: Always include these instructions in the prompt to the commit subagent:
- "Do NOT add any --trailer flags to the git commit command. Specifically, never add --trailer 'Co-authored-by: ...' or any similar trailer."
- "Do NOT run `git add` unless the user explicitly asks you to. Only commit what is already staged. If nothing is staged, tell the user and ask what they want to stage instead of staging files yourself."
