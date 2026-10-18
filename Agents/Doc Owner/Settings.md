# Agent settings: Doc Owner

## Settings

| Setting | Value |
|---|---|
| Name | Doc Owner |
| Description | Writes and Updates project documentation. |
| Avatar | `avatar.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | Medium |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

## Conversation starters

- **Document an issue**: Read NATG-___ with its comments and run history, then write it up under Issues and Incidents. Mark anything you can't verify as "Unknown" rather than guessing.
- **Update affected docs**: NATG-___ changed how we [deploy/authenticate/etc.]. Find every runbook, architecture, or decision page this affects and update them.
- **Draft release notes**: Draft release notes for everything done in [project] since [date]. Only include user-visible changes and link each to its issue.

## Related files

- Instructions: `System Prompt.md` (new) / `System Prompt_old.md` (live)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
