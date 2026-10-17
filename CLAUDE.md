# CLAUDE.md — multica-config

Source of truth for the configuration of a Multica (multica.ai) workspace: agent instructions and squad instructions, kept as Markdown. The user copies these files to the Multica server by hand. Nothing here deploys automatically.

## Working rules (from the user — non-negotiable)

- **Do not edit any config Markdown (`Agents/**`, `Squads/**`) without the user reviewing the change first.** Propose the change as a diff or snippet, wait for approval, then apply.
- **Present changes in small pieces**: one file, or one section, at a time.
- This file (CLAUDE.md) is project tracking and may be updated as needed.
- Commit or push only when the user asks. Branch off `main` first (the repo uses PRs, e.g. `CHORE--...`).

## Repo layout

```
Agents/<Agent Name>/System Prompt_old.md -> LIVE agent instructions (what the server runs today)
Agents/<Agent Name>/System Prompt.md     -> NEW agent instructions being written (blank as of 2026-10-17)
Squads/Dev Squad 1/Instructions_old.md   -> LIVE squad instructions
Squads/Dev Squad 1/Instructions.md       -> DRAFT proposal for the new squad instructions (not deployed)
```
Convention: `*_old.md` = live and deployed. The un-suffixed file = the new version being drafted, which the user deploys once it is approved.

**Current state (2026-10-17):** `Instructions_old.md` is live. `Instructions.md` is the user's new draft, written to introduce a status-driven flow (Research → Refinement → Implementation → Testing → Documentation, plus custom statuses). The live agent prompts (`System Prompt_old.md`) were written against the live squad instructions. The new `System Prompt.md` files start blank and will be rewritten to match the new squad flow.

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
- System behaviours: a failed run reverts `in_progress → todo`; a merged PR with close intent moves the issue to `done`; `backlog` never triggers runs.
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
  - Unverified: whether a wakeup run for the squad leader carries the squad briefing. User rules target an *agent*, whereas the built-in child-done rule targets the squad's leader. Test this on a real issue.
- **Project workflows** (in source, not yet in the public docs). A project can define ordered status columns where entering a column **reassigns the issue to that column's handler** (an agent, squad, member, project lead or creator) and starts their run. A column may also name a "needs changes" status. A named workflow rejects statuses outside it, and only the assignee gets the step's instructions. **This could replace much of the status-routing logic currently written into the Dev Squad 1 instructions.** Availability on our server version is unconfirmed.

- **Status changes don't wake anyone** (`handler/issue.go` `shouldEnqueueAgentTask`). The exception is leaving the fixed `backlog` key, which enqueues the assigned agent. A human moving `refinement_complete → implementation` wakes nobody on its own. Something has to catch it: a comment or @mention, an `issue wakeup --until-status`, or a project-workflow handoff.
- **PR merge status is configurable** (workspace setting `pr_merge_status`, `handler/pr_auto_complete.go`). It decides what status an issue moves to when all its linked PRs merge. The default is `done`, and `none` leaves the status alone. It can be set to a custom key, such as `documentation`. With the default, a merge sends the issue straight to `done` and skips any post-merge steps.

### Docs index
Key pages: `/docs/concepts`, `/docs/how-multica-works`, `/docs/agents`, `/docs/agents-create`, `/docs/skills`, `/docs/squads`, `/docs/triggering-agents`, `/docs/mentioning-agents`, `/docs/tasks`, `/docs/issues`, `/docs/autopilots`, `/docs/project-resources`, `/docs/cli`.
Context7 IDs: `/multica-ai/multica` (source, includes the runtime brief and squad briefing code), `/websites/multica_ai` (docs site).

---

## Observations to review with the user (not yet actioned)

Items 2, 3, 5 and 10 concern the **draft** `Instructions.md`. Items 4, 7 and 8 also apply to the **live** `Instructions_old.md` (it has the same tester, "assign to Dev Squad 1" and admin/secrets sections). Also, the live version says Research findings arrive as a *Notion* page, while the draft says *Confluence*. The Researcher and Doc Owner prompts should be checked against whichever is intended.

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

**Tech Lead `System Prompt.md` (new). When we write it, it must include:**
- Permission to register issue wakeups (`multica issue wakeup create/list`), matching the HUMAN CHECKPOINTS section of the squad `Instructions.md`.
- Permission to set the custom statuses the squad flow needs, especially `refinement_complete` and `in_review`. Agent Identity outranks the built-in workflow and the squad protocol's status rules, so the prompt must state which statuses it may write.
- The live prompt (`System Prompt_old.md`) has an ad-hoc "register a wakeup… when status becomes documentation" line. Replace it with a pointer to the squad instructions rather than duplicating it.

**Every member agent's new `System Prompt.md` (all except Tech Lead):**
- Post your result as a comment on the issue. That comment is what wakes the Tech Lead.
- Do not reassign the issue. It stays assigned to `Dev Squad 1`, as the ROUTING section of the squad `Instructions.md` says.
- Do not change the issue status (observation 7), or state explicitly which statuses this agent may write.

**Workspace Context (or each member prompt):**
- The member-side "session limit" rule: if a run hits "You've hit your session limit", stop all actions and await instructions. It used to sit in the squad instructions, which only the leader reads.

**Server / workspace settings (user action, not Markdown):**
- Set the PR merge status (workspace setting `pr_merge_status`) to `documentation`. Otherwise a merge moves the issue to `done`, which disables the `--until-status documentation` wakeup and skips the Doc Owner.
- If a custom `to_do` status exists on the server, move its issues to the built-in `todo` and archive it.
- Test on a throwaway issue that a wakeup-started Tech Lead run still behaves as squad leader (gets the squad briefing and instructions).

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
