# Sampling Parameters

Stage 1 — Foundations · P0. How the model turns its output distribution into an actual next token — the knobs you set on every call.

Part of [[Applied AI]]. Prev: [[Tokens and Context Windows]]. Next: [[Token Economics]].

## The setup

The model doesn't output a token — it outputs a score for **every token in the vocabulary**. After the forward pass: raw scores (**logits**) → softmax → a probability distribution summing to 1. Sampling parameters decide how a single token is drawn from that distribution.

Example distribution for the next token: `"cat"` 0.60, `"dog"` 0.25, `"bird"` 0.10, then a long tail.

## Temperature

Scales the logits **before** softmax → controls how peaked vs flat the distribution is. Memory hook: **heat = chaos**.

- **Near 0 (cold):** distribution sharpens onto the top token → greedy, stable, repetitive. For extraction, classification, code, tool/JSON calls.
- **1.0:** raw distribution, untouched.
- **> 1 (hot):** distribution flattens, long-tail tokens gain real probability → creative, diverse, more errors and hallucination. For brainstorming, varied copy.
- **Trap:** temperature 0 is *not guaranteed* deterministic — floating-point non-associativity across batch layouts/hardware, and provider-side changes, produce variance. Never build correctness on "temp 0 = identical output"; build validation.

## Top-p (nucleus sampling)

Truncate the tail **before** sampling: keep the smallest set of tokens whose **cumulative probability ≥ p**, discard the rest, renormalize, then sample.

- `top_p = 0.9` → sample only from the tokens that together cover 90% of the mass.
- **Adaptive by count.** Confident step (one token ≈ 0.95) keeps ~1; uncertain step (mass spread thin) keeps many. The *count* flexes with the model's confidence.

## Top-k

Keep the **k highest-probability** tokens, drop the rest. Fixed count, not adaptive — keeps exactly k whether the model is certain or torn, so it drags in junk when confident and cuts valid tokens when uncertain. Largely superseded by top-p.

**Top-k vs top-p:** top-k = fixed *count*; top-p = fixed *probability mass*, variable count. That variable count is the whole reason top-p is preferred.

## What to actually set

- Providers apply roughly top-k → top-p → temperature.
- **Tune one, not both.** Cranking temperature *and* top-p high compounds randomness into incoherence.
- Deterministic task (JSON args, classification) → **temperature 0**, leave top-p default.
- Creative task → raise temperature *or* top-p, not both.

## Interview traps

- **Direction:** cold = deterministic, hot = creative. Temp 1 is the raw distribution (already random), not "safe"; > 1 adds more chaos.
- **"Make tool calls reliable"** → temperature 0. You want the same valid structure every time.
- **Top-p adapts, top-k doesn't** — name the mechanism (cumulative mass vs fixed count), not just the formula.
