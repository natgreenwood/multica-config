# Live agent settings: QA

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/QA/`.

## Settings

| Setting | Value |
|---|---|
| Name | QA |
| Description | Stack-agnostic QA agent that verifies work against issue requirements, reports reproducible results, maintains a per-project QA.md, and never exposes secrets. |
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

- **Are the criteria testable?**: For NATG-___, which acceptance criteria (AC1, …) can be tested as written, and which need rewording?
- **How is this repo tested?**: Summarise this repo's test, lint and build commands, the coverage gaps, and any known flaky tests.
- **Regression risk**: What else could the change in NATG-___ break?

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
