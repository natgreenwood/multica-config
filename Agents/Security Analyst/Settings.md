# Agent settings: Security Analyst

## Settings

| Setting | Value |
|---|---|
| Name | Security Analyst |
| Description | Review proposed changes to a codebase for security issues and vulnerabilities. |
| Avatar | `avatar.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | xhigh |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 6 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level (MCP servers come from the runtime machine's own configuration) |
| Disabled runtime skills | None |

## Conversation starters

- **Review this change**: Review the pull request for NATG-___ for security issues.
- **Summarise any security findings**: Summarise all open security findings on this issue by severity.
- **Threat-check a design**: What are the security risks in the design for NATG-___?

## Related files

- Instructions: `System Prompt.md` (new) / `Archive/Agents/Security Analyst/System Prompt_old.md` (live until the new set is deployed)
- Model rationale: `Model.md`
- Skills: `Skills.md`

Exported from Multica on 2026-10-18. Agent ID and owner are not included, because a replicated workspace generates its own.
