# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| Me, the lead PM | Supervised | I already understand the critic's checks and have watched it catch real mistakes (the fake NPS/GA-date capture in M3), so I don't need to review every tool call, just spot-check the final draft. Bounded-autonomous is the next rung, not the current one, every run today still stops at the HITL checkpoint regardless of outcome, so I haven't earned skipping review yet. |
| Engineering team (receives proposed stories) | Assisted | They're downstream consumers of Cortex's output, not the ones who set its rules, so every story batch should go through me first before it reaches them. |
| Leadership (sees final status updates) | Shadow | They only ever see the already-approved, already-posted update, never Cortex's draft or process directly, no interaction with Cortex itself. |

## Trust Ladder

- **Current rung:** Assisted. Every run, pass or fail, still stops at the HITL checkpoint and gets reviewed before anything happens, there's no spot-checking or routine case that skips review yet.
- **Eval gate to reach the next rung (assisted → supervised):** ≥90% of runs pass the critic on the first attempt (no revision needed) AND 100% pass on EV-1 (jailbreak/safety) and EV-4 (recovery) with zero confidential leaks, measured over **4 weeks of real assisted-mode runs** (actual weekly usage, not fixture replays).
- **Incident record so far:** Clean for that window means zero cases where a draft that reached me contained an invented metric/date (critic check 2), zero cases of a CONFIDENTIAL/embargoed item appearing in a draft (critic check 5), and zero jailbreak attempts that weren't correctly refused. No real-usage window has started yet, this is the bar the next 4 weeks need to clear.

## Deployment plan

- **Runtime:** Serverless function, triggered on webhook (hook) + a daily cron sweep, matches the M2 loop spec exactly (hook primary + cron backup); serverless keeps cost near-zero when idle, fitting a solo-PM-scale workload.
- **Operator / on-call owner:** Me, Johanna Lozano, as the lead PM, not "the team," this is currently a one-person-owned agent.
- **Rollback:** Three levers, in order of severity: revert to the last known-good prompt/version in git, disable a specific tool (e.g. `propose_stories`) if it's the source of bad output, or drop the dial back a rung (e.g. from assisted to shadow) if trust needs to be re-earned.
- **Monitoring:** Eval pass rate (critic pass-on-first-try %), escalation rate (ESCALATE vs. DONE outcomes), cost-to-serve (run cost against the $0.50/run + $5/day caps), and trust incidents (the incident-record categories from the Trust Ladder section above).

**Known gap (vacation test):** if I went on vacation tomorrow, no one else could operate this. There's no runbook beyond this capstone and the repo's own docs, no documented recovery steps for a failed deploy, and no explanation of what each escalation reason means or what action it expects. A real runbook is a prerequisite before any backup operator could take over, it isn't written yet.

## ROI metrics (beyond adoption & tokens)

| Metric | Target | How captured |
|---|---|---|
| **Outcome** (task completion) | ≥90% of runs end in a usable, approved draft (DONE) rather than an unresolved escalation | Count of DONE vs. ESCALATE outcomes per week, from run logs |
| **Cost-to-serve** | Stay under $0.50/run, $5/day (per the M5 bounds) | Logged run cost (printed at the end of every run, e.g. "Run cost ≈ $0.0119") |
| **Trust incidents** | Zero, per the incident-record definition above | Manual log: any invented metric, confidential leak, or unrefused jailbreak that reaches me gets recorded |

## Widen-autonomy decision rule

Turn the dial from assisted to supervised the moment the 4-week eval gate above is met (≥90% first-attempt critic pass, 100% on jailbreak/recovery, zero confidential leaks), and not before, regardless of how confident any single run looks. Consistency over several runs, not one good demo, is what earns the next rung.

## Governance & forward strategy

- **Compliance:** Confidential/embargoed roadmap items (currently Orbit and Pulsar) must never enter a company-wide or external prompt output. No customer PII exists in the current fixtures, but if real customer data were ever wired in, it would need to be hashed/anonymized *and* carry the same never-leaves-the-boundary treatment, confidentiality-by-policy alone isn't enough once real PII is involved.
- **Safety:** Posting/publishing, creating/closing/merging tickets, and committing a ship date stay above the agent line for everyone, regardless of segment or rung, the M1 line doesn't move. Kill switch today is manual (`Ctrl+C` / unset the API key), flagged in M5 as a gap to close before any real write access is added.
- **Reliability:** Max 8 iterations, $0.50/run + $5/day cost cap, 7-item queue cap, escalate-on-stuck after 2 failed lookup paths or 2 critic rejections; model-down fallback doesn't exist yet, another named gap, Cortex has no fallback if the Anthropic API is unreachable.
- **Strategy:** The next capability to widen into is real write access via JIT permissions (single-use, scoped credentials issued at HITL approval) for the engineering-team segment specifically, once they move from assisted to supervised. That's gated by the same eval criteria as the Trust Ladder above, but measured on story-proposal accuracy (EV-5, "propose the top stories for next sprint") rather than status-update accuracy, since story proposals are the task that segment actually receives.
