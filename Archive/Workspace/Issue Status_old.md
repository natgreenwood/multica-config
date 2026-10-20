# Issue statuses (live, before migration)

The natg workspace's issue statuses as they were live before the new flow, for rollback. On 2026-10-20 two statuses were added live in preparation for the migration, `implementation` and `documenting` (see `Workspace/Issue Status.md`); they aren't part of this pre-migration set.

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

Implementation (key: `ready`)
- All research and refinement is complete. The issue is now ready for Implementation. (Displayed as "Implementation", but its key is `ready`: the status was renamed after creation.)

In Progress (key: `in_progress`)
- Actively being worked on.

Testing (key: `testing`)
- The issue is waiting for or is currently being tested

In Review (key: `in_review`)
- Delivered, waiting on human review. Finalizes the autopilot run.

Blocked (key: `blocked`)
- Stalled on an external dependency.

## Done

Documentation (key: `documentation`)
- Issues in this status require documentation by the Doc Owner agent. **The PR merge setting moves issues here.** It is in the **Done** group, so Multica treats it as complete: entering it switches off the issue's wakeups, and sub-issues here count as finished.

Done (key: `done`)
- Completed.

## Closed

Cancelled (key: `cancelled`)
- Decided not to do.
