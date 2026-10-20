# Live agent settings: Security Analyst

Exported from the live Multica workspace on 2026-10-20. This is what Multica is running now; the new versions are in `Agents/Security Analyst/`.

## Settings

| Setting | Value |
|---|---|
| Name | Security Analyst |
| Description | Review proposed changes to a codebase for security issues and vulnerabilities. |
| Avatar | `avatar_old.webp` in this folder |
| Runtime | Codex (dev-ai-3) (codex, local) |
| Model | `gpt-6.1-sol` |
| Thinking level | xhigh |
| Service tier | Default (Standard) |
| Access | Entire workspace |
| Max concurrent runs | 6 |
| Environment variables | None |
| Custom CLI arguments | None |
| MCP configuration | None at agent level |
| Disabled runtime skills | None |

## Conversation starters

- **Review this change**: Review the pull request for NATG-___ for security issues.
- **Summarise any security findings**: Summarise all open security findings on this issue by severity.
- **Threat-check a design**: What are the security risks in the design for NATG-___?

## Related files

- Instructions: `System Prompt_old.md`
- Model: `Model_old.md`
- Skills: `Skills_old.md`
