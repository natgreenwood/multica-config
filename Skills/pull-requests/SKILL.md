---
name: pull-requests
description: Use whenever you open, edit or describe a pull request connected to a Multica issue. Covers how a PR is linked to its issue, how merging moves the issue on, and how to keep a supporting PR from moving it.
---

# Pull requests and Multica issues

## A PR that delivers an issue
- Put the issue key in the PR **title**, for example "NATG-123: add recipe export".
- Also put a close intent in the PR **description** (the body field), on its own line: "Closes NATG-123". Nothing may sit between the keyword and the key. "Closes the export bug NATG-123" doesn't count, and neither does a markdown link. Write the key as plain text, and add a link on a separate line if you want one.
- A commit message or a later comment on the PR doesn't count. To fix a PR, edit its title or description.
- On a sub-issue, use the sub-issue's key, not the parent's.
- Once every PR linked to an issue has merged, Multica moves the issue on. Closing a PR without merging moves nothing.
- Confirm the link with `multica issue pull-requests <issue-id> --output json`.

## A supporting PR that must not move the issue
For example, a separate QA.md or test-only change. Keep the issue key out of its title, branch name and description, so it isn't linked: a linked PR changes when the issue moves. Put its URL in your issue comment instead.
