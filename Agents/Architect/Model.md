# Model: Architect

| Setting | Value |
|---|---|
| Runtime | Claude Code (Claude Max 20x) |
| Model | Claude Fable 5.1 |
| Thinking level | High |

**Why:** Design mistakes are the most expensive to unwind later, and its code reviews check work against its own design. Design runs are rare; reviews are frequent but short. Fable is capped at 50% of the weekly Claude allowance, which is why only this agent uses it.

**Revisit if:** Fable's share of the weekly allowance starts to bite. Drop to Opus 5.5 at xhigh, or move code review to a dedicated Code Reviewer agent.
