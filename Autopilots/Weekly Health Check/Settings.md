# Autopilot: Weekly Health Check

| Setting | Value |
|---|---|
| Title | Weekly Health Check |
| Mode | `create_issue` |
| Assignee | Tech Lead |
| Issue title template | `Weekly health check {{date}}` |
| Subscriber | Nathaniel Greenwood |
| Trigger | Schedule, cron `0 9 * * 6` (Saturday 09:00), timezone `Europe/London` |
| Runbook | `Runbook.md` in this folder, used as the autopilot's description (its run prompt) |

**Why `create_issue`:** each report becomes an issue that notifies the subscriber and stays on the record. A `run_only` report would only be visible in the run history.

**Why report-only:** re-enabling a paused wakeup or restarting a failed run is how a failure loop restarts. Those decisions stay with a human.

**Thresholds:** 3 days for active statuses and 7 days for statuses waiting on a person. Confirmed by the user on 2026-10-18.
