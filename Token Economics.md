# Token Economics

Stage 1 — Foundations · P0. What you pay and how long you wait — both measured in tokens, split by direction. The backbone of every cost/latency design answer.

Part of [[Applied AI]]. Prev: [[Sampling Parameters]]. Next: [[Context Engineering]].

## The one line to carry

**Input = parallel, compute-bound, cheap. Output = serial, bandwidth-bound, expensive.** Every cost/latency answer in Stage 1 collapses to this.

## The pricing model

- Billed **per token, split input vs output**, priced per million (MTok). Typical shape: input ~$3/MTok, output ~$15/MTok. **Output is 3–5× input, always.**
- You pay for **every input token on every call**. Resend a 10k-token history for 20 turns and you paid for that history 20 times.
- Output is billed for what's generated — longer answers cost linearly more money *and* time.

## Why output costs more — the mechanism

Two phases of a call:

- **Prefill (input):** all prompt tokens processed **in parallel**, one forward pass. Compute-bound, fast, cheap per token.
- **Decode (output):** tokens generated **one at a time**, autoregressive, each its own forward pass, holding the KV cache the whole time. Serial GPU time, memory-bandwidth-bound → expensive.

Input amortizes across a single parallel pass; output burns real serial time. That asymmetry *is* the price gap.

## Latency — what actually feels slow

- **TTFT (time to first token):** dominated by **prefill** → grows with *input* size. Huge prompt = slow start.
- **TPOT / tokens-per-sec:** dominated by **decode** → grows with *output* length.
- **Key insight:** latency tracks **output length far more than input length**. 50k input / 20 output feels fast; 500 input / 2000 output feels slow.

## The quadratic chat problem

Every turn resends the whole transcript, so turn 20 re-pays for turns 1–19. Input cost *per turn* rises as the conversation grows → total cost is the sum of a growing series → **quadratic, not linear**. Fix: summarize or window old turns so resent input stays bounded.

## Levers (interview-ready)

- **Cut output tokens** — biggest win: output is the expensive side of *both* ledgers (3–5× price and the serial-decode latency bottleneck). Cap `max_tokens`, ask for concise/structured output.
- **Prompt caching** — reuse KV cache across shared prefixes (system prompt, few-shots); cached input often ~10% of full price. (Stage 8 detail — know it exists.)
- **Right-size the model** — cheap model for easy calls, expensive only where needed (routing).
- **Trim resent history** — summarize/window instead of resending raw turns.

## Interview traps

- **Same total tokens, 5000-in/100-out vs 100-in/5000-out:** the output-heavy one costs more *and* is slower — for the same reason (output is serial + expensive, input is parallel + cheap).
- **Cutting input** helps cost and TTFT but the payoff is small — input is cheap and parallel. Output is where both meters spin fastest.
