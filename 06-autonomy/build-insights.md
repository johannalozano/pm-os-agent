# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you, by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

The build itself fought back early, the starter template was wired for OpenAI and I'd already spent my budget on an Anthropic key, so the first real work was porting the whole tool-calling format over, and even then the critic kept returning unparseable output until we forced a structured tool call instead of asking for free-text JSON. Later, the orchestration step was its own kind of friction: I initially wanted to add a parallel-fetch subagent for speed, and only after checking with the instructor did I realize that was unnecessary scope creep for a domain this fixed and scoped, I had to go back and strip it out of the orchestration map and roster.

## Learning

1. A bound only counts if it's enforced outside the model, a sentence in a prompt telling Cortex not to do something is not the same as a counter or a credential that makes the bad action impossible.
2. An agent can't grade its own work reliably, the independent critic subagent caught an invented NPS score, a falsely-closed issue, and an unauthorized GA-date commitment that the drafting model itself would never have flagged in its own output.
3. JIT/ephemeral permissions were new to me, instead of giving an agent a standing key that's live and open all the time, you issue a single-use, scoped credential only at the moment a human approves something, and it expires on use, so even a confused or compromised agent has no door to walk through until one is deliberately unlocked for one specific action.
4. Splitting into subagents has a real coordination cost, so it's only worth it when there's a concrete reason (an independent validator, context-window pressure), not just "it could theoretically go faster."

## Aha moment

Watching the iteration-cap trip mid-revision, not mid-drafting, the critic had already rejected a draft and Cortex was in the middle of fixing it, but the cap stopped it before it could finish. That's when autonomy design clicked for me: the bound doesn't need the agent to be "done" with its reasoning to be useful, it just needs to be the thing that decides when to stop, even if that's uncomfortable mid-thought.

## What you'd do differently

I'd write the operational runbook alongside the build instead of treating it as a deferred gap, right now if I were unreachable, no one else could actually run or recover this agent, and that's a real production blocker I only surfaced at the very end, in M6.
