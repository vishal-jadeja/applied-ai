# Applied AI — Interview Q&A Bank

Answer aloud from memory first; then write the model answer. Grown per stage of [PLAN.md](PLAN.md).

## Foundations

**Q: Prompt is 195k tokens, context window is 200k — what breaks?**
A: Input and output share one budget. Only ~5k tokens remain for the answer, so the response truncates mid-generation; push input past 200k and the API rejects the call outright. A large input starves the output — this is a *capacity* failure, distinct from lost-in-the-middle, which is a *quality* failure at long context.

**Q: How does the model "remember" earlier messages in a chat?**
A: It doesn't. The model is stateless between calls. The harness resends the entire prior transcript on every request — "memory" is re-injected context, not model state. This is also why long conversations get slower and pricier per turn, and why history eventually has to be summarized or truncated.

**Q: Why is output pricing higher than input pricing on every provider?**
A: Prefill (input) processes all tokens in parallel — one pass, compute-bound, cheap per token. Decode (output) generates autoregressively — one token per forward pass, memory-bandwidth-bound, holds the KV cache the whole time. Output tokens cost real serial GPU time; input tokens amortize.

**Q: Two calls, same total tokens — 5000-in/100-out vs 100-in/5000-out. Which costs more and which is slower?**
A: The output-heavy call loses on both. Cost: its 5000 tokens are billed at the 3–5× output rate vs the cheap input rate. Speed: those 5000 output tokens decode serially (5000 forward passes), while 5000 input tokens prefill in one parallel pass. Same reason for both — output is serial + memory-bound, input is parallel + compute-bound.

**Q: Why does chat cost climb faster than linearly as conversations grow?**
A: Every turn resends the entire transcript, so each turn's input cost rises with conversation length — total is the sum of a growing series, i.e. quadratic. Not "same cost per turn × more turns"; each turn itself gets pricier. Fix: summarize or window old turns so resent input stays bounded.

**Q: One lever to cut both cost and latency the most?**
A: Reduce output tokens. Output is the expensive side of both ledgers — 3–5× the price and the serial-decode bottleneck that dominates latency. Trimming input helps cost and TTFT but pays off less because input is cheap and parallel. Cap max_tokens, request concise/structured output.

**Q: What's context engineering vs prompt engineering?**
A: Prompt engineering = wording one message. Context engineering = deciding what earns a place in the window at all: system instructions, retrieved chunks, memory, tool results, history — selection, ordering, compression, caching. Production quality lives here; a long context is not a strategy, it's a cost.

**Q: Temperature 0 — deterministic?**
A: Mostly greedy but not guaranteed deterministic: floating-point non-associativity across batch layouts, hardware, and provider-side changes produce variance. Never build correctness on "temp 0 = same answer"; build validation instead.

**Q: Top-k vs top-p — what's the real difference and why is top-p preferred?**
A: Top-k keeps a fixed *count* of the highest-probability tokens; top-p keeps a fixed *probability mass* (smallest set with cumulative prob ≥ p), so its count varies with the model's confidence — few tokens when it's sure, many when it's torn. Top-k drags in junk when the model is confident and cuts valid tokens when it's uncertain. That adaptivity is why top-p won.

**Q: You need a step that emits valid JSON tool args every time. Sampling settings?**
A: Temperature 0 (or the provider's floor), top-p left at default. Cold = deterministic, so no creative drift in the structure; tuning both temperature and top-p is pointless and compounds randomness. Pair it with schema validation and a repair loop — never rely on sampling alone for correctness (temp 0 isn't guaranteed identical).

## Structured outputs + tools

**Q: Design a robust tool-calling contract.**
A: Tight JSON schema (enums over free strings, required fields minimal), idempotent tools where possible (or idempotency keys), argument validation before execution with machine-readable errors returned INTO the loop so the model can repair, per-tool permission scope, and logged failures. Treat the model as an untrusted caller of your API.

## RAG

