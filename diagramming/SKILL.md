---
name: diagramming
description: Create diagrams with Mermaid. Use when users ask for a diagram, flowchart, sequence diagram, architecture diagram, state machine, ERD, dependency graph, or want to visualize a process or system. Prefer Mermaid fenced blocks, and when rendering is requested use Mermaid Live Editor at https://www.mermaidonline.live/editor or fall back to giving the user raw Mermaid plus the link.
---

# Diagramming

Create clear, minimal diagrams with Mermaid. Favor one diagram per concern over one oversized diagram.

In the Codex desktop app, Mermaid fenced blocks can render directly in the response. When the user wants an external rendered view or visual verification, use Mermaid Live Editor.

## When to Use

- "draw a diagram"
- "make a flowchart"
- "show the architecture"
- "visualize this process"
- "sequence diagram", "state machine", "ERD", "dependency graph"
- any request that is easier to understand visually than as prose

## Workflow

1. Identify the question the diagram must answer.
2. Pick the smallest Mermaid diagram type that fits.
3. List the key entities, states, or steps before writing code.
4. Write the Mermaid diagram.
5. Keep labels short and split dense diagrams into multiple diagrams when needed.
6. Render according to the user's need:
   - Default: return a Mermaid fenced block in the chat response.
   - If the user wants a rendered browser view: try Mermaid Live Editor.
   - If browser automation is unavailable or fails: return the raw Mermaid and tell the user to open the editor link.

## Diagram Selection

| Need | Mermaid type | Default direction |
|---|---|---|
| process, architecture, dependency flow | `flowchart` | `TD` or `LR` |
| request/response or actor interactions | `sequenceDiagram` | n/a |
| lifecycle or workflow states | `stateDiagram-v2` | n/a |
| entity relationships or schema | `erDiagram` | n/a |
| type or object relationships | `classDiagram` | n/a |
| timelines, delivery plans | `timeline` or `gantt` | n/a |

If more than one concern matters, prefer two smaller diagrams.

## Mermaid Rules

- Use stable node IDs and short readable labels.
- Use quoted labels when text contains punctuation or parentheses.
- Keep one naming scheme throughout the diagram.
- Group related nodes with `subgraph` when it improves readability.
- Avoid decorative detail that does not answer the user's question.
- When parts of the diagram are inferred rather than directly known, say so in the accompanying prose.

Read `references/mermaid-patterns.md` for templates and common notation choices.

## Mermaid Live Editor

Use [Mermaid Live Editor](https://www.mermaidonline.live/editor) when the user asks for a rendered editor view, a screenshot, or a visual verification step.

If browser tools are available:

1. Open `https://www.mermaidonline.live/editor`.
2. Replace the default editor content with the Mermaid code.
3. Wait for the preview to render.
4. If the user asked for proof or an image, capture the rendered result.

If browser tools are unavailable or fail:

- Do not keep retrying repeatedly.
- Return the Mermaid code in a fenced block.
- Include the Mermaid Live Editor link.
- Tell the user they can paste the code there to render it.

Do not assume the editor has a stable URL format for preloading code unless you verified it during the current session. Prefer opening the page and pasting the code.

## Response Pattern

1. One short paragraph stating what the diagram shows and any assumptions.
2. The Mermaid fenced block.
3. If editor rendering was requested but not completed, add the Mermaid Live Editor link.

## Quality Bar

- The diagram answers a specific question.
- The scope is narrow enough to read quickly.
- The labels are concrete rather than vague.
- The direction and notation are consistent.
- The diagram is simpler than the prose it replaces.
