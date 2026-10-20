# Live agent settings: Doc Owner

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Doc Owner/`.

## Settings

| Setting | Value |
|---|---|
| Name | Doc Owner |
| Description | Writes and Updates project documentation. |
| Avatar | `avatar_old.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | Medium |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

## Conversation starters

- **Document an issue**: Read NATG-___ with its comments and run history, then write it up under Issues and Incidents. Mark anything you can't verify as "Unknown" rather than guessing.
- **Update affected docs**: NATG-___ changed how we [deploy/authenticate/etc.]. Find every runbook, architecture, or decision page this affects and update them.
- **Draft release notes**: Draft release notes for everything done in [project] since [date]. Only include user-visible changes and link each to its issue.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