**Q: How do you pick a chunking strategy?**
A: Match retrieval unit to answer unit. Fixed-size with overlap = baseline; structure-aware (headings, paragraphs) beats it on docs; semantic chunking costs more and helps when structure is absent. Too small → context fragments lose meaning; too big → recall drops and noise dilutes. Decide by retrieval evals on your own corpus, not by blog defaults.

**Q: Why hybrid search instead of pure vector?**
A: Embeddings miss exact identifiers — SKUs, error codes, names, versions — where BM25 is perfect; BM25 misses paraphrase where embeddings shine. Union both, rerank (cross-encoder) for precision at the top. Most production retrieval failures are "the exact-match case" — hybrid is the fix.

## Evals

**Q: How do you trust an LLM-as-judge?**
A: Calibrate: label a sample by hand, measure judge–human agreement; if it's low, fix the rubric (binary/few-level criteria beat 1–10 scores). Judge prompts are versioned code. Watch for bias (position, length, self-preference — use a different model family than the one judged). Re-calibrate periodically; judges drift too.

**Q: A model upgrade improves your main metric but users complain. What happened?**
A: The eval set no longer represents real traffic — silent eval regression in coverage. Golden sets go stale as usage shifts. Fix: sample recent production traffic into evals continuously, segment metrics by intent/feature, add the complained-about cases as regression tests.

## Agents

**Q: When is an agent the wrong tool?**
A: Known workflow → fixed pipeline: cheaper, deterministic, testable. Agents pay off only when the path depends on intermediate results. The strongest production answer: workflows with narrow LLM decision points, agents only at genuine branch points — and hard budgets around every loop.

## Decision framework

**Q: Fine-tuning vs RAG vs ICL vs distillation — how do you choose?**
A: By what's missing. Knowledge missing/changing → RAG (updatable, attributable). Behavior/format/style missing → fine-tune (but you now own evals, retraining, drift). Few examples suffice → ICL/few-shot (try first, always). Cost/latency too high on a working system → distill to a smaller model. Classic wrong tool: fine-tuning to inject facts — it's lossy, unattributable, stale on day one.

## Inference stack

**Q: Explain prefill vs decode and one optimization for each.**
A: Prefill: parallel over prompt tokens, compute-bound → optimize with prompt caching (reuse KV across shared prefixes — system prompts, few-shots). Decode: serial, bandwidth-bound → continuous batching (new requests join mid-flight batches), speculative decoding (draft model proposes, target verifies in parallel). Metrics: TTFT for prefill, tokens/sec (TPOT) for decode.

**Q: What is paged attention?**
A: KV cache managed like OS virtual memory — fixed-size blocks, non-contiguous, indirection table per sequence. Kills fragmentation from variable-length sequences, so more concurrent requests fit → vLLM's core throughput win.

**Q: When does quantization hurt?**
A: Weight-only INT8 ≈ free; INT4 (AWQ/GPTQ) usually fine on chat but degrades long-chain reasoning, math, and low-resource languages first; FP8 needs hardware support and calibration. Rule: quantize, then run YOUR evals — published benchmarks don't cover your distribution. Aggressive KV-cache quantization degrades long-context recall specifically.

## Observability + safety

**Q: How do you defend against prompt injection in a RAG/agent system?**
A: Assume retrieved/tool content is adversarial. Layers: privilege separation (model's tools scoped to the user's own permissions — the injection can't exceed them), instruction/data separation in prompts, output filtering, human approval gates on irreversible actions, adversarial test suite in CI. No single defense works; the permission boundary is the one that actually holds.

**Q: Multi-tenant LLM app — what leaks and how do you prevent it?**
A: Vectors: shared retrieval index without tenant filters (worst), prompt/semantic caches keyed without tenant ID, conversation memory bleeding across users, fine-tuned models memorizing tenant data. Prevention: tenant ID mandatory in every retrieval filter and cache key, per-tenant encryption where required, contamination tests in CI (query tenant A for tenant B's known-unique strings).
