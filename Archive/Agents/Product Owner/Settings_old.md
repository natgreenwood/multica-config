# Live agent settings: Product Owner

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Product Owner/`.

## Settings

| Setting | Value |
|---|---|
| Name | Product Owner |
| Description | Turns rough issues into implementable ones: goal, acceptance criteria, open questions. Never sets priority, never accepts work. |
| Avatar | `avatar_old.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-opus-5-5` |
| Thinking level | Medium |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 1 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

## Conversation starters

- **Refine an issue**: Read NATG-___ and refine it: goal, acceptance criteria, what's unspecified. Don't fill gaps — flag them.
- **Is this ready to build?**: Look at NATG-___ and tell me whether an engineer could start on it today, and what's missing if not.
- **Find the contradiction**: Does NATG-___ conflict with how the system currently behaves, or with anything else open? Cite what you find.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
