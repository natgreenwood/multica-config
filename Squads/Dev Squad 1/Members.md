# Dev Squad 1: members

| Setting | Value |
|---|---|
| Name | Dev Squad 1 |
| Description | Designing Building and Testing Software |
| Leader | Tech Lead (agent, Claude (dev-ai-1) · `claude-opus-5-5`) |
| Members | 13 including the leader: 11 agents besides the leader, and 1 human |
| Instructions | `Instructions.md` (new) / `Archive/Squads/Dev Squad 1/Instructions_old.md` (live until the new set is deployed) |

## Members

The **role** column is the text set on each member in Multica. It isn't just a label: Multica puts it into the Squad Roster that the leader sees on every run, and the leader picks members by their role and skills. Keep roles accurate and focused on what each member does.

| Member | Type | Runtime · model | Role (as set in Multica) | Config |
|---|---|---|---|---|
| Product Owner | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Product Lead, Conducts Refinements using Claude models | `Agents/Product Owner/` |
| Researcher | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Conducts all research using Claude models | `Agents/Researcher/` |
| Architect | Agent | Claude (dev-ai-1) · `claude-fable-5-1` | Owns system design, API contracts, data models, and technology choices. Produces ADRs (architecture decision records) so decisions are traceable. | `Agents/Architect/` |
| Frontend Designer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Designs Web UI | `Agents/Frontend Designer/` |
| Frontend Engineer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Frontend Engineer using Claude models | `Agents/Frontend Engineer/` |
| Backend Engineer | Agent | Claude (dev-ai-1) · `claude-opus-5-5` | Backend Engineer using Claude models | `Agents/Backend Engineer/` |
| Tester (Claude) | Agent | Claude (dev-ai-1) · `claude-sonnet-5-5` | Testing using Claude Models | `Agents/Tester/` |
| Tester (Codex) | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Testing using GPT Models | `Agents/Tester/` |
| Security Analyst | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Security reviews and analysis | `Agents/Security Analyst/` |
| QA | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Quality Assurance | `Agents/QA/` |
| Doc Owner | Agent | Codex (dev-ai-3) · `gpt-6.1-sol` | Creates and updates Project Documentation | `Agents/Doc Owner/` |
| Nathaniel Greenwood | Human (workspace owner) | — | Solutions Architect | — |

## Proposed roles (not yet applied)

These replace the roles above. Apply them in Multica **together with the new prompt set and `Instructions.md`, not before**: they describe the new flow, so applying them while the old instructions are live would send work to steps those instructions don't have. They fix outdated text: the Researcher said "Claude models", and the Product Owner was called "Product Lead". They also add the new duties (Architect code review, the Tester/QA split), and they drop model names except where the leader needs them to alternate between the testers.

| Member | Proposed role |
|---|---|
| Product Owner | Refines underspecified issues into a goal and labelled acceptance criteria (AC1, …). Doesn't decide what gets built or its priority. |
| Researcher | Investigates open technical questions with evidence and writes up the options. Doesn't decide or build. |
| Architect | Owns system design, API contracts, data models and technology choices; writes ADRs. Reviews every code PR before testing. |
| Frontend Designer | Designs web UI in Figma, with all states, breakpoints and accessibility. Required before any frontend engineering. |
| Frontend Engineer | Builds frontend changes from the Figma design and opens a PR. |
| Backend Engineer | Builds backend changes from the Architect's design and opens a PR. |
| Tester (Claude) | Hunts for defects in a PR: edge cases, failure paths, misuse. Runs on Claude; alternate with Tester (Codex). |
| Tester (Codex) | Hunts for defects in a PR: edge cases, failure paths, misuse. Runs on GPT; alternate with Tester (Claude). |
| Security Analyst | Reviews every code change for security. Verdict: APPROVED, CHANGES REQUESTED or BLOCK. |
| QA | Verifies each acceptance criterion and checks for regressions, after the Tester and Security Analyst. |
| Doc Owner | Updates Confluence documentation and release notes once work ships, and finalises the Architect's drafts. |
| Nathaniel Greenwood | Workspace owner. Approves refinement, merges PRs and takes escalations. |

The leader receives work assigned to the squad and delegates by @mention. Members never receive the squad instructions; they only receive their own prompt and Workspace Context.

Exported from Multica on 2026-10-18. Squad, member and agent IDs aren't included, because a recreated workspace generates its own.
