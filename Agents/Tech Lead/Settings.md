# Agent settings: Tech Lead

## Settings

| Setting | Value |
|---|---|
| Name | Tech Lead |
| Description | Breaks work down, decides the approach, and reviews changes before a human merges. Does not write production code. |
| Avatar | `avatar.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-opus-5-5` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 3 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

## Conversation starters

- **Break down an issue**: Read NATG-___ and give me a breakdown: the pieces, the order to do them in, and who should own each. Flag anything you would escalate before work starts.
- **Pressure-test an approach**: Here is the approach I am considering: ___. Tell me where it breaks down, what it costs us later, and whether there is a smaller version that gets most of the value.
- **Where is it stuck?**: Look at NATG-___ and tell me where it is in the flow, what it's waiting on, and what happens next.

## Related files

- Instructions: `System Prompt.md` (new) / `System Prompt_old.md` (live)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
