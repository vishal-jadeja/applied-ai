# Tokens and Context Windows

Stage 1 — Foundations · P0. How an LLM works *operationally* — the mechanics you defend in a design round, not the math.

Part of [[Applied AI]]. Next: [[Sampling Parameters]].

## Tokens

The model never sees text. It sees **tokens** — integer IDs drawn from a fixed vocabulary (~100k–200k entries). A tokenizer splits text into subword chunks before anything reaches the model.

- Common word → 1 token. Rare/long word → several. `"tokenization"` → `["token", "ization"]`.
- Rough English rule: **1 token ≈ 4 characters ≈ 0.75 words**. 1000 words ≈ 1300 tokens.
- Code, JSON, and non-English tokenize worse — more tokens per unit of meaning. JSON is heavy because structural characters (`{`, `"`, `:`, `,`) and repeated keys each cost tokens on every row.
- Generation is **autoregressive**: the model emits one token at a time, each conditioned on all tokens before it. This is why output is slow and input is comparatively fast (see inference stage).

Everything downstream — cost, latency, context limits — is measured in tokens, never words.

## Context window

A hard cap on the number of tokens the model can attend to in a single call. **Input and output share the same budget.**

- Window = e.g. 200k tokens. If the prompt uses 195k, only 5k remain for the answer → the response truncates, or the call is rejected if input alone exceeds the limit. Big input **starves** the output.
- Not memory. The model is **stateless between calls**. "Conversation history" is the harness resending every prior turn on each request — the model itself remembers nothing.
- Fill the window and your options are truncate, summarize, or retrieve. That last one is why RAG exists (Stage 3).
- Attention cost scales roughly **quadratically** with window length → longer context is slower and pricier, and quality can *degrade*: **lost in the middle** — the model attends worst to tokens buried in the center of a long context.

## Interview traps

- **"How does the model remember earlier messages?"** → It doesn't. The harness resends the transcript every call.
- **"Prompt is 195k / window is 200k — what breaks?"** → Only 5k left for output; the answer truncates. Input and output share one budget.
- **Lost-in-the-middle is a *quality* failure, not a *capacity* failure.** Don't conflate the two.

## Why it matters

Context engineering ([[Context Engineering]]) is entirely about spending this token budget well — what earns a slot in the window and what gets cut. Can't reason about that without tokens and the shared budget locked in first.
