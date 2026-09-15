# Applied AI — Concept Plan

Top-priority track — this IS the role. Learn top-to-bottom; each stage feeds the next; projects in `ai-projects` land after their theory stage. Priorities: **P0** interview-critical · **P1** strongly expected · **P2** differentiator.

Checkpoint after each stage: matching section in [INTERVIEW-QUESTIONS.md](INTERVIEW-QUESTIONS.md), answered aloud from memory.

## Stage 1 — Foundations `P0`

- [ ] How LLMs work operationally: tokens, tokenizers, context windows, sampling params (temperature/top-p/top-k, stop sequences)
- [ ] Token economics: pricing, input vs output cost asymmetry, latency vs tokens
- [ ] Context engineering: what earns a place in the window and in what order — instructions, retrieval, memory, tool results
- [ ] Context rot: why quality degrades well before the window limit; needle-in-haystack vs real multi-fact reasoning
- [ ] Reasoning vs non-reasoning models: extended thinking, thinking budgets, when reasoning models are the wrong (slow, expensive) tool
- [ ] Harness engineering vs prompt engineering — the system around the model ([Harness Engineering.md](Harness%20Engineering.md))
- [ ] Nondeterminism: why temp 0 still varies, and building on validation instead
- Resources: [Anthropic — Building Effective Agents](https://anthropic.com/research/building-effective-agents), Context Engineering survey (arXiv), [loop & harness engineering](https://youtu.be/3uO6V05LBjU)

## Stage 2 — Prompt engineering `P0`

*The most-asked, most-underestimated topic. Treat prompts as versioned code, not text.*

- [ ] Zero-shot vs few-shot: how many examples, how to pick them, example ordering/recency bias
- [ ] Chain-of-thought: when it helps, when it's noise, why it's redundant on reasoning models
- [ ] Task decomposition: prompt chaining vs one mega-prompt — cost, debuggability, error isolation
- [ ] Role and structure: system vs user vs assistant turns, assistant prefill for steering output
- [ ] Instruction–data separation: delimiters/XML tags, why untrusted content must never look like instructions (feeds stage 12)
- [ ] Output control: format specification, JSON-mode vs prose, refusal and "I don't know" paths
- [ ] Negative instructions fail — say what to do, not what to avoid
- [ ] Self-consistency, self-critique in-prompt, and their cost multiple
- [ ] Prompts as code: templating, versioning, diffing, A/B against an eval set (never "it feels better")
- [ ] Cache-aware prompt layout: stable prefix first, volatile content last (pairs with stage 11 prompt caching)
- [ ] Meta-prompting / automated prompt optimization — when a model writes the prompt
- [ ] Failure taxonomy: instruction drift in long contexts, lost-in-the-middle, over-anchoring on few-shot format
- Resources: [Anthropic prompt engineering docs](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

## Stage 3 — Structured outputs + tool calling `P0`

*→ build ai-projects #1*

- [ ] Schema-constrained generation; native structured-output modes vs "please return JSON"
- [ ] Validation (Pydantic), repair loops, fallback chains
- [ ] Function calling: tool contracts, argument validation, idempotency
- [ ] Parallel tool calls, tool-choice forcing, and when to disallow tools entirely
- [ ] Schema design pressure: enums over free strings, minimal required fields, descriptions as prompt surface
- [ ] Failure modes: hallucinated tool calls, malformed JSON, silently truncated output at max_tokens
- Resources: [OpenAI structured outputs](https://platform.openai.com/docs/guides/structured-outputs)

## Stage 4 — RAG `P0`

*→ build ai-projects #2 (with stage 5)*

- [ ] Ingestion: document parsing (PDF/HTML/tables), cleaning, dedupe, metadata extraction
- [ ] Chunking strategies and their failure modes; structure-aware vs fixed-size vs semantic
- [ ] Embeddings: model choice, dimensions, similarity metrics, domain fit
- [ ] Vector index types (HNSW, IVF, flat) and their tradeoffs at scale — recall vs latency vs memory
- [ ] Hybrid search (BM25 + vector), reranking (cross-encoder), metadata filtering, freshness/TTL
- [ ] Index lifecycle: incremental updates, re-embedding on model change, deletion and tenancy filters
- [ ] Citations and grounding: attributing answers to source spans, abstaining when retrieval is weak
- [ ] Agentic RAG — when retrieval becomes a tool the model drives, query rewriting, re-query loops
- [ ] GraphRAG / structured retrieval — when relationships beat similarity
- Resources: [Weaviate — Intro to RAG](https://weaviate.io/blog/introduction-to-rag), [Agentic RAG](https://weaviate.io/blog/what-is-agentic-rag), [RAG explained](https://youtu.be/X0btK9X0Xnk), [RAG frameworks compared](https://florinelchis.medium.com/top-10-rag-frameworks-on-github-by-stars-january-2026-e6edff1e0d91)

## Stage 5 — Evals `P0`

*Before agents on purpose — evals gate everything after. #1 differentiator in 2026 loops.*

- [ ] The vocabulary (Anthropic's): **task** (test case + success criteria) → **trial** (one execution) → **transcript** (what happened) → **outcome** (what changed in the world) → **grader** (pass/fail judgment); eval **harness** runs it, eval **suite** groups tasks by capability
- [ ] Grader types: code-based (exact match, regex, DB/API state, schema checks — cheap, rigid) vs model-based (rubrics, pairwise, reference-based — flexible, biased) vs human (calibrates the other two) — complementary, not either/or
- [ ] Capability evals (what can it do — hard tasks, headroom) vs regression evals (did something break — should pass ~always); maintain both
- [ ] pass@k vs pass^k: "can eventually solve" vs "solves consistently" — which one your product actually needs
- [ ] Suite design: start with 20–50 realistic tasks from real failures; test "should do X" AND "should NOT do X" (don't reward over-triggering); grade outcomes, not rigid step sequences
- [ ] Golden sets, regression tests, adversarial tests
- [ ] LLM-as-judge: setup, rubric design, calibration against human labels, judge bias and drift
- [ ] Retrieval evals: recall, precision, grounding, attribution, citation quality
- [ ] Task evals: exact match, semantic similarity, structured-field accuracy — picking the metric that matches the failure you fear
- [ ] Online vs offline: shadow runs, A/B, production traffic sampled back into the golden set
- [ ] Human feedback loops: explicit ratings vs implicit signals (retries, corrections, escalations, abandonment), routing them into datasets
- [ ] Feedback isn't ground truth — a thumbs-down can mean wrong answer, wrong retrieval, bad UX, or user misunderstanding; the loop is feedback → failure analysis → root cause → fix → **new eval** → regression protection
- [ ] Silent eval regressions — why they're the scariest failure mode
- [ ] Evals in CI: gating deploys on prompt/model/index changes
- Resources: [LangSmith docs](https://docs.smith.langchain.com), [Ragas](https://docs.ragas.io), [LLM evaluation course](https://youtu.be/6W92_t9FveA)

## Stage 6 — Agents `P0`

*→ build ai-projects #3*

- [ ] ReAct, plan-and-execute, reflection
- [ ] Guardrails: loop budgets, tool budgets, wall-clock and cost ceilings, termination conditions
- [ ] Human-in-the-loop: approval gates on irreversible actions, resumable/interruptible runs
- [ ] Graceful degradation; runaway-agent prevention
- [ ] Loop failure modes: infinite loops, oscillation (fix A breaks B, fix B breaks A), reward-hacking the stopping condition
- [ ] Production failure modes catalogue (stale retrieval, tool-call hallucination, cost blowups)
- [ ] When an agent is over-engineering — fixed pipeline with narrow LLM decision points instead

## Stage 7 — Decision framework `P0`

- [ ] Fine-tuning vs in-context learning vs RAG vs distillation — when each is the WRONG tool
- [ ] Build vs buy: framework (LangChain/LlamaIndex) vs raw SDK, hosted vs self-hosted
- [ ] Model selection: capability tiers, routing cheap→expensive, benchmark skepticism (test on your own data)
- [ ] Latency / quality / cost / reliability trade-offs across the stack — argue any corner

**Checkpoint →** P0 complete = can whiteboard an LLM feature end-to-end with evals and failure handling.

## Stage 8 — Memory, MCP, multi-agent `P1`

*→ build ai-projects #4*

- [ ] Memory systems: short-term buffers, long-term vector recall, context compression ([Letta](https://letta.com))
- [ ] What carries forward per turn vs is re-fetched vs dropped — and how compaction loses things
- [ ] MCP: what it standardizes, tool discovery, server/client split ([modelcontextprotocol.io](https://modelcontextprotocol.io), [MCP playlist](https://youtube.com/playlist?list=PLKnIA16_Rmva_oZ9F4ayUu9qcWgF7Fyc0))
- [ ] Multi-agent: LangGraph, supervisor patterns, handoffs, when multi-agent is over-engineering ([LangGraph docs](https://docs.langchain.com/oss/python/langgraph/overview))
- [ ] Topology choice: sequential pipeline vs orchestrator/worker vs debate — and when a pipeline is not really multi-agent at all
- [ ] Context isolation as the real reason to spawn a subagent

## Stage 9 — Application layer: streaming, UX, reliability `P1`

*The part that separates a demo from a product. Rarely studied, always asked about in system-design rounds.*

- [ ] Streaming: SSE/websockets, token streaming, streaming partial structured output
- [ ] Perceived latency: TTFT as the UX metric, skeletons, progressive disclosure, cancellation
- [ ] Rate limits, exponential backoff, retries with jitter, request queueing and concurrency caps
- [ ] Idempotency across retries; never double-executing a side-effectful tool call
- [ ] Batch APIs and async jobs for non-interactive work (large cost savings)
- [ ] Timeouts, circuit breakers, provider failover
- [ ] Cost controls at the app layer: per-user/per-tenant budgets, hard caps, degraded-mode fallback

## Stage 10 — Multimodal + extraction `P1`

- [ ] Vision inputs: image tokens and their cost, resolution/detail tradeoffs, what VLMs still fail at
- [ ] Document AI: PDF/scan → structure, OCR vs native VLM extraction, tables and forms ([Docling](https://github.com/docling-project/docling))
- [ ] Classification and extraction pipelines — the highest-ROI, least-glamorous applied AI work
- [ ] Multimodal RAG: image+text indexes, captioning vs multimodal embeddings
- [ ] Voice pipelines: STT → LLM → TTS, turn-taking, latency budget per hop

## Stage 11 — Inference stack `P1`

- [ ] Prefill vs decode latency — why they optimize differently (TTFT vs TPOT)
- [ ] KV cache: management, eviction, reuse, memory pressure
- [ ] Continuous batching, paged attention
- [ ] Quantization: INT8/INT4/FP8, AWQ, GPTQ — when it hurts quality
- [ ] Speculative decoding vs quantization vs distillation trade-offs
- [ ] Prompt caching vs semantic caching — hit-rate design, invalidation, cache-key safety
- [ ] The full caching taxonomy: response cache (exact + semantic), embedding cache (don't re-embed unchanged docs), retrieval cache, external-API cache — and why the real question is never "can we cache this" but "**when does it become invalid**" (stale/mis-scoped caches are correctness and security bugs)
- [ ] LLM gateway pattern: one unified API over providers — routing, fallbacks, load balancing, cost tracking, per-key budgets ([LiteLLM](https://docs.litellm.ai), [gateway walkthrough](https://youtu.be/RN3baOpNA6w))
- [ ] Model routing, graceful fallback, degraded-mode UX
- [ ] Model deprecation & versioning: provider lifecycles, pinning vs floating versions, migration paths and re-eval on switch — DepLyx's core feature

## Stage 12 — Observability + safety `P1`

*→ build ai-projects #5*

- [ ] Traces, spans, tokens, latency, errors, drift as first-class discipline ([OTel GenAI semconv](https://opentelemetry.io/docs/specs/semconv/gen-ai/), [tracing with Arize](https://youtu.be/fHGSxOhWO-g))
- [ ] What to capture per trace stage: latency (p50/p95/p99), cost (input/output tokens), quality (eval scores, feedback), RAG signals (retrieved docs, similarity scores, context size), versions (model/prompt/retrieval) — so "quality dropped 90→75%" is answerable, not mysterious
- [ ] Guardrails architecture: input rails (injection/PII detection, validation, authn/authz) → LLM → output rails (schema, safety, policy checks); the model **proposes**, deterministic business rules **enforce** (e.g. LLM says "refund ₹50k", hard cap says "max ₹10k automated → human approval")
- [ ] Prompt injection defense — direct and indirect; privilege separation as the only boundary that holds
- [ ] Jailbreaks, content moderation, refusal handling and over-refusal as a product bug
- [ ] Multi-tenant isolation, cache safety, cross-user contamination
- [ ] PII redaction, data retention/training-opt-out, sandboxed execution
- [ ] Output-side risk: unsafe code execution, SSRF via tool calls, exfiltration through rendered links
- Resources: [OWASP Top 10 for LLM Apps](https://owasp.org/www-project-top-10-for-large-language-model-applications/), [AI guardrails & security masterclass](https://youtu.be/rQE3w8Qjx98), [LLMOps intro](https://youtu.be/NuWRAiYnjxw)

## Stage 13 — Fine-tuning & data `P2`

*Operating a fine-tune, not inventing training methods. Decision lives in stage 7; mechanics live here.*

- [ ] SFT basics; LoRA/QLoRA and why PEFT dominates in practice ([PEFT](https://huggingface.co/docs/peft))
- [ ] Preference tuning (DPO) at a conceptual level — what it buys over SFT
- [ ] Dataset curation: size, quality over volume, dedupe, leakage into eval sets
- [ ] Synthetic data generation and its failure modes (mode collapse, inherited bias)
- [ ] Distillation: big-model outputs → small-model speed/cost
- [ ] Embedding fine-tuning for domain retrieval — often a bigger win than tuning the generator
- [ ] Post-fine-tune ownership: eval suites, drift, retraining cadence, rollback

## Stage 14 — Depth extras `P2`

- [ ] Cost attribution per feature / workflow / tenant
- [ ] vLLM / SGLang serving internals ([vLLM docs](https://docs.vllm.ai))
- [ ] GPU basics for inference: VRAM math, batch size, why a 70B won't fit where you think
- [ ] Reference builds: [ai-engineering-hub](https://github.com/patchy631/ai-engineering-hub)

## Stage 15 — Tooling & model landscape `P1` *(interleave anytime — no dependencies)*

*Fluency, not depth. FDE interviews probe "what's out there and when would you pick it"; using these daily is the fastest way to have opinions.*

- [ ] Agentic coding tools: Claude Code / Cursor / Codex — what a coding agent's harness actually does; use one for every build in this plan ([agentic coding with Claude Code](https://youtu.be/K_KIQA849cs))
- [ ] Open vs closed models: the current landscape (Llama, Qwen, DeepSeek, Mistral, Gemma) — licensing, capability tiers, when self-hosting wins
- [ ] Hugging Face ecosystem: Hub, models/tasks/datasets, inference SDK — enough to find and run any open model
- [ ] Local inference: Ollama / LM Studio — run a small model locally once, understand quantized GGUF tradeoffs
- [ ] Provider API landscape: OpenAI-compatible APIs as the de-facto wire format, OpenRouter-style aggregators, choosing a provider (capability, latency, data policy, price)
- [ ] Vector DB market fluency: Pinecone / Qdrant / Weaviate / Chroma / pgvector — categorize by hosted vs embedded vs "your existing Postgres"
- Resources: [Generative AI playlist](https://youtube.com/playlist?list=PLKnIA16_RmvaTbihpo4MtzVm4XOQa0ER0), [Agentic AI playlist](https://youtube.com/playlist?list=PLKnIA16_RmvYsvB8qkUQuJmJNuiCUJFPL)

### Paper reproduction targets `2026 goal`

Candidates for the "read and reproduce research papers" goal — fill in as chosen.

- (empty)

## Out of scope (deliberate)

Pretraining from scratch, deep ML math, RLHF implementation, custom CUDA kernels — interviews test *using and operating* models, not training them. Stage 13 covers fine-tuning as an operational decision only.
