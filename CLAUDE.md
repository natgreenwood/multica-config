# CLAUDE.md — multica-config

Source of truth for the configuration of a Multica (multica.ai) workspace: agent instructions and squad instructions, kept as Markdown. The user copies these files to the Multica server by hand. Nothing here deploys automatically.

## Working rules (from the user — non-negotiable)

- **Do not edit any config Markdown (`Agents/**`, `Squads/**`) without the user reviewing the change first.** Propose the change as a diff or snippet, wait for approval, then apply.
- **Present changes in small pieces**: one file, or one section, at a time.
- This file (CLAUDE.md) is project tracking and may be updated as needed.
- **Confluence is the only documentation and research store** (decided 2026-10-18). Notion was the old solution: never reference it in new or revised files. Research output goes in the project Confluence space, Research folder.
- **Research and planning are Confluence-only, with no repo index** (decided 2026-10-18). Drop the `docs/research/[KEY].md` index PR: it required human merges, and with PR-merge-status = `documentation` it would push research issues to `documentation`. All agents share one Atlassian account, so Draft pages are visible to every agent. The Researcher posts the page link in its issue comment, and that comment is the review signal.
- **The Tech Lead is used in chat too** (only about Dev Squad 1 work). Chat runs get only the System Prompt and Workspace Context: no squad instructions and no squad protocol. The Tech Lead prompt must therefore stand alone; see the chat rule in its PERMISSIONS.
- Commit or push only when the user asks. Branch off `main` first (the repo uses PRs, e.g. `CHORE--...`).

## Repo layout

```
Archive/Agents/<Agent Name>/System Prompt_old.md -> LIVE agent instructions (what the server runs today)
Agents/<Agent Name>/System Prompt.md     -> NEW agent instructions being written (blank as of 2026-10-17)
Archive/Squads/Dev Squad 1/Instructions_old.md   -> LIVE squad instructions
Squads/Dev Squad 1/Instructions.md       -> DRAFT proposal for the new squad instructions (not deployed)
Workspace/Context.md                     -> DRAFT for Workspace Settings → Context (seen by EVERY agent on every run)
Agents/<Agent Name>/Model.md             -> recommended runtime / model / thinking level (settings reference ONLY; never pasted into a prompt)
Skills/<skill-name>/SKILL.md             -> Multica workspace skills; import from the GitHub URL so "update from source" works
Agents/<Agent Name>/Skills.md            -> skill assignments for the agent, each with Attached/Removed/Not attached and the reason
Agents/<Agent Name>/Settings.md          -> every item on the agent's Settings tab, exported from Multica; together with avatar*.webp it lets someone rebuild the agent
Squads/Dev Squad 1/Members.md            -> the squad's leader and members with their NEW roles (roles feed the leader's roster); live roles are in Archive/.../Members_old.md
Autopilots/<Name>/Runbook.md             -> the autopilot's description (its run prompt); Settings.md holds mode, assignee, schedule and subscriber
Archive/Agents/<Agent Name>/{Settings,Model,Skills}_old.md + avatar*_old.webp -> LIVE agent config exported from Multica, mirroring Agents/
Archive/Workspace/{Settings,Context}_old.md -> LIVE workspace settings (prefix, repos, PR merge status, MCP server library names) and Workspace Context
Workspace/Issue Status.md + Archive/Workspace/Issue Status_old.md -> the workspace's issue statuses (display name, description, group), from Workspace Settings → Issue Statuses (supplied by the user)
```
`Workspace/Context.md` holds rules every agent needs. Keep it short, because its tokens go to every agent on every run. Not yet deployed. If the server field already has content, merge it rather than pasting over it.
Convention: `*_old.md` = live and deployed, kept under `Archive/` (moved there 2026-10-18), with a folder structure that mirrors the repo root. The un-suffixed file = the new version, which the user deploys once it is approved. **Until the new set is deployed, the files in `Archive/` are still what Multica runs.** After deployment they become history only.

**DEPLOYED 2026-10-24:** the new set (11 `System Prompt.md` → 12 agents, squad `Instructions.md`, `Workspace/Context.md`, `Members.md` roles) is now live. `Archive/` is history and the rollback source: re-push the `_old` files, and restore statuses from `Archive/Workspace/Open Issues_old.md`.

**Earlier state (2026-10-17):** `Instructions_old.md` is live. `Instructions.md` is the user's new draft, written to introduce a status-driven flow (Research → Refinement → Implementation → Testing → Documentation, plus custom statuses). The live agent prompts (`System Prompt_old.md`) were written against the live squad instructions. **Update 2026-10-18:** all 11 new `System Prompt.md` files are now drafted against the new squad flow. Together with the draft `Instructions.md` and `Workspace/Context.md`, they form one set to deploy together.

Agents: Architect, Backend Engineer, Doc Owner, Frontend Designer, Frontend Engineer, Product Owner, QA, Researcher, Security Analyst, Tech Lead, Tester.
Squads: Dev Squad 1 (leader is presumably Tech Lead; to be confirmed).

---

## Multica knowledge (condensed from multica.ai/docs and the multica-ai/multica source, Oct 2026)

### Core objects
- **Workspace**: the container for everything.
- **Issue**: a unit of work with title, description, status, priority, assignee (member, agent or squad), labels, project, parent/sub-issues, and an activity log.
- **Project**: groups issues. It can bind resources (GitHub repos, local directories) and has a description, and both are injected into runs.
- **Agent**: a reusable config made up of name, description, **instructions**, model/thinking level, runtime, skills, access, env vars, CLI args, MCP config and a concurrency limit (default 6).
- **Skill**: a reusable `SKILL.md` (plus optional files) describing *how* a type of work is done. Skills are attached to agents many-to-many.
- **Squad**: one **leader agent** plus members (agents or humans). Work assigned to the squad goes only to the leader.
- **Runtime/daemon**: a machine with the AI coding tool (Claude Code, Codex, etc.) that claims and executes runs.
- **Run (task)**: one execution. States: `deferred → queued → dispatched → running → completed | failed | cancelled` (plus `waiting_local_directory`).
- **Autopilot**: a cron or webhook trigger with a **Runbook** prompt. It either creates an issue or does a run-only execution.

### What reaches the model (important for writing prompts)
- **Agent instructions** are used on *every* run, including issues, mentions, chat and autopilot.
- **Agent description** never enters the prompt. It is UI only.
- **Squad description** has no prompt impact. **Squad instructions** go **only to the leader**, never to members.
- At run time the daemon writes a "runtime brief" into the working directory as `CLAUDE.md` (Claude), or `AGENTS.md` (Codex and most others), inside marker blocks. The brief's sections, in order:
  header → background safety → **agent identity (instructions)** → requesting user → workspace context (workspace system prompt) → available CLI commands → issue/comment formatting → repositories → project context → instruction precedence → workflow (chat / issue / autopilot) → sub-issue creation → **skills** → mentions → attachments → "always use CLI" → output.
- **Squad leader briefing** is appended on every leader turn and has 3 blocks:
  1. **Squad Operating Protocol** (hard-coded). It covers: read the issue; delegate by @mention; be terse and don't restate the issue; record `multica squad activity <issue> action|no_action|failed --reason "..."` every turn; stop after dispatching; on `no_action` exit silently with no comment; never do the work yourself; one delegation comment per turn; don't both @mention *and* create a `todo` child issue assigned to the same agent (that runs it twice).
     - If the squad owns the issue: the leader sets `in_progress` on first dispatch and `in_review` when the goal is met, and leaves `done` to humans or integrations.
     - If the leader was pulled in by @mention on someone else's issue, it must never change status.
  2. **Squad Roster**: each member with exact mention markdown.
  3. **Squad Instructions (<name>)**: our `Squads/*/Instructions.md`. This means squad instructions should hold *routing rules, norms and background*, not repeat the protocol.

