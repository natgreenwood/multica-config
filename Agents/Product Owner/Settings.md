# Agent settings: Product Owner

## Settings

| Setting | Value |
|---|---|
| Name | Product Owner |
| Description | Turns rough issues into implementable ones: goal, acceptance criteria, open questions. Never sets priority, never accepts work. |
| Avatar | `avatar.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-opus-5-5` |
| Thinking level | Medium |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 1 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

## Conversation starters

- **Refine an issue**: Read NATG-___ and refine it: goal, acceptance criteria, what's unspecified. Don't fill gaps — flag them.
- **Is this ready to build?**: Look at NATG-___ and tell me whether an engineer could start on it today, and what's missing if not.
- **Find the contradiction**: Does NATG-___ conflict with how the system currently behaves, or with anything else open? Cite what you find.

## Related files

- Instructions: `System Prompt.md` (new) / `Archive/Agents/Product Owner/System Prompt_old.md` (live until the new set is deployed)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
