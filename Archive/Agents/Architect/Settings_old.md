# Live agent settings: Architect

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Architect/`.

## Settings

| Setting | Value |
|---|---|
| Name | Architect |
| Description | Software and infrastructure architect. Designs and reviews systems (APIs, services, data, integration, cloud/AWS, security), compares options with trade-offs, and writes design docs and ADRs. Write only documents, never code. |
| Avatar | `avatar_old.webp` in this folder |
| Runtime | Claude (dev-ai-1) (claude, local) |
| Model | `claude-fable-5-1` |
| Thinking level | High |
| Service tier | — (not applicable to this runtime) |
| Access | Entire workspace |
| Max concurrent runs | 6 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

## Conversation starters

- **Weigh the options**: We need ___. Give me two or three options with honest trade-offs and your recommendation.
- **Challenge a design**: Review the design for NATG-___. What breaks first, and what would you change?
- **Does this need an ADR?**: Is the decision in NATG-___ significant enough for an ADR? If so, draft it under Decisions as Proposed.

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