### Triggers
- Four triggers: **assign** (only when status ≠ `backlog`), **@mention** in a comment (assignee/status unchanged), **chat** (each message is one run), **autopilot**.
- Mentions only fire with the exact markdown `[@Label](mention://<type>/<uuid>)`, where type is `agent`, `member` or `squad`. A plain `@name` does nothing. UUIDs come from `multica agent list` / `squad list` / `workspace member list --output json` (members use `user_id`).
- A plain comment with no mention routes by context: replying to an agent's comment hands the run to that agent.
- **Squad leader re-wakes** on non-member comments and on member updates *without* an @mention. It does **not** wake when a member @mentions another agent (the leader yields), or on its own comments.
- A comment that @mentions a squad goes to the leader only (no fan-out).
- Agent↔agent ping-pong is de-duplicated but **not stopped** by the system. **Stop conditions must be written into instructions.**
- New comments merge into an agent's already-queued run on the same issue.
- Changing assignee or status does **not** stop a running run. Runs must be stopped from the execution log.

### Issue statuses
- Built-in: `backlog`, `todo` (Unstarted); `in_progress`, `in_review`, `blocked` (Started); `done`; `cancelled`.
- **Custom statuses** can be added by owners. They inherit a category, which is fixed once created, but **none of the built-in behaviours**.
- System behaviours: a failed run reverts `in_progress → todo`; when every PR linked to an issue has merged, the issue moves to the workspace's PR-merge status (default `done`; see the PR-linking note below); `backlog` never triggers runs.
- Sub-issues can have **stages** (1, 2, 3…). When the earliest unfinished stage is all done or cancelled, the parent is notified (the leader wakes on the parent).
- A child issue created with `--status todo` and an agent assignee triggers that agent immediately.

### Agent-side CLI (what agents use during runs)
```
multica issue get <id>
multica issue list [--status --assignee --project ...]
multica issue create --title ... [--description --status --priority --assignee --parent]
multica issue update <id> [...]
multica issue status <id> <status>
multica issue assign <id> --to <name>
multica issue comment list <id> [--thread <cid> --tail N | --recent N | --since <ts>]
multica issue comment add <id> --content "..." [--parent <cid>]
multica squad activity <issue-id> action|no_action|failed --reason "..."
multica agent list | squad list | workspace member list --output json
```

### Writing good instructions (docs guidance)
Instructions should cover: what the agent is and isn't responsible for, what to check first, what it may modify, how to deliver results, and when to stop and check with a human. Skills hold reusable *method*, and instructions hold *identity and boundaries*. Put stop conditions for agent-to-agent loops in the instructions.

### Verified in source (github.com/multica-ai/multica @ `18fa9f7`, 2026-10-16)
Files to re-check: `server/internal/daemon/execenv/runtime_config_sections.go` (runtime brief), `server/internal/handler/squad_briefing.go`, `server/internal/service/builtin_skills/multica-platform/references/*.md` (the platform skill every agent can load).

