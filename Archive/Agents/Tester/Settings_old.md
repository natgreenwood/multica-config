# Live agent settings: Tester

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Tester/`.

## Tester (Claude)

### Settings

| Setting | Value |
|---|---|
| Name | Tester (Claude) |
| Description | Verifies changes against what the issue actually asked, and files defects with reproductions. Never fixes anything. |
| Avatar | `avatar-claude_old.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-sonnet-5-5` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
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
| Avatar | `avatar-codex_old.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | High |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

### Conversation starters

- **How would you break it?**: For NATG-___, list the edge cases and failure paths you'd try first.
- **Go looking for trouble**: Probe ___ for defects — edge cases, error paths, boundaries. Report what reproduces and what doesn't.
- **Is this a bug or unspecified?**: ___ does ___. Tell me whether that contradicts a stated requirement or is just behaviour nobody specified.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
