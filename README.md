# Cortex, a PM Chief-of-Staff Agent

> A chief-of-staff agent, drafter + independent validator, that triages a PM task, pulls internal state (project, activity, roadmap, norms, past updates), and preps a status update plus a story batch, so the team approves instead of assembling from scratch.

_Johanna Lozano · Agentic Workflows & Loops Cohort · November 2026_

This repo is my final project for the Agentic Workflows & Loops Certification, **Cortex**. Each module's artifact lives in its own folder; this README is the dashboard and the pitch.

---

## Module artifacts

### M1 · The Agent Line
- **Agent-line map**: [`01-agent-line/agent-line-map.md`](01-agent-line/agent-line-map.md)

### M2 · Loop Engineering
- **Loop spec**: [`02-loop-design/loop-spec.md`](02-loop-design/loop-spec.md)

### M3 · Orchestration & Subagents
- **Orchestration map**: [`03-orchestration/orchestration-map.md`](03-orchestration/orchestration-map.md)

### M4 · Choose What Each Agent Step Is Allowed to See
- **Memory & context plan**: [`04-memory-context/memory-and-context.md`](04-memory-context/memory-and-context.md)

### M5 · Bounds & Evals
- **Bounds & evals**: [`05-bounds-evals/bounds-and-evals.md`](05-bounds-evals/bounds-and-evals.md)

### M6 · Autonomy & Production
- **Production & autonomy plan**: [`06-autonomy/production-and-autonomy.md`](06-autonomy/production-and-autonomy.md)
- **Prototype write-up**: [`06-autonomy/prototype.md`](06-autonomy/prototype.md)
- **Build insights**: [`06-autonomy/build-insights.md`](06-autonomy/build-insights.md)

---

## Ship plan

### Autonomy dial (per segment)
- Me, the lead PM → Supervised, I already understand the critic's checks and have watched it catch real mistakes, so I spot-check the final draft rather than reviewing every tool call.
- Engineering team (receives proposed stories) → Assisted, every story batch goes through me first before it reaches them.
- Leadership (sees final status updates) → Shadow, they only ever see the already-approved update, never Cortex's draft or process.

### Trust Ladder rung + eval gate
Current rung: **Assisted** (every run, pass or fail, still stops at the HITL checkpoint and gets reviewed, there's no spot-checking yet).
Eval gate to climb to Supervised: ≥90% of runs pass the critic on the first attempt (no revision needed) AND 100% pass on the jailbreak/safety and recovery eval cases, with zero confidential leaks, over **4 weeks of real assisted-mode runs**.
Clean incident record: zero invented metrics/dates reaching me, zero confidential/embargoed items appearing in a draft, zero unrefused jailbreak attempts.

### Deployment plan
- Runtime: serverless function, hook-triggered on an inbound PM task + a daily cron backup sweep (matches the M2 loop spec); pay-per-run suits bursty, solo-PM-scale load.
- On-call owner: me, Johanna Lozano, as the lead PM, not "the team." **Known gap:** no runbook exists yet beyond this repo's own docs, so if I were unreachable, no one else could actually operate it.
- Rollback: revert to the last known-good prompt/version in git, disable a specific tool (e.g. `propose_stories`), or drop the dial back a rung.
- Monitoring: critic pass-on-first-try %, escalation rate (ESCALATE vs. DONE), cost-to-serve against the $0.50/run + $5/day caps, and trust incidents.

### ROI metrics + widen-autonomy rule
- Outcome: ≥90% of runs end in a usable, approved draft (DONE) rather than an unresolved escalation.
- Cost-to-serve: stay under $0.50/run, $5/day; captured from the run cost already printed at the end of every run.
- Trust incidents: zero, per the incident-record definition above, logged manually.
Widen rule: Cortex climbs from Assisted to Supervised the moment the 4-week eval gate is met, and not before, regardless of how confident any single run looks. Consistency over several runs, not one good demo, earns the next rung.

### Governance & strategy
- Compliance: confidential/embargoed roadmap items (Orbit, Pulsar) never enter a company-wide or external output; real customer PII, if ever wired in, would need to be hashed/anonymized *and* carry the same never-leaves-the-boundary treatment.
- Safety: posting, creating/closing/merging tickets, and committing a ship date stay above the agent line for everyone, regardless of segment or rung. Kill switch today is manual (process kill / unset the API key), flagged as a gap to close before real write access is added.
- Reliability: max 8 iterations, $0.50/run + $5/day cost cap, 7-item queue cap, escalate-on-stuck after 2 failed lookup paths or 2 critic rejections; no model-down fallback yet (named gap).
- Strategy: next widen into real write access via JIT permissions (single-use, scoped credentials issued at HITL approval) for the engineering-team segment, gated by the same eval criteria but measured on story-proposal accuracy.

---

## Build insights

- **Friction point.** Porting the starter build off OpenAI onto Anthropic/Claude Haiku (budget constraint), then fighting unparseable critic output until I forced a structured tool call instead of free-text JSON. Also had to walk back an added "parallel-fetch" subagent after realizing it was unnecessary scope creep for a domain this fixed and scoped.
- **Key learning.** A bound only counts if it's enforced outside the model; an agent can't reliably grade its own work, which is why the independent critic caught things the drafter itself never would; and JIT/ephemeral permissions (a single-use, scoped credential issued only at approval, expiring on use) beat a standing key that's live all the time.
- **Aha moment.** Watching the iteration cap trip mid-revision, not mid-drafting, the bound doesn't need the agent to be "done" with its reasoning to be useful, it just needs to be the thing that decides when to stop.

---

_Certification submission, Agentic Workflows & Loops Certification._
