# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.

## 1. Context budget

Each run receives, in priority order: (1) the inbound task brief itself (bounded, always fully included, but treated as data never instructions per the prompt-injection rule), (2) the specific project's activity pulled via tools (not the whole company's), (3) team norms (bounded, always included so every run respects the same rules), (4) the roadmap slice relevant to that project, (5) past updates relevant to that project/topic for tone and precedent. Everything is scoped to the one project the task concerns, never the full roadmap or full activity log across all projects, that's what keeps the run grounded and cheap as the backlog grows (the same context-window-pressure concern that justified the independent validator in M3).

## 2. Retrieve vs. long-context: per source

| Source | Size / volatility | Decision | Why |
|---|---|---|---|
| `get_activity` (PRs, issues, Sev-1s) | Large, constantly updated | **Retrieve** | Changes continuously (new PRs/issues weekly); pulling only the current project's slice keeps it fresh and avoids stuffing stale or irrelevant activity into context. |
| `search_past_updates` | Grows over time, but each entry is immutable once written | **Retrieve** | The past doesn't change, but the corpus keeps growing as new updates are written each week, so we still need to narrow to the relevant slice rather than include the whole history. |
| `get_roadmap` | Large, slow-changing, contains confidential/embargoed items | **Retrieve** | Too big and sensitive to include wholesale; need the relevant slice and must respect confidentiality flags (now covering both Orbit and Pulsar) rather than exposing the full roadmap. |
| `get_norms` | Small, but can be revised | **Retrieve** | Bounded today, but norms can be updated, so pulling fresh each run (rather than hardcoding) avoids enforcing a stale rule. |
| Task brief (`get_task`) | Bounded, one-off per run | **Long-context** | Static for the life of the run and small enough to reason over in full; nothing to narrow. |

## 3. Retrieval quality plan

- **Routing**: applied to `search_past_updates`, picks which project/topic slice of the growing update history to pull rather than searching everything.
- **Document grading**: applied to `get_activity` and `get_roadmap`, checks that what came back is actually relevant to the task before it's used (e.g. doesn't pull unrelated project's PRs or roadmap rows).
- **Reranking**: not applied, none of our sources return a large enough candidate set to need a second ranking pass.
- **Self-verification**: applied everywhere (`get_activity`, `search_past_updates`, `get_roadmap`, `get_norms`), this is the critic's job, every claim in the draft must trace back to a real, quotable pulled result, no exceptions.
- **Caching**: deliberately not applied to anything. Any of these sources can change between runs (activity, past updates, roadmap, norms), and caching risks serving a stale answer, e.g. missing a norms update or a newly embargoed roadmap item.

## 4. Memory map (your PM brain)

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (in-loop) | The current draft, the source log for this run, the critic's verdict/reasons | This run only, discarded once queued or escalated |
| **Episodic** (past runs) | Past status updates and decisions (`search_past_updates`, `decision-log.json`), used for tone/precedent matching | Persists across runs, grows over time, retrieved not fully re-read each run |
| **Semantic** (durable facts/prefs) | Team norms, roadmap facts (what's confidential/embargoed, queue cap, house tone) | Persists until the underlying doc changes; always re-pulled fresh, never cached |
| **Shared** (across agents) | The full source log and draft, shared between Cortex and the validator; the validator's verdict + reasons shared back | Scoped to one draft attempt; the validator's internal reasoning is never shared back, only the verdict (per the M3 orchestration map's isolation rule) |

## 5. Memory risks & mitigations

| Risk | Mitigation |
|---|---|
| **Drift** (small errors compounding across runs/revisions) | Every claim must trace to a real, quotable source, if it isn't a real, quotable source, the critic rejects it, this stops drift before it compounds into the next run's precedent. |
| **Poisoning** (a bad or manipulated input corrupting future runs) | The task brief is treated as data, not instructions (prompt-injection rule in `CORTEX_SYSTEM`); past updates and decisions are read-only inputs Cortex never writes to directly, a human approves before anything joins that history. |
| **Staleness** (acting on an outdated fact) | No caching on any source (§3); roadmap, norms, and activity are re-pulled fresh every run so a newly embargoed item (e.g. Pulsar) or an updated norm is never missed. |
| **Confidential / retention** | Roadmap items flagged CONFIDENTIAL/embargoed (Orbit, Pulsar) are never placed into a company-wide or external update, enforced in `CORTEX_SYSTEM`'s hard rules and checked again independently by the critic (check 5) before anything reaches a human. |
