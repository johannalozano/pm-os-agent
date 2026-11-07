# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | · |
| Decide relevant context | M | M | M | HITL | Cortex decides, human approves |
| Draft the update | H | L | M | HITL | Cortex drafts, human approves |
| Decide tone/commitment level | L | H | L | Above | · |
| Flag at-risk/escalation | H | L | H | Below | · |
| Choose what to escalate | M | M | L | HITL | Cortex drafts picks, human approves |
| Propose a story batch (capped) | H | L | H | Below | · |
| Post an update / approve a company-wide one | L | H | H | Above | · |

## Agent anatomy (sketch)

- **Model:** Claude Haiku 4.5 (`claude-haiku-4-5-20251001`) as the default, cheap and fast, good enough for routine status drafts and tool-calling. Escalate to a frontier model (e.g. Claude Opus/Sonnet) when the critic rejects twice in a row (signals the task is harder than expected) or when a genuinely ambiguous judgment call comes up (e.g. conflicting signals about whether something counts as a Sev-1).
- **Tools:** `get_project`, `get_activity`, `search_past_updates`, `get_roadmap`, `get_norms` (all read-only lookups) and `propose_stories` (capped, queues for approval, creates nothing). Deliberately absent: no `post_update`, no `create_ticket`, no `merge`/`close` tool exists at all.
- **Memory:** persists across runs, roadmap, team norms, and past-update history (used for tone/precedent matching). Purged per-run: the specific draft/conversation state, so each run starts clean rather than compounding context.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

- **Decide relevant context** sits at HITL because it's medium on all three axes, no single factor is safe enough to leave fully unsupervised, deciding factor: measurability (hard to tell after the fact if the right context was chosen).
- **Draft the update** sits at HITL because it's highly reversible and low blast radius, but only medium to verify, deciding factor: measurability (a "good enough" draft is fuzzy to judge without a human glance).
- **Decide tone/commitment level** sits above the line because it's low to reverse, has a high blast radius, and is low to verify, deciding factor: blast radius (a false commitment to leadership is a real-world promise, not just text).
- **Choose what to escalate** sits at HITL because it's medium reversibility and blast radius, but low to verify, deciding factor: measurability (a silent missed escalation leaves no trace to catch).
- **Post an update / approve a company-wide one** sits above the line because it's low to reverse and has a high blast radius, even though it's highly measurable, deciding factor: reversibility (once sent, it can't be unsent).

## Hardest call

The hardest calls were **"Decide relevant context"** and **"Choose what to escalate."** Both felt safe to hand fully to Cortex at first glance (it's just reading, it's just flagging), but scoring them honestly showed low/medium measurability on both, a wrong call here is silent: nothing breaks, nothing bounces back, it just quietly happens. That's what moved both from a flat call to HITL instead of leaving them below the line. In both cases the deciding axis was **measurability**, not reversibility or blast radius, the risk isn't that it's hard to undo or damaging, it's that we'd have no way to notice it went wrong.
