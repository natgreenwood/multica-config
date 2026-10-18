# Agent settings: Tester

Two Multica agents share this folder's prompt and differ only in runtime and model.

## Tester (Claude)

### Settings

| Setting | Value |
|---|---|
| Name | Tester (Claude) |
| Description | Verifies changes against what the issue actually asked, and files defects with reproductions. Never fixes anything. |
| Avatar | `avatar-claude.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-sonnet-5-5` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

### Conversation starters

- **How would you break it?**: For NATG-___, list the edge cases and failure paths you'd try first.
- **Go looking for trouble**: Probe ___ for defects — edge cases, error paths, boundaries. Report what reproduces and what doesn't.
- **Is this a bug or unspecified?**: ___ does ___. Tell me whether that contradicts a stated requirement or is just behaviour nobody specified.

## Tester (Codex)

### Settings

| Setting | Value |
|---|---|
| Name | Tester (Codex) |
| Description | Verifies changes against what the issue actually asked, and files defects with reproductions. Never fixes anything. |
| Avatar | `avatar-codex.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | High |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

### Conversation starters

- **How would you break it?**: For NATG-___, list the edge cases and failure paths you'd try first.
- **Go looking for trouble**: Probe ___ for defects — edge cases, error paths, boundaries. Report what reproduces and what doesn't.
- **Is this a bug or unspecified?**: ___ does ___. Tell me whether that contradicts a stated requirement or is just behaviour nobody specified.

## Related files

- Instructions: `System Prompt.md` (new) / `Archive/Agents/Tester/System Prompt_old.md` (live until the new set is deployed)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
