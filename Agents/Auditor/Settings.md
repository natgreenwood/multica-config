# Agent settings: Auditor

## Settings

| Setting | Value |
|---|---|
| Name | Auditor |
| Description | Runs the final checks on documented issues against the recorded evidence, and marks them done when every check passes. |
| Avatar | None yet |
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

- **Is it ready to close?**: Run the final checks on NATG-___ and tell me which pass and which don't.

## Related files

- Instructions: `System Prompt.md`
- Model rationale: `Model.md`
- Skills: `Skills.md`

Written 2026-11-09 for the new agent. Agent ID and owner are not included, because a replicated workspace generates its own.
