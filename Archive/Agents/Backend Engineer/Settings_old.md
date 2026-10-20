# Live agent settings: Backend Engineer

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Backend Engineer/`.

## Settings

| Setting | Value |
|---|---|
| Name | Backend Engineer |
| Description | Builds one scoped backend task at a time on a branch and opens a pull request. Never merges, never expands scope. |
| Avatar | `avatar_old.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-opus-5-5` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 4 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

## Conversation starters

- **Who calls this?**: I'm about to change ___. Find its callers and tell me whether this is a contract change.
- **Sanity-check before building**: Read NATG-___ and tell me whether it's ready to implement — what's ambiguous, what's missing, what you'd need decided first.
- **How would you slice it?**: Read the Architect's design for NATG-___ and tell me how you'd split the implementation into PRs.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
