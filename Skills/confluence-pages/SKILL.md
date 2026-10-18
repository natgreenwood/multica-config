---
name: confluence-pages
description: Use whenever you create, update, publish or link a Confluence page — research, designs, ADRs, planning or project documentation. Covers draft versus published status, placing pages in the right space and folder, checking pages after writing, and what to do when Confluence is unreachable.
---

# Confluence pages

## Before you write
- Confirm the Atlassian connection works (fetch the current user and accessible resources) before relying on it.
- Each project has a Confluence space with the same name. Work only in that project's space.
- Search the space for an existing page on the subject, by title and by key terms, within the target folder. Confluence holds near-duplicates without warning. Update an existing page rather than adding a second.

## Creating a page
- Pass the space and the parent folder or page id explicitly. Defaults can put the page in a personal space.
- "Draft" means the page is created with status `draft`, which is unpublished. It does not mean "Draft" in the title, or a label or banner on a published page. Create a page as published (`current`) only when your instructions say you publish it.
- After creating or updating, read the page back and confirm its id, space, parent and status. Use the URL from that read; never construct or guess one.
- If a write is rejected as too large, split the content across several updates rather than guessing at the result.

## Updating
- Read a page before you change it.
- Updating a published page creates a new live version. If your instructions don't let you publish, propose the change as a draft or describe it in your issue comment instead.

## Linking, and when Confluence fails
- Link every page you create or change in your issue comment, and say whether it is a draft or published.
- If Confluence is unreachable (unauthorised, expired or erroring), say so plainly in your comment and put the content there instead. Never claim a page exists that you could not create or read.
