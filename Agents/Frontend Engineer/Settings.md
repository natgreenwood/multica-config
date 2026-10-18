# Agent settings: Frontend Engineer

## Settings

| Setting | Value |
|---|---|
| Name | Frontend Engineer |
| Description | Builds one scoped frontend task at a time on a branch and opens a pull request. Never merges, never expands scope. |
| Avatar | `avatar.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-opus-5-5` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

## Conversation starters

- **Sanity-check before building**: Read NATG-___ and tell me whether it's ready to implement — what's ambiguous, what's missing, what states the design doesn't specify.
- **Explain a package's conventions**: I'm about to change ___. What are that package's actual conventions — component patterns, test command, lint setup?
- **Is the design buildable?**: Look at the Figma design for NATG-___ and tell me what's missing before it can be implemented.

## Related files

- Instructions: `System Prompt.md` (new) / `System Prompt_old.md` (live)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
