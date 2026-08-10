# Applied AI — Concept Plan

Top-priority track — this IS the role. Learn top-to-bottom; each stage feeds the next; projects in `ai-projects` land after their theory stage. Priorities: **P0** interview-critical · **P1** strongly expected · **P2** differentiator.

Checkpoint after each stage: matching section in [INTERVIEW-QUESTIONS.md](INTERVIEW-QUESTIONS.md), answered aloud from memory.

## Stage 1 — Foundations `P0`

- [ ] How LLMs work operationally: tokens, context windows, sampling params (temperature/top-p)
- [ ] Token economics: pricing, input vs output cost, latency vs tokens
- [ ] Context engineering: what goes in the window and why — instructions, retrieval, memory, tool results
- [ ] Harness engineering vs prompt engineering — the system around the model
- Resources: [Anthropic — Building Effective Agents](https://anthropic.com/research/building-effective-agents), Context Engineering survey (arXiv)

## Stage 2 — Structured outputs + tool calling `P0`

*→ build ai-projects #1*

- [ ] Schema-constrained generation; native structured-output modes
- [ ] Validation (Pydantic), repair loops, fallback chains
- [ ] Function calling: tool contracts, argument validation, idempotency
- [ ] Failure modes: hallucinated tool calls, malformed JSON

## Stage 3 — RAG `P0`

*→ build ai-projects #2 (with stage 4)*

- [ ] Chunking strategies and their failure modes
- [ ] Embeddings: models, dimensions, similarity metrics
- [ ] Hybrid search (BM25 + vector), reranking, freshness
- [ ] Agentic RAG — when retrieval becomes a tool the model drives
- Resources: [Weaviate — Intro to RAG](https://weaviate.io/blog/introduction-to-rag), [Agentic RAG](https://weaviate.io/blog/what-is-agentic-rag)

## Stage 4 — Evals `P0`

*Before agents on purpose — evals gate everything after. #1 differentiator in 2026 loops.*

- [ ] Golden sets, regression tests, adversarial tests
- [ ] LLM-as-judge: setup, calibration against human labels, drift
- [ ] Retrieval evals: recall, precision, grounding, attribution, citation quality
- [ ] Silent eval regressions — why they're the scariest failure mode
- Resources: [LangSmith docs](https://docs.smith.langchain.com)

## Stage 5 — Agents `P0`

*→ build ai-projects #3*

- [ ] ReAct, plan-and-execute, reflection
- [ ] Guardrails: loop budgets, tool budgets, termination conditions
- [ ] Graceful degradation; runaway-agent prevention
- [ ] Production failure modes catalogue (stale retrieval, tool-call hallucination, cost blowups)

## Stage 6 — Decision framework `P0`

- [ ] Fine-tuning vs in-context learning vs RAG vs distillation — when each is the WRONG tool
- [ ] Latency / quality / cost / reliability trade-offs across the stack — argue any corner

**Checkpoint →** P0 complete = can whiteboard an LLM feature end-to-end with evals and failure handling.

## Stage 7 — Memory, MCP, multi-agent `P1`

*→ build ai-projects #4*

- [ ] Memory systems: short-term buffers, long-term vector recall, context compression ([Letta](https://letta.com))
- [ ] MCP: what it standardizes, tool discovery ([modelcontextprotocol.io](https://modelcontextprotocol.io))
- [ ] Multi-agent: LangGraph, supervisor patterns, handoffs, when multi-agent is over-engineering ([LangGraph docs](https://docs.langchain.com/oss/python/langgraph/overview))

## Stage 8 — Inference stack `P1`

- [ ] Prefill vs decode latency — why they optimize differently (TTFT vs TPOT)
- [ ] KV cache: management, eviction, reuse, memory pressure
- [ ] Continuous batching, paged attention
- [ ] Quantization: INT8/INT4/FP8, AWQ, GPTQ — when it hurts quality
- [ ] Speculative decoding vs quantization vs distillation trade-offs
- [ ] Prompt caching vs semantic caching
- [ ] Model routing, graceful fallback, degraded-mode UX

## Stage 9 — Observability + safety `P1`

*→ build ai-projects #5*

- [ ] Traces, spans, tokens, latency, errors, drift as first-class discipline
- [ ] Prompt injection defense, data leakage prevention, permission boundaries
- [ ] Multi-tenant isolation, cache safety, cross-user contamination
- [ ] PII redaction, sandboxed execution

## Stage 10 — Depth extras `P2`

- [ ] Cost attribution per feature / workflow / tenant
- [ ] vLLM / SGLang serving internals
- [ ] Reference builds: [ai-engineering-hub](https://github.com/patchy631/ai-engineering-hub)

## Out of scope (deliberate)

Training from scratch, deep ML math, RLHF implementation — interviews test *using and operating* models, not training them.
