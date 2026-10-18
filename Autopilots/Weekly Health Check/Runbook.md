Report on the health of Dev Squad 1's board. This is a report only: don't change
any issue's status or assignee, don't re-enable or create wakeups, don't trigger or
@mention anyone. A human reads this and decides what to do.

Check the following, using the multica CLI (load the multica-platform skill for
command details). Only look at issues assigned to Dev Squad 1 or its members,
plus their sub-issues.

1. Wakeups that won't fire. For each open issue, list its wakeups. Report any that
   are paused (loop, rate or max_fires), disabled, expired, or showing a
   last_error.
2. Stalled issues, using time since the last comment or status change:
   - `needs_refinement` for more than 7 days (refinement waiting on a person)
   - `refinement_complete` for more than 7 days (waiting for a human to approve
     implementation)
   - `in_progress`, `implementation` or `testing` for more than 3 days with no
     run in that time
   - `in_review` for more than 7 days (PR waiting for a human)
3. Stages that weren't promoted. A parent whose earlier stages are all done or
   cancelled, but whose next stage's sub-issues are still in `backlog`.
4. Failed runs in the last 7 days that no later successful run on the same
   issue has followed. Flag any issue with 3 or more failed runs: that's the
   failure-loop pattern.
5. Untriaged backlog. Count issues created in the last 7 days in `backlog` with
   label `research-followup` or filed by a tester, and list the five oldest.

Post one comment on this issue:
- First line: `Outcome: done — <n> items need attention` (or "nothing needs attention").
- Then one section per check, listing each item as a link to the issue with a
  short reason. Leave out sections that have nothing in them.
- End with the one or two items you'd look at first, and why.

If a check can't be run (a command fails or data is missing), say which check and
why, rather than skipping it silently.
