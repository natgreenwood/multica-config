# Full SDLC workflow

A Multica project workflow (Settings → Workflows) for projects that Dev Squad 1 runs end to end. It turns the review chain into four steps, each handed to one agent. Entering a step reassigns the issue to that agent and starts its run, so the Tech Lead isn't woken between reviewers. The other projects stay on Default, and a simpler workflow for them is still to come.

`steps.json` is the definition exported from Multica, and the source of truth for it. To apply a change: `multica workflow update 62c3ed3e-5e3f-4bc5-b5c0-50f30f661754 --steps-file "Workflows/Full SDLC/steps.json"`, then read it back with `multica workflow get 62c3ed3e --output json`. Step instructions are shown only to that step's handler, in its brief's `[Project workflow]` block.

## Steps

| Step | Handler | Step done → | Needs changes → |
|---|---|---|---|
| `code_review` | Architect | `testing` | `implementation` |
| `testing` | Tester (Codex) | `security_review` | `implementation` |
| `security_review` | Security Analyst | `qa` | `implementation` |
| `qa` | QA | `in_review` | `implementation` |

All other steps (`backlog`, `todo`, `research`, `needs_refinement`, `ready_for_implementation`, `implementation`, `in_progress`, `in_review`, `documenting`, `final_checks`, `blocked`, `done`, `cancelled`) have no handler. The squad routes them, as on Default.

## How issues get back to the squad

Multica can't hand an issue to a squad: the server rejects a squad as a handler (`issueworkflow.ValidateHandoff`, MUL-7420). So when a reviewer's move lands on a step with no handler (`in_review` after QA passes, `implementation` on Needs changes, or `blocked`), the reviewer assigns the issue to Dev Squad 1. That assignment wakes the Tech Lead once, with its squad briefing. The order is always: comment, move, assign.

## Prompt support and test (2026-11-06)

- The Architect, Tester, Security Analyst and QA prompts each have an exception: when their brief says they are handling a workflow step, they make its moves (their usual rule is "never change status or assignee", and Agent Identity outranks the workflow). The squad instructions' FULL SDLC WORKFLOW section tells the Tech Lead to start the chain with `code_review` and how to handle each way an issue comes back.
- Tested on a throwaway project, all four checks passed:
  - **Clean pass (NATG-579):** the four reviewers ran in about 8 minutes with no Tech Lead runs between them, and QA's assign-back woke the Tech Lead as squad leader.
  - **Needs changes (NATG-584):** the Tester failed the first pass, the Tech Lead routed the fix and re-entered at `testing`, not `code_review`. 5 Tech Lead runs in all, against about 7–8 for a clean pass under the old flow.
  - **Blocked (NATG-583):** the Architect set `blocked` with no PR to review, and the Tech Lead woke and escalated.
- Found in testing: QA pushed a QA.md commit to the PR after the other reviews. Its prompt now says QA.md changes go in a separate PR, and never in the PR under test.
- Live on Homelab Containers (attached by the user).

## Step instructions

### `code_review`: Code Review → Architect

```
Review the PR linked to this issue against the design and the acceptance criteria, as your instructions describe. If the Tech Lead returned the issue to this step after a fix, review the fix against your earlier findings.
1. Post your verdict as one comment on the issue, starting with the Outcome line.
2. Approved, or only non-blocking findings: run Step done. The issue moves to Testing and hands off to Tester (Codex).
3. Blocking findings: run Needs changes, then assign the issue to Dev Squad 1 (multica issue assign <issue> --to "Dev Squad 1") so the Tech Lead routes the fix.
4. Can't review (no PR, no design, PR won't load): comment why, set blocked, then assign the issue to Dev Squad 1.
Don't @mention other agents. The workflow starts the next step.
```

### `testing`: Testing → Tester (Codex)

```
Hunt for defects in the PR linked to this issue, against its acceptance criteria, as your instructions describe. If the Tech Lead returned the issue to this step after a fix, retest the fix and anything it touches.
1. Post your report as one comment on the issue, starting with the Outcome line.
2. No blocking defects: run Step done. The issue moves to Security Review and hands off to the Security Analyst. File non-blocking defects in backlog as your instructions say, assign these defects to "Dev Squad 1".
3. Blocking defects: run Needs changes, then assign the issue to Dev Squad 1 (multica issue assign <issue> --to "Dev Squad 1").
4. Can't test: comment why, set blocked, then assign the issue to Dev Squad 1.
Don't @mention other agents. The workflow starts the next step.
```

### `security_review`: Security Review → Security Analyst

```
Review the PR linked to this issue for security, as your instructions describe. If the Tech Lead returned the issue to this step after a fix, review the fix against your earlier findings; re-raise only what still blocks.
1. Post your verdict as one comment on the issue, starting with the Outcome line.
2. APPROVED, or only non-blocking findings: run Step done. The issue moves to QA and hands off to QA.
3. Blocking findings: run Needs changes, then assign the issue to Dev Squad 1 (multica issue assign <issue> --to "Dev Squad 1").
4. Can't review: comment why, set blocked, then assign the issue to Dev Squad 1.
Don't @mention other agents. The workflow starts the next step.
```

### `qa`: QA → QA

```
Verify the PR linked to this issue against each acceptance criterion, check for regressions and update QA.md, as your instructions describe.
1. Post your report as one comment on the issue, starting with the Outcome line, with a verdict per criterion.
2. Pass: run Step done (the issue moves to In Review), then assign the issue to Dev Squad 1 (multica issue assign <issue> --to "Dev Squad 1"). The Tech Lead tells Nathaniel it is ready to merge.
3. Fail: run Needs changes, then assign the issue to Dev Squad 1.
4. Can't verify: comment why, set blocked, then assign the issue to Dev Squad 1.
Don't @mention other agents.
```
