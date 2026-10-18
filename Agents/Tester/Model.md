# Model: Tester

This prompt is shared by two agents in Multica, used in sequence by the Tech Lead (never in parallel).

| Agent | Runtime | Model | Thinking level |
|---|---|---|---|
| Tester Claude | Claude Code (Claude Max 20x) | Claude Sonnet 5.5 | High |
| Tester Codex | Codex (ChatGPT Pro $100) | GPT-6.1 Sol | High |

**Why:** Adversarial defect-hunting benefits from models other than the engineers' Claude Opus 5.5, so each tester sees the change differently. Sonnet 5.5 keeps Tester Claude cheaper on the shared Claude allowance. Two providers also mean one exhausted allowance doesn't stop testing; the Tech Lead's one-fallback rule switches testers.

**Revisit if:** Tester Claude misses defects the other tester finds (try Opus 5.5), or either allowance runs short (try Medium).
