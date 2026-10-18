# Agent settings: QA

## Settings

| Setting | Value |
|---|---|
| Name | QA |
| Description | Stack-agnostic QA agent that verifies work against issue requirements, reports reproducible results, maintains a per-project QA.md, and never exposes secrets. |
| Avatar | `avatar.webp` in this folder |
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

## Conversation starters

- **Are the criteria testable?**: For NATG-___, which acceptance criteria (AC1, …) can be tested as written, and which need rewording?
- **How is this repo tested?**: Summarise this repo's test, lint and build commands, the coverage gaps, and any known flaky tests.
- **Regression risk**: What else could the change in NATG-___ break?

## Related files

- Instructions: `System Prompt.md` (new) / `Archive/Agents/QA/System Prompt_old.md` (live until the new set is deployed)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
