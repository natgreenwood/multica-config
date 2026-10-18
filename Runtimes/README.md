# Runtimes

A runtime is a machine running the Multica daemon plus an AI coding tool. Agents are bound to a runtime and run there, so whatever an agent needs that isn't in its Multica settings has to be set up on its runtime machine: MCP servers, CLI tools, credentials. This file records that setup so it can be rebuilt.

Facts marked ✅ come from `multica runtime list` (2026-10-18). Items marked ❓ are inferred from the prompts and skills and still need confirming on the machine.

## Machines and runtimes

| Machine | Runtime | Tool and version | Multica daemon | Used by |
|---|---|---|---|---|
| dev-ai-1 | Claude (dev-ai-1) | Claude Code 2.1.301 ✅ | 0.6.3 ✅ | Tech Lead, Architect, Product Owner, Frontend Engineer, Backend Engineer, Frontend Designer, Tester (Claude) |
| dev-ai-1 | Codex (dev-ai-1) | codex-cli 0.154.0 ✅ | 0.6.3 ✅ | None (spare Codex runtime) |
| dev-ai-3 | Codex (dev-ai-3) | codex-cli 0.162.1 ✅ | 0.6.2 ✅ | Researcher, Doc Owner, QA, Security Analyst, Tester (Codex) |

dev-ai-1 and dev-ai-3 are the workspace's normal runtimes. Both run in `local` mode with private visibility. A Multica CLI set up on another machine (for example to manage this repo) registers that machine as a runtime too. Ignore it unless agents are bound to it.

## Accounts

| Tool | Subscription | Signed in on |
|---|---|---|
| Claude Code | Claude Max 20x | dev-ai-1 ✅ |
| Codex | ChatGPT Pro $100 | dev-ai-3 ✅, dev-ai-1 ❓ (same account or a different one?) |

Agents on the same subscription share its weekly allowance; see `Agents/*/Model.md`.

## MCP servers

No agent has an MCP configuration in Multica (see `Agents/*/Settings.md`), so these come from each machine's own tool configuration.

| MCP server | Needed by | Runtime machine | Status |
|---|---|---|---|
| Figma | Frontend Designer, Frontend Engineer | dev-ai-1 (Claude) | ✅ confirmed by the user to be configured |
| Atlassian (Confluence) | Researcher, Doc Owner | dev-ai-3 (Codex) | ❓ |
| Atlassian (Confluence) | Tech Lead, Architect | dev-ai-1 (Claude) | ❓ |
| Context7 | Researcher, Architect | dev-ai-3, dev-ai-1 | ❓ |
| AWS Knowledge / AWS MCP | Architect | dev-ai-1 | ❓ (named in the Architect's prompt) |
| Microsoft Learn | Architect | dev-ai-1 | ❓ (named in the Architect's prompt) |

All agents share one Atlassian account, so Confluence drafts are visible to every agent.

To check what is configured, run these on the machine:

```bash
claude mcp list     # dev-ai-1 (Claude Code)
codex mcp list      # dev-ai-3 (Codex)
```

## CLI tools and credentials

| Tool | Why | Machines | Status |
|---|---|---|---|
| `git` | Branches and commits | dev-ai-1, dev-ai-3 | ❓ |
| `gh`, authenticated to GitHub with access to the Foodie repo | Pull requests, CI checks | dev-ai-1, dev-ai-3 | ❓ |
| `bws` (Bitwarden Secrets Manager) | Secrets, per `Workspace/Context.md` | dev-ai-1, dev-ai-3 | ❓ (stated as installed in Workspace Context) |
| Foodie's own toolchain (language runtimes, package manager, test and lint commands) | Engineers, Tester and QA run the project's checks | dev-ai-1, dev-ai-3 | ❓ |

**GitHub permissions.** Agents open pull requests but never merge. Branch protection on the Foodie repo's `main`, requiring a human review, enforces that regardless of what any prompt says. ❓ Confirm it's enabled.

## Rebuilding a runtime machine

1. Install the AI coding tool (Claude Code or Codex) and sign in with the subscription above.
2. Install and configure the MCP servers this machine needs (table above).
3. Install the CLI tools and authenticate `gh` and `bws`.
4. Install the Multica CLI, then run `multica setup` and `multica daemon start`.
5. Confirm the runtime is `online` with `multica runtime list`.
6. Bind the agents to it, using the runtime and model from each `Agents/*/Settings.md`.
