### BRANCH NAMES

- Trunk branches are used as-is, with no prefix or description: `main`, `master`, `develop`.
- All other branches use the format `<type>/<description>`.
- Types:
  - `feat`: new functionality
  - `fix`: non-urgent bug fixes
  - `chore`: maintenance, dependencies, tooling
  - `release`: release preparation
- Description rules:
  - Lowercase letters (a-z) and digits (0-9) only.
  - Words are separated by a single hyphen (-).
  - Dots (.) are allowed inside a word, for example in version numbers like 2.4.1. A dot must have a letter or digit on both sides.
  - Exactly one `/`, between the type and the description.
  - No uppercase, underscores, spaces or other special characters. No consecutive hyphens or dots.
  - No hyphens or dots at the start or end of the description.

### SECRETS

- Bitwarden Secrets Manager (`bws`) is installed on the runtime. Use it to retrieve secrets as needed.

### DOCUMENTATION

- Confluence is the only store for documentation and research. Use the project's Confluence space; research goes in its Research folder.
- Do not create or update anything in Notion. It is no longer used. If you find an old Notion link, treat it as historical and say so rather than following it as current.

### HAND-BACK COMMENTS

When you hand work back on an issue, the first line of your comment is:
`Outcome: done | changes needed | blocked | needs decision — <one-line summary>`
- done: you delivered what you were asked for. A passing test run, a clean review, a ready refinement and a finished design all count.
- changes needed: you found problems someone else must fix. Failed tests, review findings and a BLOCK verdict all count.
- blocked: you couldn't finish without something you don't have. Say what, and from whom.
- needs decision: you stopped at a choice that isn't yours to make. Give the options and your recommendation.
Detailed verdicts (Pass/Fail, APPROVED/BLOCK) go in the body of the comment, not instead of this line.

### FAILURE NOTICES

These rules apply to every agent. They exist to stop failure loops, so they never depend on another agent, including a squad leader, being available.
- A failure notice is either a system comment saying a run failed, or a message that a task you delegated "ended in a final failure".
- If the failure has a platform cause (usage or session limit, runtime offline, agent error with no result):
  - Do not retry or @mention the failed agent, and do not reply in its thread. The cause is usually shared, such as an account limit, so the next run fails too.
  - Do not hand the work to another agent, unless your own instructions name a specific fallback. Even then, do it only once per failure.
  - If you must record the failure, post one note with an @all broadcast ([@all](mention://all/all)) and no agent or squad mention, so no run is re-triggered. Never post more than one such note per thread.
  - End your turn. Your own completed work stands, and a human will resume the failed work.
- If you hit "You've hit your session limit" yourself, stop all actions and wait for instructions.
