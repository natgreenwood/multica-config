# Runtimes

A runtime is a machine running the Multica daemon plus an AI coding tool. Agents are bound to a runtime and run there, so whatever an agent needs that isn't in its Multica settings has to be set up on its runtime machine: MCP servers, CLI tools, credentials. This file records that setup so it can be rebuilt.

Facts marked ✅ come from `multica runtime list` (2026-10-18, machines table updated 2026-11-10). Items marked ❓ are inferred from the prompts and skills and still need confirming on the machine.

## Machines and runtimes

| Machine | Runtime | Tool and version | Multica CLI | Daemon runs as | Used by |
|---|---|---|---|---|---|
| multica-worker-1 | Claude (multica-worker-1) | Claude Code 2.1.311 ✅ | 0.8.1 ✅ | `multica` ✅ (rebuilt 2026-11-10) | Tech Lead, Architect, Product Owner, Frontend Designer, Frontend Engineer, Backend Engineer, Tester (Claude) |
| dev-ai-3 | Codex (dev-ai-3) | codex-cli 0.166.0 ✅ | 0.8.0 ✅ | ❓ | Researcher, Doc Owner, QA, Security Analyst, Tester (Codex), Auditor, Host Ops (dev-ai-3) |
| dev-ai-1 | Claude (dev-ai-1) | Claude Code 2.1.317 ✅ | 0.8.1 ✅ | `nat` (seen on NATG-564) | Host Ops (dev-ai-1) only |

multica-worker-1 and dev-ai-3 are the squad's runtimes. All run in `local` mode with private visibility. The Claude squad agents moved from dev-ai-1 to multica-worker-1 on 2026-11-09/10. The first multica-worker-1 runtime ran as `nat` and couldn't read the Infisical token (NATG-633); it was rebuilt as `multica` (new runtime `d7534eb9`) and the old one (`af417abf`) deleted. The spare Codex runtime on dev-ai-1 is no longer listed.

**dev-ai-1 runs as `nat`.** `Setup.md` step 9 expects a dedicated `multica` user with no `sudo`, `lxd`, `docker` or `adm`. Check `id nat` there: if `nat` has any of those, the Host Ops agent on it can get root. ❓ Confirm which user dev-ai-3's daemon runs as.

A Multica CLI set up on another machine (for example to manage this repo) registers that machine as a runtime too. Ignore it unless agents are bound to it.

## Accounts

| Tool | Subscription | Signed in on |
|---|---|---|
| Claude Code | Claude Max 20x | multica-worker-1 ✅ (squad agents), dev-ai-1 ✅ (Host Ops) |
| Codex | ChatGPT Pro $100 | dev-ai-3 ✅ |

Agents on the same subscription share its weekly allowance; see `Agents/*/Model.md`.

## MCP servers

No agent has an MCP configuration in Multica (see `Agents/*/Settings.md`). The MCP servers come from the coding agent on each runtime machine.

| Runtime machine | How MCP servers are provided | Status |
|---|---|---|
| multica-worker-1 (Claude Code) | Synced from the claude.ai account Claude Code is signed in with, if `multica` signed in with the same account | ❓ check with `claude mcp list` as `multica` |
| dev-ai-1 (Claude Code) | Synced from the claude.ai account Claude Code is signed in with, so no setup is needed on the machine | ✅ user confirmed 2026-10-18 |
| dev-ai-3 (Codex) | Codex's own configuration (`codex mcp list`) | ❓ |

| MCP server | Needed by | Runtime |
|---|---|---|
| Figma | Frontend Designer, Frontend Engineer | multica-worker-1 ❓ |
| Atlassian (Confluence) | Tech Lead, Architect | multica-worker-1 ❓ |
| Atlassian (Confluence) | Researcher, Doc Owner | dev-ai-3 ❓ |
| Context7 | Architect / Researcher | multica-worker-1 / dev-ai-3 ❓ |
| Notion | None yet: documentation is Confluence only. Kept for a possible per-project Notion option | dev-ai-3 ✅ (user confirmed 2026-11-06); multica-worker-1 via the claude.ai sync, not checked |
| AWS Knowledge / AWS MCP, Microsoft Learn | Architect (named in its prompt) | multica-worker-1 ❓ |

All agents share one Atlassian account, so Confluence drafts are visible to every agent.

## CLI tools and credentials

| Tool | Why | Machines | Status |
|---|---|---|---|
| `git` | Branches and commits | multica-worker-1, dev-ai-3 | ❓ |
| `gh`, authenticated to GitHub with access to the Foodie repo | Pull requests, CI checks | multica-worker-1, dev-ai-3 | ✅ installed (user confirmed 2026-10-18) |
| Infisical CLI (`/usr/bin/infisical`) and Infisical Agent, with the pinned server and token file `scripts/secret.sh` reads | Secrets from `https://infisical.natg.co.uk`, per `Workspace/Context.md` | multica-worker-1, dev-ai-3 | ✅ in use (the containers repo's agents fetch secrets this way); ❓ service setup not yet documented from the hosts. `bws` is retired (2026-11-05) |
| Foodie's own toolchain (language runtimes, package manager, test and lint commands) | Engineers, Tester and QA run the project's checks | multica-worker-1, dev-ai-3 | ❓ |

**GitHub permissions.** Agents open pull requests but never merge. Branch protection on the Foodie repo's `main`, requiring a human review, enforces that regardless of what any prompt says. ❓ Confirm it's enabled.

## Rebuilding a runtime machine

For a full walkthrough on Ubuntu, see [`Setup.md`](Setup.md). Its step 1 installs most tools with Homebrew from [`Brewfile`](Brewfile): Claude Code, `gh`, `git`/`git-lfs`, Node.js, `uv`, the AWS and Kubernetes CLIs, `agent-browser`, and a few utilities. The Infisical CLI is commented out of the Brewfile because `scripts/secret.sh` runs only `/usr/bin/infisical`, so it comes from apt. Codex and the Multica CLI use their own installers. The Brewfile works on macOS too, but there is no macOS guide for secrets, Docker or the daemon service. In short:

1. Install the AI coding tool (Claude Code or Codex) and sign in with the subscription above.
2. Install and configure the MCP servers this machine needs (table above).
3. Install the CLI tools, authenticate `gh`, and set up the Infisical Agent with this host's machine identity.
4. Install the Multica CLI, then run `multica setup` and `multica daemon start`.
5. Confirm the runtime is `online` with `multica runtime list`.
6. Bind the agents to it, using the runtime and model from each `Agents/*/Settings.md`.
