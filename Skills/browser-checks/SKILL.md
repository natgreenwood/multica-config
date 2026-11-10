---
name: browser-checks
description: Use whenever you check a web page in a browser: a live site, a preview, or a dev server you started. Covers driving agent-browser, what you may and may not do on a live site, and how to report what you saw.
---

# Checking pages in a browser

## How
- The tool is `agent-browser`. Before your first command in a run, load its guide with `agent-browser skills get core` and follow it: open, `snapshot -i`, act on the `@e` refs, and take a new snapshot after anything changes the page.
- Use a session named after the issue (`--session natg-123`), and run `agent-browser --session natg-123 close` when you're done.
- Save screenshots inside your working directory, so you can attach them.
- The runtime limits which sites agent-browser can reach. If a page you need is blocked, report it. Don't change agent-browser's configuration or start Chrome another way.

## What you may do
- **A dev server or preview you started for this issue:** anything. It's yours.
- **Any other site, including production:** look only. Open pages, scroll, take snapshots and screenshots, and read console errors. Don't submit forms, create accounts, complete first-run setup, log in, change settings, or upload or delete anything, unless the issue names that exact action.
- Never type a secret into a page, and never save browser login state. If a page needs a login you don't have, report it; don't look for credentials.

## Reporting what you saw
- For each check, give the URL, the viewport (for example 390×844 or 1280×800), what you did, what you saw, and pass or fail.
- Back every visual claim, such as contrast, layout or a focus ring, with a screenshot: `multica issue comment add <issue> --content-file report.md --attachment shot.png`.
- A check you couldn't run is "not run", never "passed". Say why.
- When a check asks what a page shows, look at the page. Don't infer it from the API or the HTML. If the browser can't show it, say so.
