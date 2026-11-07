# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Hook (primary) + Cron (backup)

A new task/request (an inbound email, ticket, or message) is a real event, so Cortex should fire on it immediately rather than wait for a sweep, nothing gets missed. A daily cron sweep runs as a backup, to catch anything that slipped through without firing a clean event.

Ruled out: **Heartbeat**, a workaround for systems that can't emit real events; ours can, so polling on a fixed cadence would be wasted runs and cost creep. **Goal**, each run is a single, bounded task (one draft + one story batch), not an open-ended outcome to iterate toward.

**Idempotency / dedupe:** content-based, not ID-based. If the same underlying request arrives twice from different sources (e.g. an email, then a ticket, minutes apart), Cortex compares the substance of the request, what's being asked and which project/context it references, and treats it as one request with multiple source mentions, not two separate drafts.

## 2. Goal / definition of done

A run is done the moment the drafted status update, and any proposed stories, are written and sitting in the approval queue, and Cortex has made zero posts or commits. Nothing goes out, nothing is created in a tracker, until a human approves it.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | The critic passes the draft | Stop at the HITL checkpoint, update + stories queued, nothing posted |
| **Stuck / give up** | A required data fetch fails on 2 separate attempts/paths, **or** the critic rejects the draft more than 3 times | Halt, log the reason, escalate to a human instead of looping |
| **Escalate to human** | A risk emerges that conflicts with a committed date (e.g. a blocker reported by email) · the draft implies a public commitment · a proposed story requires something not currently available (e.g. new headcount/resource) | Held at the HITL checkpoint for human review, same as success, but flagged with why |

## 4. State

Persists across runs: recent past updates (for tone/precedent matching), the roadmap and team norms (ground truth for what's confidential/embargoed and what rules apply), and a log of handled requests, keyed by request substance rather than ID, so the dedupe check in §1 works across runs, not just within one.

## 5. The five things a loop can lean on

`state` is always-on. `connectors` only if you already have one wired (e.g. a Jira key or Google MCP), otherwise just note it as a plan. `skills`, `subagents`, `work tree` scale with autonomy; "not needed yet, because…" is a valid answer.

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Needed, planned. Once other actors or concurrent runs are in play, isolate each run's workspace so they can't collide on shared files. |
| **Skills** (reusable capabilities) | Not needed yet, because the current tool list already covers our one task type; revisit once we build multiple composite routines. |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet, fixtures only. Planned: Jira, HubSpot, Slack, Gmail. |
| **Subagents** (independent check when the loop can't grade itself) | Already covered, the critic *is* our independent check. |
| **State tracking** | See §4 above. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

`00-build/agent.py`
