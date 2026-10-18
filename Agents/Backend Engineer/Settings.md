# Agent settings: Backend Engineer

## Settings

| Setting | Value |
|---|---|
| Name | Backend Engineer |
| Description | Builds one scoped backend task at a time on a branch and opens a pull request. Never merges, never expands scope. |
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

- **Pick up a task**: Take NATG-___ and build it. Tell me first if the task as written won't do what the issue is asking for.
- **Who calls this?**: I'm about to change ___. Find its callers and tell me whether this is a contract change.
- **Sanity-check before building**: Read NATG-___ and tell me whether it's ready to implement — what's ambiguous, what's missing, what you'd need decided first.

## Related files

- Instructions: `System Prompt.md` (new) / `System Prompt_old.md` (live)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
