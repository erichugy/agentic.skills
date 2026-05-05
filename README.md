# agentic.skills

Public repository for reusable agent skills.

This repo holds markdown-first skills and supporting reference files. The initial contents were synced from a local `~/.agents/skills` directory.

## Structure

- each skill lives in its own directory
- `SKILL.md` is the main entrypoint for a skill
- optional reference or support files can live alongside the skill

## Notes

- this repo is public by design
- it should avoid private prompts, secrets, logs, or machine-local state
- Botpress-specific skills (`adk/`, `adk-old/`) are listed in `.gitignore` and kept local-only — never commit them here

