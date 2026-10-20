# Issue statuses (new flow)

The natg workspace's issue statuses for the new Dev Squad 1 flow. The prompts, squad instructions and wakeup commands use the **keys**.

Built-in statuses (backlog, todo, in_progress, in_review, blocked, done, cancelled) have fixed keys. Custom keys and groups can't be changed after creation, and an archived status keeps its key.

## Unstarted

Backlog (key: `backlog`)
- Parked. Assigning an issue here never starts an agent run.

Todo (key: `todo`)
- Queued. Moving an issue here starts the assigned agent.

## Started

Research (key: `research`)
- The issue requires further research before next steps

Needs Refinement (key: `needs_refinement`)
- The issue requires refinement by the Product Owner agent.

Refinement Complete (key: `refinement_complete`)
- The issue has been refined and is ready for implementation

Implementation (key: `implementation`)
- All research and refinement is complete. The issue is now ready for Implementation.

In Progress (key: `in_progress`)
- Actively being worked on.

Testing (key: `testing`)
- The issue is waiting for or is currently being tested

In Review (key: `in_review`)
- Delivered, waiting on human review. Finalizes the autopilot run.

Blocked (key: `blocked`)
- Stalled on an external dependency.

Documenting (key: `documenting`)
- Issues in this status require documentation by the Doc Owner agent. **The PR merge setting should move issues here.** It is in the **Started** group, so wakeups keep working and sub-issues aren't counted as finished.

## Done

Done (key: `done`)
- Completed.

## Closed

Cancelled (key: `cancelled`)
- Decided not to do.

## Retiring the old statuses

These still exist live, and are recorded in `Archive/Workspace/Issue Status_old.md`. Archive them once their issues have moved. Their keys stay reserved after archiving.

| Old status (key) | Group | Move its issues to | Then |
|---|---|---|---|
| Implementation (`ready`) | Started | `implementation` | Archive `ready` |
| Documentation (`documentation`) | Done | `documenting` | Archive `documentation`, after pointing the PR merge setting at `documenting` |
