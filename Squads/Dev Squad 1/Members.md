# Dev Squad 1: members

| Setting | Value |
|---|---|
| Name | Dev Squad 1 |
| Description | Designing Building and Testing Software |
| Leader | Tech Lead (agent, Claude (dev-ai-1) · `claude-opus-5-5`) |
| Members | 13 including the leader: 11 agents besides the leader, and 1 human |
| Instructions | `Instructions.md` (new) / `Archive/Squads/Dev Squad 1/Instructions_old.md` (live until the new set is deployed) |

## Members

The **role** is the text set on each member in Multica. It isn't just a label: Multica puts it into the Squad Roster that the leader sees on every run, and the leader picks members by their role and skills. Keep roles accurate and focused on what each member does.

These roles describe the new flow, so apply them in Multica together with the new prompt set and `Instructions.md`. Until then, the live roles are in `Archive/Squads/Dev Squad 1/Members_old.md`.

| Member | Type | Runtime · model | Role | Config |
|---|---|---|---|---|
| Product Owner | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Refines underspecified issues into a goal and labelled acceptance criteria (AC1, …). Doesn't decide what gets built or its priority. | `Agents/Product Owner/` |
| Researcher | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Investigates open technical questions with evidence and writes up the options. Doesn't decide or build. | `Agents/Researcher/` |
| Architect | Agent | Claude (dev-ai-1) · `claude-fable-5-1` | Owns system design, API contracts, data models and technology choices; writes ADRs. Reviews every code PR before testing. | `Agents/Architect/` |
| Frontend Designer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Designs web UI in Figma, with all states, breakpoints and accessibility. Required before any frontend engineering. | `Agents/Frontend Designer/` |
| Frontend Engineer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Builds frontend changes from the Figma design and opens a PR. | `Agents/Frontend Engineer/` |
| Backend Engineer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Builds backend changes from the Architect's design and opens a PR. | `Agents/Backend Engineer/` |
| Tester (Claude) | Agent | Claude (dev-ai-1) · `claude-sonnet-5-5` | Hunts for defects in a PR: edge cases, failure paths, misuse. Runs on Claude; alternate with Tester (Codex). | `Agents/Tester/` |
| Tester (Codex) | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Hunts for defects in a PR: edge cases, failure paths, misuse. Runs on GPT; alternate with Tester (Claude). | `Agents/Tester/` |
| Security Analyst | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Reviews every code change for security. Verdict: APPROVED, CHANGES REQUESTED or BLOCK. | `Agents/Security Analyst/` |
| QA | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Verifies each acceptance criterion and checks for regressions, after the Tester and Security Analyst. | `Agents/QA/` |
| Doc Owner | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Updates Confluence documentation and release notes once work ships, and finalises the Architect's drafts. | `Agents/Doc Owner/` |
| Nathaniel Greenwood | Human (workspace owner) | — | Workspace owner. Approves refinement, merges PRs and takes escalations. | — |

The leader receives work assigned to the squad and delegates by @mention. Members never receive the squad instructions; they only receive their own prompt and Workspace Context.

Exported from Multica on 2026-10-18. Squad, member and agent IDs aren't included, because a recreated workspace generates its own.
