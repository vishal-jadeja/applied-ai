---
aliases: [Loop Engineering]
---

# Loop Engineering

Part of [[AI-Learning]].

## What it is
Designing the repeated control loop an agent runs through: observe → plan/reason → act → check result → repeat (or stop). This is the "agent loop" — ReAct-style reasoning loops, multi-step tool-use loops, self-correction loops (generate → critique → revise).

Different from a single LLM call: the loop's structure determines whether the agent can recover from mistakes, knows when to stop, and doesn't spin forever or burn tokens re-deriving the same conclusion.

## Core concepts to learn
- [ ] ReAct pattern (Reason + Act) — interleaving reasoning traces with tool calls
- [ ] Stopping conditions — how the agent decides "done" vs. "keep iterating" (task completion checks, max-iteration caps, confidence thresholds)
- [ ] Self-critique / revision loops — generator produces output, critic evaluates it, reviser fixes it (exactly the [[Genora]] writer→critic→reviser shape)
- [ ] State/memory across loop iterations — what gets carried forward each turn vs. re-fetched vs. dropped
- [ ] Loop failure modes — infinite loops, oscillation (fix A breaks B, fix B breaks A), reward hacking the stopping condition
- [ ] Multi-agent loops — when one agent's loop output feeds another agent's loop input (pipeline vs. true multi-agent debate/negotiation)

## Why this matters for me
- [[Genora]]'s writer→critic→reviser is literally a loop-engineering problem: how many revision passes, when does the critic approve, how to avoid the reviser undoing the writer's voice-matching.
- Relevant to any RAG pipeline that needs a "did retrieval actually answer the question, or should I re-query" loop.

## Notes
- 
