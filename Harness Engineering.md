---
aliases: [Harness Engineering]
---

# Harness Engineering

Part of [[AI-Learning]].

## What it is
Designing the scaffolding an LLM agent runs inside — the tools, permissions, context management, and control flow around the model — as opposed to the model itself. "Harness" = everything between the raw LLM call and the finished agent (tool definitions, sandboxing, retry/error handling, memory/context injection, permission prompts).

Claude Code itself is a harness: tool schemas, permission modes, context compaction, hooks, subagents — none of that is the model, all of it shapes what the model can reliably do.

## Core concepts to learn
- [ ] Tool/function-calling design — how tool schemas, descriptions, and error messages affect model behavior (bad tool descriptions → bad tool use, regardless of model quality)
- [ ] Context window management — compaction, summarization, what to keep vs. discard as a session grows
- [ ] Permission/sandboxing models — least-privilege execution, human-in-the-loop confirmation for risky actions
- [ ] Subagent orchestration — when to delegate to isolated agents vs. keep work inline, context isolation tradeoffs
- [ ] Hooks/lifecycle events — intercepting/modifying behavior at defined points (pre-tool-call, post-response, session-start)
- [ ] Error recovery — retries, fallback strategies, graceful degradation when a tool call or model response fails
- [ ] Observability — logging/tracing what the agent actually did, for debugging and eval

## Why this matters for me
- Directly applicable to [[Genora]] (writer→critic→reviser loop needs a harness deciding when to stop, retry, or escalate to the user) and [[DepLyx]] (agent needs sandboxed repo access + PR-creation tooling with guardrails).
- Distinct from prompt engineering — harness bugs look like "the model is dumb" but are actually "the scaffolding gave it bad tools/context."

## Reading list
- (fill in as found — papers, blog posts, source code worth studying)
