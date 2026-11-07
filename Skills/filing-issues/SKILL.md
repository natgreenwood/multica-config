---
name: filing-issues
description: Use before you create a Multica issue for something you found while working on another issue: a bug, a security problem, a test or fixture problem, a documentation error or an improvement. Covers what not to file, the limit of two per run, the label, the priority, the first line, and duplicates.
---

# Filing issues you find along the way

## First: does it need an issue at all?
- **It blocks the work you're on:** don't file it. Report it in your hand-back as blocking (`Outcome: changes needed`), so this issue's own flow fixes it.
- **It's a question** ("Should…?", "Is this intended?", "Does X need Y?"): never file it. List it under **Open questions** in your hand-back; the Tech Lead decides whether it becomes work.
- **It's already tracked:** search first, for example `multica issue list --project <project-id> --output json --fields identifier,title,status`, and read any likely match. If an open issue covers it, comment there with your new evidence instead.
- **It's minor and unrelated** (naming, style, a tidy-up): mention it in your hand-back only.

## No more than two per run
File at most **2** new issues per run. If you found more, file the two that matter most (security first, then bugs a user would hit), and list the rest in your hand-back under **Not filed**, one line each.

## How to file one
1. Create it in `backlog`, assigned to `Dev Squad 1`, in the same project as the issue you're working on. Don't use `--parent`: a sub-issue changes how the parent's stages finish.
2. Set the priority from its type:

   | Type | Label ID | Priority |
   |---|---|---|
   | `security`: data exposure, crossing accounts, auth, injection, a crash any user can cause | `8fe1fd26-b8da-4f23-af48-2e78571f67b1` | `high` |
   | `bug`: wrong behaviour a user can hit | `d04b67a1-84c4-4d08-a928-f41502502c2f` | `medium` |
   | `testing`: flaky or wrong tests, fixtures, test tooling | `625ac7d5-ef48-499a-8b58-4608cea76f2b` | `low` |
   | `documentation`: wrong or missing docs in the repository | `60166a36-8803-4374-b94f-dee392ee2a4b` | `low` |
   | `improvement`: works, but could be better | `6248d75e-32be-4fb3-9c09-64984036f6ee` | `low` |

3. Add exactly one type label: `multica issue label add <new-issue> <label-id>`.
4. Start the description with this line, then the evidence:

       Found while working on NATG-123 (PR #45). Blocks it: no.

   Then: what happens, what should happen, how to reproduce it (a command, test name, or file and line), and how you know. Title it as the problem, not the topic: "Boot 401 clears another account's gym_user", not "Boot handling".
5. List each issue you filed in your hand-back, by key, with its type.

## Security issues
File them like any other, with the `security` label and `high` priority. Describe where the problem is and what it allows, but never include a working exploit, secrets or real account data.

## Duplicates
If you find an issue that duplicates an older one, don't file another. If you are the Tech Lead triaging, cancel the newer one with `multica issue update <key> --duplicate-of <older-key>`.
