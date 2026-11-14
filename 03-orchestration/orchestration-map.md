# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Cortex's current design is a single, event-based agent: it pulls data, organizes it, drafts a status update and stories, and submits them for human review before anything is posted.

Scored against the four reasons to split:
- **Separation of concerns:** No. The core task (read → draft → queue) is one coherent, sequential job, not multiple contaminating kinds of work.
- **Parallelism:** No. The project's domain is scoped and fixed, pulling from multiple sources in parallel would add coordination cost without a real time-saving case at this scope.
- **Independent validator:** Yes. AI isn't a reliable critic of its own work, it can miss the same mistakes it made while writing. A separate model call, with its own context, is needed to catch problems before a human sees them.
- **Context-window pressure:** Yes (forward-looking). As the backlog and history pulled per run grow over time, one agent holding everything risks losing important details, even though the domain itself stays fixed.

**Decision:** Cortex splits into a small team of exactly two, a drafter (Cortex) and an independent validator (the critic). No further split is justified right now.

## 2. Topology

**Pattern:** single + subagent (one drafter, one independent validator)

```
[Inbound PM task/event] → [Cortex: pulls data, drafts update + stories]
                        → [Validator], fail → back to Cortex (max 2 revisions)
                                            → still failing → escalate (quoting the critic's last reasons)
                                            → pass → [PM review checkpoint] → queued, nothing posted
```

## 3. Roster

| Agent / subagent | Responsibility | Runs which Loop Spec |
|---|---|---|
| Cortex (drafter) | Reads the task, pulls data via tools, drafts the status update + proposed stories | Hook-triggered (M2 spec §1), with a cron backup sweep |
| Validator (critic) | Independently checks Cortex's draft against the checklist in §5 before a human sees it | Runs inline per Cortex draft attempt, no own trigger |

## 4. Communication & hand-offs

Plain in-process hand-off, no MCP/A2A protocol, just a direct function call between the two Python modules (`agent.py` → `critic.py`).

- **Cortex → Validator:** the full proposed draft text, plus the complete source log (everything Cortex pulled: project, activity, roadmap, norms). The validator needs the full source log, not just the draft, to verify every claim traces to a real, quotable source.
- **Validator → Cortex (on fail):** only the verdict + rejection reasons are passed back into Cortex's conversation, not the validator's internal reasoning, so Cortex can retry without the validator's scratch-thinking polluting its next attempt.
- **Validator → PM checkpoint (on pass):** the draft is written to `run-output/` and printed. No further agent is involved.

## 5. The validator

- **What the critic checks:**
  1. References the correct project + real PR/issue IDs from the pulled activity data.
  2. Every claim and figure traces to a real, quotable source, if someone asked "where did this come from," there's an exact tool result to point to. No invented numbers, no invented progress.
  3. If stories were proposed, the batch stays within the queue cap, or the output correctly flags/escalates if it would exceed it.
  4. Tone matches house style, and it avoids any commitment Cortex may not make (a firm date, a discount, a GA promise, marking a launch gate).
  5. Stays within team norms otherwise: posts nothing, commits nothing, creates/closes/merges nothing, leaks no CONFIDENTIAL/embargoed roadmap item.
- **Fail action:** revise, up to 2 times, with the critic's rejection reasons fed back to Cortex each time. If still failing after 2 revisions, escalate to a human, quoting the critic's last rejection reasons directly so the human isn't guessing why it stalled.
- **Pass action:** advances to the PM review checkpoint. It is never auto-sent, this stays above the M1 agent line regardless of the verdict.

## 6. State: shared vs isolated

- **Shared:** the source data (project, activity, roadmap, norms) and the draft itself, both agents see these.
- **Isolated:** the Validator never sees Cortex's own drafting conversation or reasoning, so it judges the output fresh rather than inheriting Cortex's blind spots. Conversely, the Validator's own reasoning process stays isolated from Cortex, only the final verdict + reasons are handed back on a failure, never the validator's full internal thinking.

## 7. Cost & latency budget

The validator adds exactly one extra model call per Cortex draft attempt, one critic check per draft, regardless of outcome. In the normal case (critic passes on the first try), that's one extra call on top of Cortex's own tool-calling turns, costing about $0.02 total for a full run (measured). At the revision cap of 2, the worst case is 3 draft attempts × 3 critic calls before escalating, roughly 2–3x the base run cost (~$0.04–0.06) and an estimated 10–15 extra seconds of latency before the PM sees anything, since each revise-and-recheck round-trip takes a few seconds. This is the forward-link to the bounds we'll formalize in M5.
