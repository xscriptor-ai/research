---
title: "The statistical pattern — what is disclosed and what is probable"
description: "What Anthropic has said about the algorithm, and what public evidence suggests: MirrorMark, MCmark and the distortion-free family."
date: "2026-08-14"
tags: ["cryptography", "probability", "technology", "research"]
---

# 03 — The statistical pattern

> *A secret is not only what is not said: it is also what is shown without meaning to. Anthropic has not revealed the algorithm, but it has described its consequences so precisely that the algorithm gives itself away.*

## 1. What Anthropic has disclosed

Anthropic **has not published the specific algorithm** nor the exact nature of the statistical pattern. What it has stated, consistently, is a set of properties that strongly bound the technical family:

1. the mark is embedded **during text generation** (at the model level);
2. it is **imperceptible** to the naked eye;
3. it **does not change the distribution** or quality: "does not change the meaning, quality, or readability";
4. it is **multi-surface**: present in the app, API, Claude Code, Cowork, Claude Tag;
5. it **survives editing** (copy-paste, insertions, deletions, partial substitutions);
6. it will be **detectable by third parties** (the Code of Practice requires it), with technical documentation pending publication.

Additionally, Anthropic's *Transparency* page, updated on July 23, 2026, says they have *"worked with industry and academia"* on marking techniques. Before that date, the same page stated that they did **not** provide text marking.

## 2. What these properties rule out

Properties 2 and 3 are the most informative:

- **Not altering the distribution** rules out Kirchenbauer-type marks (logit bias $\delta$): any green-list bias is a measurable distortion, adversarially detectable and perceptible as quality loss. Anthropic's claim is incompatible with that design.
- Properties 3 and 6 combined point to the **distortion-free / unbiased** family: marks that live in the sampling randomness ($u$ values or *ranks*) and preserve the marginal distribution by construction.
- Property 5 —survival of edits— additionally requires that the pattern not depend on a rigid token-by-token alignment, but on a **contextual assignment** that localizes the effect of edits.

## 3. The candidates in the literature

### 3.1. Kirchenbauer et al. (2023) — green/red list

The foundational reference, but **distorting**. It does not fit Anthropic's quality claim, although its conceptual framework (pseudorandom key seeded by context, $z$-test) underlies everything that followed. It is mentioned here as a theoretical starting point, not as a candidate.

### 3.2. Aaronson & Kirchner (2023) — mark in the randomness

Watermarking based on the Gumbel trick with a secret key and a hash of the previous context. Detection sums the reproduced $u$ values and compares them with the expectation under the null hypothesis:

$$
S = \sum_{t} u_{x_t} \quad \text{vs.} \quad \mathbb{E}[S] = \frac{n}{2}
$$

The deviation of the sum of the $u$ from the uniform mean $n/2$ is the signal. It is **distortion-free**, and it is the direct ancestor of modern techniques.

### 3.3. MCmark (2025) — unbiased watermarking

MCmark introduces an *unbiased* mark that embeds a hidden statistical signal **during token generation**, preserving the original distribution. Its defining feature: the mark can be detected **without access to the prompt or the model API**, and it is designed to remain detectable after some text modification.

**Relevant weakness**: under paraphrasing its true-positive rate (TPR) drops to **11 %** with 1 % false positives (FPR); under GPT rewriting, to 48 %. It is robust against mechanical edits, but fragile against semantic paraphrase.

### 3.4. MirrorMark (2026) — the most likely candidate

**MirrorMark** (Ya Jiang, Massieh Kordi Boroujeny, Surender Suresh Kumar, Kai Zeng; George Mason University; arXiv:2601.22246, January 2026) is, according to Search Engine Journal's analysis, the candidate that **best fits the six disclosed properties**. The researchers commercialize it through **InvisibleID**, also at George Mason — a fact that fits Anthropic's claim of having worked "with industry and academia".

#### The mechanism in three steps

**Step 1 — mod-1 reflection (the "mirror").** Each $b$-bit symbol is encoded through a reflection transformation on the sampling value $u \in [0,1)$, around a message-specific pivot:

$$
u' = (2\tau - u) \bmod 1
$$

The reflection **preserves the uniformity** of $u$ (it is a bijection of $[0,1)$ onto itself), so the distribution of the chosen token does not change: by design, *distortion-free*. The detector, knowing the key, reproduces $u$ and determines which of the two possible values (the original or its reflection) was observed; that encodes the bit.

**Step 2 — Context-Anchored Balanced Scheduler (CABS).** CABS decides **which symbol is embedded at each step**, mapping tokens to message positions in a **context-dependent and balanced** way. This achieves two things: it avoids empty or highly imbalanced allocations, and it **localizes the effect of edits** (an insertion or deletion only corrupts the affected positions, not the whole message).

**Step 3 — Decoding.** The detector *replays* the process: applies CABS to recover the token→position assignment, decodes each symbol from the reflected $u$ with the appropriate *score* function, aggregates the values over all tokens and compares against a threshold:

$$
\text{declared marked} \iff \sum_{\text{tokens}} \text{score}(\cdot) \geq \theta
$$

#### Declared results

- with **54 bits** embedded in **300 tokens**, it improves bit accuracy by 8–12 % and correctly identifies up to 11 % more marked texts at 1 % FPR;
- **robust to insertions, deletions and substitutions**;
- under **heavy paraphrasing**, TPR drops to ≈ **57.8 %** (at 1 % FPR), significantly better than MCmark, but not invulnerable.

## 4. Candidate comparison

| Method | Type | Distortion | Edit resilience | Paraphrase resilience | Fit with the 6 properties |
|--------|------|-----------|-----------------|-----------------------|---------------------------|
| Kirchenbauer (2023) | Logit bias | Yes | Medium | Low | Low |
| Aaronson–Kirchner (2023) | $u$-values | No | Low | Low | Medium |
| MCmark (2025) | Unbiased | No | Medium | Low (TPR 11–48 %) | High |
| **MirrorMark (2026)** | mod-1 reflection + CABS | No | High | Medium (TPR ≈ 57.8 %) | **Very high** |

## 5. Verdict

The honest answer to *"have they said anything about the statistical pattern they will use?"* is: **not directly, but yes indirectly**. Anthropic has disclosed a set of properties that only one technical family fulfills —the *multi-bit, distortion-free* marks embedded in the sampling randomness, with contextual assignment robust to edits— and that family has **MirrorMark** as its canonical, coinciding representative, coming from a commercialized academic group (InvisibleID), consistent with Anthropic's public statement.

It is a **reasonable inference, not a confirmation**. Anthropic itself has said it will publish detection documentation "in the coming weeks". Until then, any mitigation analysis must assume the strongest hypothesis (a MirrorMark-type multi-bit distortion-free mark), because it is the *worst* case for the evader: the one that best survives edits and is hardest to destroy without rewriting the text.

## References

- MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
- MCmark — [arXiv:2502.11268](https://arxiv.org/pdf/2502.11268)
- Kirchenbauer et al. — [arXiv:2301.10226](https://arxiv.org/abs/2301.10226)
- Aaronson & Kirchner — [Watermarking GPT outputs](https://www.scottaaronson.com/blog/?p=8012)
- Search Engine Journal, *Why Anthropic's Claude Watermark May Be A New Text-Marking Method* — [link](https://www.searchenginejournal.com/why-anthropics-claude-watermark-may-be-a-new-text-marking-method/585703/)
- Anthropic, *Voluntary commitments / Transparency* — [link](https://www.anthropic.com/transparency/voluntary-commitments)
