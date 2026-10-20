# Live agent settings: Researcher

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Researcher/`.

## Settings

| Setting | Value |
|---|---|
| Name | Researcher |
| Description | Investigates open technical questions and writes up the options with evidence. Decides nothing, changes nothing. |
| Avatar | `avatar_old.webp` in this folder |
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

## Conversation starters

- **Investigate a question**: Research this and write it up: ___. Options with costs, not a decision.
- **How does this actually work?**: Trace how ___ works in this repo. Cite paths and line ranges; tell me what you inferred versus what you read.
- **Is this worth doing?**: We're considering ___. Tell me what it costs, what it forecloses, and whether something in the monorepo already solves it.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