- **Where our agent instructions land.** They go verbatim under `## Agent Identity`, after `**You are: <Agent Name>** (ID: ...)`. So a prompt that starts with its own `# Heading` ends up as an H1 nested inside an H2 section. That's harmless, but it's redundant with the name line.
- **Workspace Context** (Workspace settings → "context") is a per-workspace system prompt injected into *every* agent's brief as `## Workspace Context`. Shared rules belong there rather than being copied into each agent: branch naming, secrets (`bws`), monorepo conventions, Confluence space.
- **On squad-leader runs, the squad briefing is part of Agent Identity** (`handler/daemon.go`: the briefing is appended to the agent's instructions). That makes it System Prompt + Squad Operating Protocol + roster + squad Instructions in one block, so any duplication between the leader's System Prompt and the squad instructions is paid twice. There is no stated precedence *within* that block, so state permissions explicitly in the System Prompt. Chats, direct agent @mentions and possibly wakeup-started runs get the System Prompt only.
- **Instruction precedence** (issue runs): "Agent Identity instructions have priority over the issue workflow… skip the conflicting action." Agent instructions can therefore *forbid* status writes, comments, delegation and similar actions. That is the lever for controlling built-in behaviour.
- **The built-in issue workflow is given to every agent on every issue run:**
  1. Read the issue.
  2. Scan all comment threads.
  3. Set `in_progress` *first* if the turn produces any of the issue's own deliverable.
  4. Post results as a comment. This is the only delivery channel, because terminal output is not seen.
  5. Leave the status accurate: delivered → `in_review`, continuing → `in_progress`, stuck → `blocked`, Q&A only → no write.

  This applies **"whoever the assignee is"**. A squad member that finishes its step will therefore move the parent to `in_review` unless its own instructions say otherwise. When custom statuses exist, the brief adds that these rules refer to exact built-in keys.
- **Custom statuses** carry lifecycle category only, with none of the built-in behaviours (backlog parking, In Review completion, Blocked failure, In Progress recovery). Moving `backlog` → *any* non-terminal status enqueues the assigned agent, so `to_do` triggers just like `todo`. A failed run rolls back only `in_progress` → `todo`.
- **Mention types.** `agent` → runs that agent. `squad` → runs the leader only. `member` → link only, **no run and no notification**. `issue` → link only. `[@all](mention://all/all)` is a broadcast that *suppresses* the assignee's implicit on-comment trigger; explicit agent/squad mentions in the same comment still fire. A wrong UUID fails silently (`invocation_not_allowed`).
- **Issue wakeups exist** (`multica issue wakeup create <issue> ...`). They persist on the issue and start a later ordinary run.
  - Conditions: `--until-status KEY` (custom keys work), `--until-pr checks|merged`, `--until-children-done [--stage N]`, `--until-issue X --until-issue-state done|ended|in_review`, `--until-assignee`, `--until-label`, `--until-property`.
  - Timers: `--kind at|every|cron`.
  - Options: `--instruction`/`--instruction-file`, `--expires-in`, `--on-timeout wake`, `--max-fires`.
  - A "wake on any status change" rule is `--kind event --event issue.status_changed --mode continuous`, optionally with `--filter-actor-type member|agent --filter-actor-id <id>`.
  - Rules live **on the issue** and outlive the run that created them. Limits:
    - Continuous rules default to `--max-fires 20` (max 1000).
    - A rule pauses when its trigger chain passes through it a 3rd time without a person in between (`paused_reason=loop`), or after 12 runs in an hour (`rate`). A paused rule stays off until someone re-enables it.
    - `done`/`cancelled` disables a rule permanently; reopening the issue doesn't restore it.
    - The registering agent's own status changes and comments never wake it.
  - **Verified 2026-10-24 (NATG-381):** a wakeup run for the squad leader carries the full squad briefing.
- **Project workflows** (in source, not yet in the public docs). A project can define ordered status columns where entering a column **reassigns the issue to that column's handler** (an agent, squad, member, project lead or creator) and starts their run. A column may also name a "needs changes" status. A named workflow rejects statuses outside it, and only the assignee gets the step's instructions. **This could replace much of the status-routing logic currently written into the Dev Squad 1 instructions.** Availability on our server version is unconfirmed.

- **Status changes don't wake anyone** (`handler/issue.go` `shouldEnqueueAgentTask`). The exception is leaving the fixed `backlog` key, which enqueues the assigned agent. A human moving `refinement_complete → implementation` wakes nobody on its own. Something has to catch it: a comment or @mention, an `issue wakeup --until-status`, or a project-workflow handoff.
- **PR merge status is configurable** (workspace setting `pr_merge_status`, `handler/pr_auto_complete.go`). It decides what status an issue moves to when all its linked PRs merge. The default is `done`, and `none` leaves the status alone. It can be set to a custom key, such as `documentation`. With the default, a merge sends the issue straight to `done` and skips any post-merge steps.

### PR linking and merge status (checked in source 2026-10-18; server behaviour inferred from its settings)
- **What links a PR to an issue:** the key in the PR **title**, in the **branch** name, or right after a closing keyword ("Closes KEY") in the title or body. A bare mention in the body links nothing.
- **What moves the issue:** in current source (`handler/pr_auto_complete.go`), "the keyword only links; it has no say in what the merge does". When **every** linked PR has merged, the issue moves to the workspace's PR-merge status, however each PR was linked.
- **Conflict with the live prompts:** they claimed a key only in the title or branch "links without moving the issue on". The user's server can set the merge target to `documentation`. That choice came with the same change (MUL-7726) that made any link count; older servers only had an on/off switch that moved issues to `done`. So the server almost certainly follows the current-source behaviour, and the old engineer wording was out of date.
- **Rule that works either way** (in the `pull-requests` skill):
  - A delivering PR has the key in its title *and* "Closes KEY" in its body.
  - A supporting PR carries the key nowhere, because a linked PR delays the move until it merges too.

### Skills (`Skills/`, 2026-10-18)
- **`confluence-pages`**, attached to the Researcher, Architect, Doc Owner and Tech Lead. Covers draft vs published status, space and parent, reading pages back, search before creating, and what to do when Confluence is unreachable.
- **`pull-requests`**, attached to both Engineers and QA. Covers linking a delivering PR, and keeping a supporting PR unlinked.
- **Prompt pointers:** each prompt keeps a one-line pointer plus its critical rule, in case the skill isn't loaded.
- **User actions:** import each skill from its GitHub URL (the repo is public), attach it as above, and deploy it with the new set.

### Confluence (checked via Atlassian MCP 2026-10-18)
- Site `natgreenwood.atlassian.net`, cloudId `b76a4ca6-c903-4c4d-abdb-df3444a06d06`.
- The space `Foodie` has the folders `Research` (id 524516) and `Planning` (id 1081666). Prompts say "the project's Confluence space" generically, because each project has its own space.

### Docs index
Key pages: `/docs/concepts`, `/docs/how-multica-works`, `/docs/agents`, `/docs/agents-create`, `/docs/skills`, `/docs/squads`, `/docs/triggering-agents`, `/docs/mentioning-agents`, `/docs/tasks`, `/docs/issues`, `/docs/autopilots`, `/docs/project-resources`, `/docs/cli`.
Context7 IDs: `/multica-ai/multica` (source, includes the runtime brief and squad briefing code), `/websites/multica_ai` (docs site).

---

## Observations to review with the user (not yet actioned)

Items 2, 3, 5 and 10 concern the **draft** `Instructions.md`. Items 4, 7 and 8 also apply to the **live** `Instructions_old.md` (it has the same tester, "assign to Dev Squad 1" and admin/secrets sections). Notion vs Confluence (item 4 in the review): resolved 2026-10-18, Confluence only. The live Researcher and Doc Owner prompts already use Confluence.

1. ~~Architect filename typo ("Sytem Prompt.md")~~ **Done 2026-10-17.** Renamed to `System Prompt.md` with `git mv`.
2. **Partly resolved 2026-10-18:** use the built-in `todo` (`to_do` was a mistake), and a single `testing` status covers Tester → Security Analyst → QA. Original note: `Squads/Dev Squad 1/Instructions.md` uses many custom statuses (`to_do`, `research`, `needs_refinement`, `refinement_complete`, `implementation`, `testing`, `documentation`). These need to exist on the server.
   - `to_do` vs the built-in `todo`: the assign trigger keys off "not backlog", but this should be confirmed.
   - Custom statuses don't inherit built-in behaviours. For example, a failed run reverts to `todo`.
3. Squad Instructions line 3 is an unfinished sentence: "the issue is returned to".
4. Squad Instructions reference "Tester Claude" and "Tester Codex", but the repo has a single `Agents/Tester`. Need to confirm whether two server agents share this prompt.
5. Some squad instructions duplicate or conflict with the hard-coded Squad Operating Protocol:
   - "Tech Lead may mark Done" vs the protocol's "leave `done` to a human".
   - "After an agent completes its task it will assign the issue back to Dev Squad 1": reassigning re-triggers the leader, which overlaps with the automatic wake on member comments.
6. ~~Tech Lead "register a wakeup… when status becomes documentation": does Multica support it?~~ **Yes.** The command is `multica issue wakeup create <issue> --until-status documentation`. The prompt could name the command explicitly.
7. **Status-write conflict.** The built-in workflow tells every agent to set `in_progress`/`in_review` itself. None of the member agents' instructions restrict this, so after (say) Researcher finishes, it will likely move the issue to `in_review` and skip the squad's custom flow. Each member agent probably needs a line stating which statuses it may write, or that it must not write status.
8. **Shared rules.** Branch naming and the `bws` secrets note live only in the squad instructions, which only the leader sees, so engineers never see them. Candidates for Workspace Context.
9. **Project workflows** may be a better fit than prose routing for the Research → Refinement → Implementation → Testing → Documentation flow. Need to check whether our server version has them.
10. **Addressed in the draft 2026-10-17** (ROUTING rewritten; member side tracked under Pending follow-ups). Leader wakes: a squad member's reply already wakes the leader. The rule "assign back to Dev Squad 1" re-triggers it again, so it is likely redundant or the cause of double runs.

## Pending follow-ups (carry into later stages)

**Tech Lead `System Prompt.md`: DONE 2026-10-18 (draft, 1,348 words).** All earlier follow-ups are covered by its WHAT YOU MAY DO ON THE PLATFORM section:
- wakeups
- every squad status, with an explicit override of the default `in_progress` progression
- staged sub-issues and promotion out of `backlog`
- assigning to the Product Owner
- `done` per ISSUE DONE, overriding "leave done to a human"
- a guard for issues not assigned to the squad
- the chat rule

It also:
- publishes approved research pages itself
- leaves rejected drafts for a human to archive
- leaves its own Planning pages for a human to publish

The live "Leave done to a human" and wakeup lines were dropped.

**Researcher `System Prompt.md`: DONE 2026-10-18 (draft, 1,544 words).** Built from `_old`:
- The index PR is gone, along with all repo writes.
- The issue comment links the Draft page and is the Tech Lead's review signal.
- The Tech Lead publishes the page.
- On squad issues it never changes status or assignee. A research issue assigned to it directly, outside the squad, keeps the live behaviour (`in_review`).
- FAILURE NOTICES is removed (now in Workspace Context).

**Every member agent's new `System Prompt.md` (all except Tech Lead): DONE 2026-10-18.** All 10 member prompts carry these rules:
- The member posts its result as a comment on the issue, and that comment wakes the Tech Lead.
- It doesn't reassign the issue. **Exception:** the Product Owner hands back after refinement.
- It doesn't change status, explicitly overriding Multica's default workflow. **Exception:** the Researcher and Doc Owner keep their live behaviour for issues assigned to them directly, outside the squad.

**Implementation and testing batch (2026-10-18, branch `feat/member-system-prompts`):**
- **Frontend and Backend Engineers:**
  - The PR carries the key in its title and "Closes KEY" in its body (since 2026-10-18 this lives in the `pull-requests` skill). Merging moves the issue to `documentation`. On sub-issues, use the sub-issue's key.
  - No "move to Testing and assign" step any more.
  - Separate issues go in `backlog`.
  - The Frontend Engineer builds from the Frontend Designer's **Figma** design; the Backend Engineer builds from the Architect's design. Both stop if no design is linked.
- **Tester:** the `in_review` and reassign lines are removed. Defects are still filed in `backlog`, assigned to `Dev Squad 1`.
- **QA:**
  - Pass, Fail and Blocked are reported as a *verdict* in the comment, with no status changes and no closing issues.
  - The test plan is folded into the report (user decision A: one comment, one Tech Lead wake).
  - A separate QA.md PR must not carry the issue key anywhere (`pull-requests` skill).
  - The H1 is removed.
- **Frontend Designer:**
  - **Figma is the medium** (user decision; a Figma MCP is configured on the agent).
  - Handoff notes go in Figma and are summarised in the comment, with no @mention of the Frontend Engineer.
  - A "can't reach Figma" honesty line is added.
  - The H1 is removed.
- **Security Analyst:** the comment is always the hand-back, because the Tech Lead needs APPROVED verdicts too, given mandatory review before `in_review`. It never reassigns.

**Product Owner `System Prompt.md`: DONE 2026-10-18 (draft, 1,475 words).** Built from `_old` with three amended passages and a new IN DEV SQUAD 1 section:
- It owns refinement while assigned and never changes status, overriding the default workflow.
- It hands back to `Dev Squad 1` with a *ready* or *blocked* comment. "Blocked" means a question nobody in the conversation can answer.
- **Decision 2A:** if a person stops replying, the issue stays in `needs_refinement`. No chasing and no wakeup.
- It appends the refinement block when the issue is in `needs_refinement`, and comments instead when someone is already building.
- **Decision 1A:** it may @mention the Architect once per set of questions, with no acknowledgement replies. These are the loop stop conditions.

**Architect `System Prompt.md`: DONE 2026-10-18 (draft, 900 words).** Built from `_old`:
- Designs and ADRs are Confluence Drafts (user decision A): a design is a child of the **Architecture** page; an ADR goes under **Decisions** with status *Proposed*. Nothing goes in the repo.
- It doesn't dispatch, @mention engineers or create issues; its brief goes to the Tech Lead.
- It never changes status or assignee.
- It answers the Product Owner's refinement questions once, in thread, with no @mention back.
- Skills: `architecture` + `system-design` (both attached in Multica, both from `anthropics/knowledge-work-plugins`). Their "action items / create tasks" output is treated as a proposal only.
- Mermaid renders in Confluence through the `narva.net.mermaid-diagrams` app.

**Doc Owner `System Prompt.md`: DONE 2026-10-18 (draft, 1,538 words).** Built from `_old` with a new "In Dev Squad 1" section:
- It works in `documentation`, and its comment wakes the Tech Lead.
- It never changes status or assignee.
- It finalises and publishes the Architect's Proposed ADRs and design drafts.
- It leaves Research and Planning pages to their publishers.

Other changes: Research/Planning were added to the space structure, follow-up issues go in `backlog`, the output goes in the issue comment, and FAILURE NOTICES and the `# Docs Agent` H1 were removed.

**Workspace Context (`Workspace/Context.md`):**
- Done 2026-10-18: DOCUMENTATION section (Confluence only, Research folder, no Notion).
- Possible trim later: once Context is live, remove the duplicate "Research output is stored…" line from RESEARCH in the squad `Instructions.md`.
- Note: branch names and `bws` were moved here from the squad instructions on 2026-10-18.

- Done 2026-10-18: FAILURE NOTICES, including the member-side session-limit rule.

**All new agent `System Prompt.md` files:**
- Do NOT include a FAILURE NOTICES section. It lives in Workspace Context now; the 8 live `_old` prompts each carry an identical copy.
- An agent with a named fallback (only the testers: Tester Claude ↔ Tester Codex) may use it **once** per failure. Decided 2026-10-18.

### Models and subscriptions (decided 2026-10-18)
- **Subscriptions:** Claude Max 20x and ChatGPT Pro $100 (5× Plus). The limit that matters is weekly allowance, not dollars.
  - Fable 5.1 may use at most 50% of the weekly Claude allowance.
  - Pro $100 has no 5-hour cap but does have a weekly limit.
- **Per-agent choices** are in `Agents/*/Model.md`. In summary:
  - **Claude:**
    - Fable 5.1: the Architect.
    - Opus 5.5: Tech Lead, Product Owner, both Engineers and the Frontend Designer.
    - Sonnet 5.5: Tester Claude.
  - **Codex (GPT-6.1 Sol):** Researcher, Doc Owner, QA, Security Analyst (xhigh) and Tester Codex.
- **Design principles:**
  - Spend capability where mistakes are costly or hard to spot.
  - Keep reviewers on a different provider from the engineers.
  - Keep both providers in use, so one exhausted allowance doesn't stop testing.
  - Known single point of failure: the Tech Lead is on Claude.
- **Sources:**
  - The Claude model facts come from Anthropic's reference.
  - The OpenAI lineup comes from third-party sources (GPT-6.1 Sol is Codex's default since 2026-09-29; GPT-6 Luna is the cheaper high-volume model).
  - Always select from Multica's model list, because names come from each runtime.

### Release management (decided 2026-10-18)
- **No Release Manager agent yet.** Releasing is a process step before it is a role.
- **Staged plan:**
  1. The Doc Owner feeds the Release Notes page.
  2. Once Foodie has a deploy pipeline, add a `release` step owned by the Tech Lead.
  3. Add a dedicated agent only when there are environments to operate.
- **Foodie is not deployed** (confirmed 2026-10-18). Stages 2–3 are deferred until it is. Stage 1 can be done now as a one-line Doc Owner change.
- **Foodie security:** the repo already has secret scanning, dependency scanning and CodeQL.

### Agreed improvement backlog (2026-10-18, in order)
1. **DONE 2026-10-18.** Trace acceptance criteria through the flow: the Product Owner's numbered criteria, the engineers' evidence for each, Tester and QA verdicts for each, and the Tech Lead's ISSUE DONE check.
2. **DONE 2026-10-18.** A standard first line for hand-back comments (`Outcome: done | blocked | needs decision — …`), defined once in Workspace Context.
3. **DONE 2026-10-18.** Separate Tester and QA duties: Tester does adversarial defect-hunting; QA does acceptance verification, regression and QA.md.
4. **DONE 2026-10-18 (option A: the Architect reviews, not the Tech Lead).** A code review step before `in_review`. The built-in squad protocol forbids the leader from reviewing PRs itself ("review this PR" must be delegated).
5. **DONE 2026-10-18.** A CI gate: the Tech Lead checks `checks_conclusion` via `multica issue pull-requests`, and registers `--until-pr checks` only while checks are pending. The condition ignores results that finished before registration.
6. **DONE 2026-10-18.** Shared Multica skills (`confluence-pages`, `pull-requests`) to remove procedures duplicated across prompts.
7. **DONE 2026-10-18 (live).** A weekly autopilot health check: paused wakeups, issues stalled in `needs_refinement`, unpromoted stages, failed runs.
8. **DONE 2026-10-18.** Release notes now: the Doc Owner adds merged, user-visible changes to the Release Notes page (stage 1). The `release` step waits until Foodie is deployed.

### Live Multica workspace (CLI connected 2026-10-18)
- **Connection:** `multica` CLI is set up on the user's Mac. It's connected to Multica Cloud, workspace `natg` (`571ea6c3-5b15-49a5-ac6f-abfc5e395753`).
  - Setup also started a local daemon on the user's Mac, which registered Claude, Codex and Hermes runtimes for `Mac.natg.co.uk`. The user says to ignore them: the normal runtimes are **dev-ai-1** and **dev-ai-3** (see `Runtimes/README.md`).
- **Runtimes:**
  - `Claude (dev-ai-1)`, id `8fb37238…`
  - `Codex (dev-ai-3)`, id `e3c21b4a…`
- **Agents:** 12 in the workspace. The Tester prompt is used by two of them, `Tester (Claude)` and `Tester (Codex)`.
- **Changes applied to the live workspace on 2026-10-18 (user-approved):**
  - Imported the skills `confluence-pages` and `pull-requests` from this repo's GitHub URLs. They have a GitHub source, so `multica skill refresh <id>` pulls updates.
  - Set every agent's skills as listed in `Agents/*/Skills.md`.
  - Set every agent's model and thinking level as listed in `Agents/*/Model.md`. Before this, the Tech Lead and Product Owner were on Fable, Tester (Claude) on Opus, the Security Analyst on GPT-6 Astra, and most agents on thinking `medium`.
- **Not yet confirmed at runtime:** `claude-sonnet-5-5` with `high` for Tester (Claude), and `xhigh` for the Security Analyst. The daemon only checks a model and thinking-level pair when a run starts.
- **Rollback:** the settings before 2026-10-18 are in the table below. To restore them, re-run `multica agent update <id> --model … --thinking-level …` and `multica agent skills set <id> --skill-ids …` (get skill ids from `multica skill list`).

| Agent | Agent id | Model | Thinking | Skills |
|---|---|---|---|---|
| Tech Lead | `0a2cf79b-2aca-4483-a2bc-04a339edf621` | claude-fable-5-1 | high | code-review, grill-me |
| Tester (Claude) | `2fc80465-fe7d-460d-858a-049de399cf39` | claude-opus-5-5 | medium | code-review, testing-strategy |
| Tester (Codex) | `c0a163bf-92d4-45f3-83a1-cef5b3733754` | gpt-6.1-sol | medium | code-review, testing-strategy |
| Researcher | `250e294a-8afb-4f58-a900-c46bb8678aed` | gpt-6.1-sol | medium | — |
| Frontend Engineer | `8c29de4a-e388-41bc-9784-afed10fe384e` | claude-opus-5-5 | medium | figma-code-connect, figma-design-to-code, frontend-ui-engineering |
| Backend Engineer | `9be97dc7-f3b4-485a-a452-4897da6643c3` | claude-opus-5-5 | medium | — |
| Product Owner | `5dbd5e9d-f0f6-439e-ab4a-77f7c2afbde4` | claude-fable-5-1 | high | grill-me |
| Doc Owner | `5ac65b87-6860-48de-b643-94901710a58e` | gpt-6.1-sol | high | — |
| Security Analyst | `da2b3f95-0ba0-4781-b844-d0fbf5f42e4c` | gpt-6-astra | medium | analyzing-code-security, detecting-secrets |
| QA | `a281e28d-360d-42af-bdee-cb64ee373bfc` | gpt-6.1-sol | medium | — |
| Frontend Designer | `8622da57-afa0-449e-b3c3-6ef4da3ddfbe` | claude-opus-5-5 | medium | figma-create-new-file, figma-generate-design, figma-generate-diagram, figma-generate-library, figma-use, figma-use-figjam, figma-use-motion, frontend-design |
| Architect | `a76aa289-410c-47ea-ae0f-dbafc432b941` | claude-fable-5-1 | medium | architecture, code-review, system-design |

- **Prompts and squad instructions are still the live `_old` versions.** Only skills and models changed.
- **Weekly Health Check autopilot created 2026-10-18** (id `e5f8f176-74eb-4749-bd50-f3df42500e09`, trigger `f0027a59-…`):
  - Mode `create_issue`, assigned to the Tech Lead, with the user as subscriber.
  - Runs Saturdays at 09:00 Europe/London (cron `0 9 * * 6`). First run: 2026-10-24.
  - Its description is `Autopilots/Weekly Health Check/Runbook.md`. After editing the runbook, push it with `multica autopilot update <id> --description "$(cat …/Runbook.md)"`.

### Failure-loop incident (why FAILURE NOTICES exists — do not weaken)
When Claude ran out of usage allowance, an agent on Codex kept re-triggering the failing Claude agents, which caused **200+ failure loops**. Design constraints that follow from it:
- Failure handling must **not depend on the Tech Lead**, which may be on the same exhausted provider. It must live in Workspace Context, which reaches every agent including Codex ones (via `AGENTS.md`).
- Two loop routes were found in the source (`service/task.go`):
  1. The system failure comment is credited to the failed agent, in the triggering thread, so a reply in that thread routes back to it.
  2. The "delegated failure" signal wakes the agent that @mentioned the failed one with "Resume coordination… reassign it, skip it, or end the workflow". This invites re-delegation, and the per-signal cap doesn't stop a chain of new delegations.
- A rule in a prompt is not a guarantee. The operational backstop is to pause or archive the looping agent, or take its runtime offline.

**Server / workspace settings (user action, not Markdown):**
- ~~Set the PR merge status to `documentation`.~~ **Already set** (user confirmed 2026-10-18): merges move issues to `documentation`.
- ~~If a custom `to_do` status exists, archive it.~~ There is no `to_do` status (checked 2026-10-20).
- ~~Statuses at deployment~~ **Done live by the user on 2026-10-20, and verified:**
  - The PR merge status is `documenting`.
  - No issues remain on `ready` or `documentation`; both are retired.
  - The live Tech Lead and squad instructions now refer to `documenting` / “Documenting”, and `Archive/` was refreshed to match.
- ~~Test one refinement on a throwaway issue~~ **Done 2026-10-24 (NATG-380, passed).** The Tech Lead is not woken while the Product Owner has the issue, and is woken by the hand-back. **Still untested:** whether the Architect's threaded reply (no @mention) wakes the Product Owner; the Product Owner didn't involve the Architect. If it doesn't, the Architect needs to @mention the Product Owner once.
- ~~Test that a wakeup-started Tech Lead run still behaves as squad leader~~ **Done 2026-10-24 (NATG-381, passed).** A wakeup-started run's `CLAUDE.md` has the Squad Operating Protocol, the Squad Roster and the (new) Squad Instructions.

## Change log
- 2026-10-17: Created CLAUDE.md with condensed Multica knowledge and an initial list of observations.
- 2026-10-17: Added findings verified against the multica-ai/multica source (`18fa9f7`): instruction placement and precedence, the built-in status workflow, workspace context, mention types, wakeups and project workflows. Observations 6–10 added or updated.
- 2026-10-17: Draft squad `Instructions.md`: replaced "Human ONLY steps" with HUMAN CHECKPOINTS (one-off `--until-status` wakeups; fixed "ready" → `implementation`). Added the Pending follow-ups section.
- 2026-10-17: Draft squad `Instructions.md` ROUTING: the issue stays assigned to the squad and members don't reassign it. The session-limit rule was rewritten from the leader's side. Member-prompt and Workspace Context follow-ups added.
- 2026-10-18: Draft squad `Instructions.md` status list:
  - Completed the line 3 sentence.
  - Switched to the built-in `todo`.
  - Fixed the `research` wording.
  - `testing` now covers Tester → Security Analyst → QA.
  - `in_review` is aligned with the merge → `documentation` checkpoint.
  - Added a statement that members don't change the status.
- 2026-10-18: Draft squad `Instructions.md` SEQUENCE now matches the five work statuses. Testing covers Tester → Security Analyst → QA, and Security Analyst review of code changes is stated as the one mandatory step.
- 2026-10-18: Draft squad `Instructions.md`:
  - Moved the sequence rules out of ROUTING into SEQUENCE.
  - Confluence-only documentation issues now go straight to `documentation`, skipping implementation and testing, with no human review. That is the user's decision; the Tech Lead's final check is the gate.
  - Changes to docs in the repo follow the normal flow.
- 2026-10-18: Draft squad `Instructions.md`: ISSUE DONE rewritten as conditional checks (acceptance criteria; code checks only if the issue changed the codebase; documentation done or explained). Removed the duplicate "may mark Done" line from DOCUMENTATION.
- 2026-10-18: Draft squad `Instructions.md`: added a SUB-ISSUES section.
  - Sub-issues are assigned to `Dev Squad 1`, and engineers are dispatched by @mention within them.
  - Stages are used for ordering, with later stages parked in `backlog`; the leader may promote them.
  - A sub-issue runs implementation → testing → in_review, and the merge → `documentation` checkpoint marks it done.
  - The parent goes to `documentation` when all sub-issues are closed.
  - The `backlog` rule and the IMPLEMENTATION line were updated to match.
- 2026-10-18: Live `Instructions_old.md`: Notion → Confluence for Researcher drafts (plus the user's two blank lines). **User to paste into Multica.** Confluence is now a standing rule.
- 2026-10-18: Draft squad `Instructions.md` tidy-up: fixed the "Reasarch" typo, TESTING now uses `in_review`, and the redundant DOCUMENTATION section was removed (covered by the status list and SEQUENCE). Open: are "Tester Claude"/"Tester Codex" two server agents sharing `Agents/Tester`?
- 2026-10-18: Efficiency trim 1: moved ADMINISTRATION (branch names, `bws` secrets) from the draft squad `Instructions.md` to the new `Workspace/Context.md`. `&lt;`/`&gt;` escapes were fixed and the list flattened; the rules are unchanged. Remaining trims 2–5 (protocol duplicates, status-list overlap, line 1, the double Security Analyst rule) and the refinement-routing decision are still pending.
- 2026-10-18: Added FAILURE NOTICES to `Workspace/Context.md`, rewritten from the live prompts' copy:
  - It covers both failure routes.
  - Recovery doesn't depend on the leader.
  - One named fallback is allowed.
  - It merges the session-limit rule.

  The squad ROUTING session-limit comment now uses @all. The loop incident is recorded above.
- 2026-10-18: Efficiency trims 2–5 on the draft squad `Instructions.md`:
  - Removed protocol duplicates: the auto-wake explanation, "one delegation per turn" and the double-run warning.
  - Shortened the `implementation`/`testing` status entries.
  - Cut "marks done" from line 1.
  - Removed the third copy of the Security Analyst rule from TESTING.

  The file is about 1,720 words, down from about 1,980.
- 2026-10-18: Refinement is now routed via the Product Owner (user decision, to avoid a no-op Tech Lead run per refinement comment).
  - The Tech Lead moves the issue to `needs_refinement` and assigns it to the Product Owner, without an @mention.
  - The Product Owner assigns it back to `Dev Squad 1` when it is ready or blocked.
  - The ROUTING "keep assigned to squad" rule now has a refinement exception.
  - The trigger "a question that blocks and nobody has answered for a while" moved to the Product Owner.
- 2026-10-18: Added a DOCUMENTATION section to `Workspace/Context.md`: Confluence is the only store, research goes in the Research folder, and old Notion links are treated as historical.
- 2026-10-18: Wrote the new Tech Lead `System Prompt.md` (1,348 words), section by section from `System Prompt_old.md`. The new sections are WHAT YOU MAY DO ON THE PLATFORM (permissions, guard, chat rule) and RESEARCH AND PLANNING (Confluence-only, publishes approved research).
  - Trimmed the @mention paragraph, which the built-in Mentions section covers.
  - The squad `Instructions.md` WHEN WORK COMES BACK research lines are replaced by a pointer to the Tech Lead's own instructions.
  - `gitleaks` secret scanning runs in GitHub Actions (`.github/workflows/gitleaks.yml`). The repo is **public**, so never commit secrets.
- 2026-10-18: Wrote the new Product Owner `System Prompt.md` (1,475 words): the live prompt plus IN DEV SQUAD 1. Changes:
  - The assignee rule has a hand-back exception.
  - The @mention rule has an Architect exception.
  - It appends the refinement block in `needs_refinement`.
  - User decisions: 1A (the Product Owner @mentions the Architect, with stop conditions) and 2A (no chasing a silent person).
- 2026-10-18: Wrote the new Architect `System Prompt.md` (900 words).
  - Designs and ADRs are Confluence Drafts under Architecture and Decisions; nothing goes in the repo.
  - The implementation brief goes to the Tech Lead, with no dispatching.
  - A new "In Dev Squad 1" section adds the status rule and the refinement reply rule.
  - Skills are now `architecture` and `system-design`.
  - Added a Doc Owner follow-up.
- 2026-10-18: Wrote the new Researcher `System Prompt.md` (1,544 words, down from 1,704):
  - No repo writes and no index PR. The issue comment with the Draft link is the review signal, and the Tech Lead publishes.
  - The squad status rule is added.
  - FAILURE NOTICES is removed, and the "Altassian" typo is fixed.
- 2026-10-18: Wrote the new Doc Owner `System Prompt.md` (1,538 words, up from 1,437). It adds the "In Dev Squad 1" section, which finalises the Architect's drafts and leaves Research and Planning alone. Follow-up issues now go in `backlog`, and output goes in the issue comment. FAILURE NOTICES and the H1 were removed.
- 2026-10-18: Wrote the last six member prompts:
  - Frontend Engineer 1,208 words, Backend Engineer 1,183, Tester 794, QA 1,802, Frontend Designer 652, Security Analyst 507.
  - Shared changes: comment as the hand-back, no status or assignee changes, issues filed in `backlog`, FAILURE NOTICES removed.
  - Agent-specific changes are listed under Pending follow-ups.
  - All 11 agent prompts are now drafted.
- 2026-10-18: Added a `Model.md` to every agent folder (runtime, model, thinking level, why, and when to revisit). Recorded the model decisions, the "no Release Manager yet" decision, and the agreed improvement backlog.
- 2026-10-18: Flow consistency pass (backlog items 1, 2, 3, 8):
  - **Acceptance criteria:** labelled AC1, AC2, … by the Product Owner and carried through the engineers' hand-backs, Tester defects, QA's per-criterion results and the Tech Lead's ISSUE DONE check. Sub-issues get their own criteria.
  - **Hand-back first line:** `Outcome: done | changes needed | blocked | needs decision`, defined in Workspace Context. The Tech Lead routes from it.
  - **Tester vs QA:** the Tester hunts for defects; QA checks acceptance criteria, regressions and QA.md. QA waits until blocking Tester defects are fixed.
  - **Release notes:** the Doc Owner adds user-visible changes under "Unreleased".
  - Workspace Context is now 524 words.
- 2026-10-18: Code review and CI gate (backlog items 4, 5):
  - **New flow:** engineer PR → CI GATE → `testing`: Architect code review → Tester → Security Analyst → QA → `in_review`.
  - **Requirements:** CI and Architect review are now required for every code change, alongside Security Analyst review. ISSUE DONE check 2 and the Tech Lead's wakeup permission were updated, and the Architect has a review bullet.
  - **Decided:** the Architect stays on Fable 5.1, and its `Model.md` rationale is updated: design runs are rare, reviews are frequent but short.
  - **No dedicated Code Reviewer agent yet** (decided 2026-10-18). The Architect's value is checking code against its own design, and a change already passes four other checks. Revisit if Fable's 50% cap starts to limit design work, if the Architect's reviews turn generic, or if review volume slows design work.
- 2026-10-18: Backlog item 6, shared skills:
  - Added `Skills/confluence-pages` and `Skills/pull-requests`.
  - Replaced the duplicated procedures in 7 prompts with pointers: Researcher −290 words, Engineers about −165 each.
  - Found in source that any PR link counts towards merge status, not just "Closes", and corrected the earlier notes.
- 2026-10-18: User confirmed the PR merge status is already `documentation`. From that, inferred the server has MUL-7726 (any link counts towards merge status).
- 2026-10-18: Connected the Multica CLI. Reviewed all 19 workspace skills on their merits and wrote `Agents/*/Skills.md`.
  - **Applied to the live workspace:** imported the 2 repo skills, set every agent's skill assignments, and set every agent's model and thinking level to match `Model.md`.
  - **Removed skills:**
    - `grill-me` (broken stub)
    - `code-review` from the Tech Lead and Testers
    - `testing-strategy`
    - `figma-code-connect` (Figma plan is Professional)
    - FigJam and diagram skills
  - **Left unattached:** `threat-modeling` (Bitwarden-specific).
- 2026-10-18: Backlog item 7, the Weekly Health Check autopilot.
  - Added `Autopilots/Weekly Health Check/` (Runbook.md and Settings.md) and created the autopilot live with the CLI.
  - It's report-only, with 5 checks: wakeups that won't fire, stalled issues (3 days active, 7 days waiting), stages not promoted, failed runs and loop patterns, and untriaged backlog.
  - All 8 backlog items are now done.
- 2026-10-18: Added `Agents/*/Settings.md`, generated from `multica agent list`, and the agent avatars (`avatar.webp`; the Tester folder has `avatar-claude.webp` and `avatar-codex.webp`).
  - **Contents:** name, description, avatar, runtime, model, thinking level, service tier, access, maximum concurrent runs, environment variables, CLI arguments, MCP, disabled runtime skills, and conversation starters.
  - **Not included:** agent IDs and owners. No environment variable values exist.
  - **Stale conversation starters noted, not yet changed:**
    - The Tester starters use the `DEV-___` prefix, but the workspace prefix is `NATG`.
    - The Tech Lead's "Review before merge" and the Tester's "Verify a change" predate the Architect-review and Tester/QA split.
- 2026-10-18: Conversation starters updated live and in `Agents/*/Settings.md` (user-approved).
  - **Principle:** starters are chats, which get no squad flow, so they advise rather than do work.
  - **Removed:** "Pick up a task" from both Engineers, the Tech Lead's "Review before merge", and the Tester's `DEV-___` starter.
  - **Added:** starters for the Architect, Frontend Designer and QA.
  - **Placeholders:** all now use `NATG-___`.
- 2026-10-18: Added `Runtimes/README.md`.
  - **dev-ai-1:** Claude Code 2.1.301, running the 7 Claude agents. It also has a spare Codex runtime.
  - **dev-ai-3:** codex-cli 0.162.1, running the 5 Codex agents.
  - **Still to confirm (❓ items in the README):** MCP servers other than Figma, `gh`, `bws`, the Foodie toolchain, branch protection, and the dev-ai-1 Codex account.
- 2026-10-18: Moved all `_old.md` files (11 agent prompts and the squad's `Instructions_old.md`) into `Archive/`, mirroring the folder structure. Updated the links in `Settings.md` and the repo layout in `CLAUDE.md`. Older change-log entries keep their original paths.
- 2026-10-18: Added `Squads/Dev Squad 1/Members.md`, exported from the live squad: 13 members including the leader, 11 member agents and 1 human (the user, role "Solutions Architect"). **No live changes; the user said not to deploy yet.** Open items for deployment:
  - ~~The live squad instructions still say "Notion".~~ **Fixed live by the user on 2026-10-20**, and the archive was refreshed to match.
  - **Member roles are out of date. They feed the leader's roster, so they steer routing:**
    - The Researcher's role says "Claude models", but it runs on Codex.
    - The Product Owner is called "Product Lead".
    - The Architect's role doesn't mention code review.
    - The Tester and QA roles don't reflect the split.
    - The user's "Solutions Architect" may compete with the Architect agent.
    - **Update (user decision):** `Squads/Dev Squad 1/Members.md` now holds the **new** roles. The live roles moved to `Archive/Squads/Dev Squad 1/Members_old.md`, matching the `_old` = live convention. Apply the new roles with the CLI only when the new set is deployed.
- 2026-10-18: Added `Runtimes/Setup.md`, a step-by-step guide to setting up an Ubuntu 24.04 host as a Multica runtime.
  - **Sources are marked in the guide:**
    - 📘 official documentation: Multica CLI and daemon, Claude Code install and `setup-token`, Codex install and `--device-auth`, gh, bws.
    - 🛠 this repo's own recommendations, not covered by Multica's docs: a dedicated `multica` user without sudo, a systemd user service with linger, an environment file for `BWS_ACCESS_TOKEN`, and hardening.
  - **MCP setup is described generically**: "configure any MCP servers on the coding agent". On dev-ai-1, the Claude Code MCP servers sync from the claude.ai account, but that isn't safe to assume for someone else's setup (user, 2026-10-18).
  - **Still to document:** the Foodie toolchain, and how MCP is provided on the Codex runtime (dev-ai-3).
  - `gh` and `bws` are confirmed installed on both runtimes (user, 2026-10-18).
- 2026-10-18: `Runtimes/Setup.md` additions (user-approved):
  - **Step 1, packages:** wget, zip, gnupg, make, python3/venv/pip, ripgrep, fd-find (plus an `fd` link), jq, tree, htop, less, openssh-client.
  - **Step 1, Node.js LTS:** installed via NodeSource.
  - **Step 1, linger:** moved here from the daemon step, because rootless Docker needs it too.
  - **New step 6, rootless Docker (option B):** Docker's apt repo, plus `docker-ce-rootless-extras`, which bundles the Ubuntu 24.04 AppArmor profile. The system daemon is disabled. `dockerd-rootless-setuptool.sh` and `DOCKER_HOST` are set in the daemon service.
  - **Docker group:** explicitly not used, because membership is effectively root access.
  - **Later steps:** renumbered 7–10.
- 2026-10-18: Markdown formatting fix:
  - **ALL-CAPS section titles became `###` headings** in the 6 plain-text agent prompts, the squad `Instructions.md` and `Workspace/Context.md`. The wording and case are kept, because files refer to sections by name, e.g. "see SUB-ISSUES".
  - **The 5 prompts that already used `##`/`###` were moved down one level**, for consistency.
  - **Why `###`:** Multica injects these files under its own `##` headings (`## Agent Identity`, `## Squad Instructions (…)`, `## Workspace Context`), so `###` nests correctly.
  - Added the missing final newlines.
  - `Archive/` was left untouched, because those files mirror what's live.
- 2026-10-19: Frontend Designer, new prompt only (`Archive/` unchanged, at the user's request):
  - **Problem:** live, the Designer puts walls of text in Figma. The new draft made it worse by putting handoff notes in Figma.
  - **New "What goes in Figma" section:** Figma holds drawn work (wireframes, page structures, visual designs, components and states, responsive layouts). Text is limited to UI copy and short annotations pinned to elements. Rationale, specs, handoff notes and open questions go in the issue comment.
  - **Why:** the Frontend Engineer reads Figma through the MCP, so text frames bloat its context and could be mistaken for UI to build. Written content belongs in the comment or in Confluence.
  - **Note:** `figma-generate-library` creates design-system documentation pages. That's legitimate only for library work; make an exception if a Figma component library is ever built.
- 2026-10-20: Exported the current live config into `Archive/`, mirroring the new files:
  - **Agents:** `Settings_old.md`, `Model_old.md`, `Skills_old.md` and `avatar*_old.webp` for each agent.
  - **Workspace:** `Archive/Workspace/Settings_old.md` and `Context_old.md`.
  - **Models, skills and conversation starters already match the new files,** because they were applied live on 2026-10-18. The pre-2026-10-18 values exist only in the rollback table above.
  - **Findings:**
    - **The live Workspace Context is empty,** so `Workspace/Context.md` can be pasted in directly with nothing to merge.
    - **Drift: the live Doc Owner instructions differ from `Archive/Agents/Doc Owner/System Prompt_old.md`.** Live opens "You are Docs"; the archive says "You are Doc Owner". The other 11 live prompts match the archive exactly. The archive prompt was left unchanged (user instruction 2026-10-19).
    - **The workspace has an MCP server library** of 7 servers: Altassian-MCP (sic), AWS-Knowledge, Composio, Context7, Figma, NetBox-MCP and SearXNG. No agent has an MCP configuration, so how (or whether) agents use this library is unverified. `Runtimes/README.md` currently says MCP servers come from the runtime machine.
    - **The workspace has 7 linked repos.** Their names aren't exported (user decision, 2026-10-20). Per the docs, the workspace's repos are always included in a run's context.
    - **The workspace settings confirm** `pr_merge_status: documentation`.
- 2026-10-20: Made the archive an exact copy of live, for rollback.
  - **Refreshed from live:** `Archive/Agents/Doc Owner/System Prompt_old.md` (live says "You are Docs") and `Archive/Squads/Dev Squad 1/Instructions_old.md`.
  - **Squad instructions:** the user fixed the live "Notion" line to say Confluence, so the archive now matches with no diff.
  - **Check:** all 12 live agent prompts and the squad instructions now match `Archive/` byte for byte.
  - **Status files:** the user supplied the issue-status list, now in `Workspace/Issue Status.md` and `Archive/Workspace/Issue Status_old.md` (identical).
  - **Findings from the status list:**
    - Research, Needs Refinement, Refinement Complete, Implementation and Testing **already exist live**. There is no `to_do` status, so that follow-up is closed.
    - ⚠️ **Documentation is in the Done group.** Done-group statuses may count as complete: a move there could disable the issue's wakeups and count sub-issues as finished before the Doc Owner runs. The group can't be changed after creation, so the fix may be a new Started-group status (for example "Documenting") with the PR merge status pointed at it. **Test this on a throwaway issue before deploying.**
    - ⚠️ **Keys vs display names.** 355 live issues use the key `ready` (9 issues), but no status is displayed as "Ready", so a status was probably renamed while keeping its key (most likely "Implementation"). The prompts and wakeups use keys (`implementation`, `refinement_complete`, `testing`, …), so **the actual keys must be confirmed before deployment**.
  - **Still to do for rollback:** a cutover snapshot of open issues (key, status, assignee), and `Archive/README.md` restore steps.
- 2026-10-20: Issue statuses for the new flow.
  - **Verified in the Multica source** (`internal/handler/issue_status.go`, `internal/issuestatus/issuestatus.go`):
    - Status keys are **immutable**, and so are status groups (categories).
    - An **archived status keeps its key**, so the key can't be reused.
    - Built-in keys: `backlog`, `todo`, `in_progress`, `in_review`, `blocked`, `done`, `cancelled`.
    - Custom statuses in the Done or Closed group behave as `done` / `cancelled`. Entering one runs `StopClosedIssueWakeups`, which disables every wakeup on the issue.
  - **Live problems found:**
    - "Implementation" had the key `ready`, because it was renamed after creation.
    - "Documentation" (`documentation`) was in the **Done** group, which would disable the checkpoint wakeup at the moment it should fire, and count sub-issues as finished.
  - **The user created two new statuses live:** Implementation (`implementation`) and **Documenting (`documenting`, Started group)**.
  - **Repo changes:**
    - `Workspace/Issue Status.md` is the target set, with all keys and a "Retiring the old statuses" table.
    - `Archive/Workspace/Issue Status_old.md` records the pre-migration set (`ready`, and `documentation` in Done).
    - **Every new file now uses `documenting`** (10 places): the squad `Instructions.md` (status list, checkpoint wakeup, docs-only path, SUB-ISSUES), the Tech Lead's permissions, the Doc Owner's squad section, and both Engineers' `Skills.md`.
  - **Repository names removed** from `Archive/Workspace/Settings_old.md` and from this file (user decision: the repo is public, and the list is easy to recreate by hand).
- 2026-10-20: The user completed the status migration live (PR merge status → `documenting`, issues moved, old statuses retired, live Tech Lead and squad instructions updated), and it was verified. The `Archive/` copies of the Tech Lead prompt and the squad instructions were refreshed to stay exact. `Archive/Workspace/Issue Status_old.md` stays as the **pre-migration** status set; `Workspace/Issue Status.md` now matches live.
- 2026-10-24: **Deployed the new config live** (user-approved), via the CLI, each piece read back and compared:
  - Workspace Context (was empty; the server trims the final newline), squad instructions, 12 agent prompts (both Testers from `Agents/Tester`), 12 member roles from `Members.md` (the leader keeps "leader").
  - Before the deploy: confirmed no drift between live and `Archive/`; wrote `Archive/Workspace/Open Issues_old.md` (131 open issues: key, status, assignee, parent; no titles because the repo is public).
  - **All 61 open issues not in `backlog` were moved to `backlog`** (user decision), so the new flow starts clean and the user re-triages. None of the 9 enabled wakeups on them fires on a move to `backlog`.
  - Context: the first scheduled Weekly Health Check (NATG-379, 2026-10-24) found the live squad instructions still said "Ready" after the 10-20 migration, so 9 `implementation` issues had no run. The new instructions don't have that problem. Its question about the 12-runs-per-hour guard on NATG-320/327 was a misreading: that guard covers wakeup rules, not comment-triggered runs (the 10-17 failure loop).
  - **Leftover:** NATG-222 still has an enabled wakeup on the retired key `documentation`; it can never fire. Replace with `documenting` or delete when the issue is re-triaged.
  - **Still to do:** `Members.md` still says "apply together with the new set" and could be reworded as live; Health Check runbook check 5 should include unassigned `research-followup` issues.
- 2026-10-24: Cutover tests on throwaway issues, both passed, both issues then cancelled (wakeups disabled):
  - **NATG-380 (refinement), 4 runs:** Tech Lead → `needs_refinement` + assigned to the Product Owner (no @mention); Product Owner's round of questions did not wake the Tech Lead; the human's reply woke the Product Owner, which mapped answers to ACs and handed back with `Outcome: done`; the Tech Lead woke, moved it to `refinement_complete` and registered the `--until-status implementation` wakeup.
  - **NATG-381 (wakeup path):** a timer wakeup aimed at the Tech Lead on a `backlog` issue started a run whose brief had all three squad headings and the new squad instructions. It changed nothing else.
  - Not covered: the Architect's threaded reply waking the Product Owner.
- 2026-10-27: **Human-action assignment** (user decision: escalations only), repo only, not deployed:
  - When an issue needs the user's action, the Tech Lead posts the comment and then assigns the issue to Nathaniel Greenwood (`multica issue assign <id> --to "Nathaniel Greenwood"`), leaving the status unchanged. Covers escalations, `blocked`/`needs decision` hand-backs it can't resolve, a blocked refinement and session-limit stops. The user acts and assigns it back to `Dev Squad 1`, which wakes the Tech Lead (assigning to a member starts no run).
  - Not used at HUMAN CHECKPOINTS (their `--until-status` wakeups would fire while the issue isn't assigned to the squad, and the Tech Lead's guard would stop it acting) or during refinement Q&A (the human's reply has to wake the Product Owner).
  - Files: squad `Instructions.md` (ROUTING, REFINEMENT) and the Tech Lead's escalation paragraph and platform permissions.
