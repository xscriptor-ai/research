---
title: "Mitigation — how to reduce or remove the signature"
description: "Technical analysis of the routes to degrade or destroy Claude's text watermark."
date: "2026-08-14"
tags: ["cryptography", "probability", "privacy", "technology"]
---

# 05 — Mitigation

> *A statistical signature is not erased: it is undone. There is no eraser that removes it; there are processes that break the relationship it is made of. And that relationship is always the same — the correlation between the text and the key that wrote it.*

## 1. General principle

Every text watermark relies on a **correlation** between the observed text and a reproducible secret pattern. Detection measures that correlation and compares it against a threshold. Therefore:

> **Reduce or remove the signature = destroy the correlation between the final text and the sampler's key.**

The evader does not need to *know* the key; transformations that **disconnect** the text from it are enough. The cost of each transformation is **semantic fidelity**: how much meaning and form change. The real problem is optimizing between two forces: *detectability* and *fidelity*.

## 2. Attack taxonomy

### 2.1. Rewriting with another model (the standard attack)

The most direct route: pass the text through another LLM —unmarked, or from another provider— with a paraphrase instruction.

```
Input:        [text marked by Claude]
Instruction:  "Rewrite the following text in your own words,
               keeping the meaning but changing structure,
               vocabulary, and sentence order."
Output:       [text with no correlation to Claude's key]
```

The second model re-samples every token choice from its own distribution, **without** Claude's key. Claude's mark cannot survive by definition; what remains is the *second* model's own trace (which may or may not be marked, depending on the provider).

**Effectiveness**: very high against Kirchenbauer and MCmark (MCmark drops to TPR 11–48 %). Against MirrorMark, heavy paraphrasing reduces TPR to ≈ 57.8 %: better, but not invulnerable — and a more aggressive paraphrase (radical rewriting, not stylistic) brings the detection rate close to random.

### 2.2. Round-trip translation

Translate the text to another language and back:

$$
T \xrightarrow{\text{trans}} T_{\text{other}} \xrightarrow{\text{trans}^{-1}} T'
$$

Each translation step re-samples the text completely. The original mark disappears. The cost is **semantic drift**: round-trip translations lose nuance, alter register, and can introduce precision errors. It is the preferred attack when reasonable fidelity is wanted with little technical effort.

### 2.3. Controlled re-sampling (for those with access to the model)

If access to Claude **without** marking is available —e.g., an earlier version, or the same model via a provider that does not yet apply the mark— the text can be *regenerated* with the same thematic seed. Technically it is the cleanest attack: the text comes from the same distribution, but from **independent sampling**, with no correlation to the marked version's key.

### 2.4. Mechanical editing (insertion, deletion, substitution)

Light manual editing —changing synonyms, reordering clauses, deleting paragraphs— degrades the signal without destroying the text. Its effectiveness depends on the scheme's **localization**:

- rigid token→position assignment schemes break with any insertion or deletion (global alignment effect);
- contextual schemes like **CABS** *localize* the effect: an edit corrupts only the affected positions, and the detector can still aggregate enough signal from the rest.

The evader can amplify the attack by **selectively editing the highest-signal regions**, if able to estimate them (score-based attack, section 2.6).

### 2.5. Blending and splicing

Combine the marked text with human material or text from another AI: *"hide the marked needle in a haystack of unmarked text"*. The detector aggregates over the whole text; if the marked fraction is low, the aggregated statistic falls below the threshold. It is the text version of the *cocktail* attack.

### 2.6. Adversarial optimization attack

The sophisticated evader treats the problem as constrained optimization:

$$
\min_{T'} \; \bigl| \text{score}(T') - \mu_0 \bigr| \quad \text{subject to} \quad \text{sim}(T', T) \geq \tau
$$

where $\mu_0$ is the mean of the statistic under unmarked text and $\text{sim}$ a semantic-similarity measure (for example, *cosine similarity* over embeddings). If the detector is public (the European Code of Practice *requires* it to be), the evader can **access the oracle**, measure each fragment's signal, and apply discrete gradients or local search to minimize it. This is the most serious threat to any scheme: **a public detector is, by construction, an oracle for training the attack**.

### 2.7. The route that avoids the problem: not generating marked text

The simplest mitigation is not to *produce* marked text:

- use **models released before August 2, 2026** during the transition period (until December 2, 2026);
- use **open-weight models** locally or from other unmarked providers;
- use the text only as a **human starting point**: write, edit and restructure by hand until the proportion of generated text is negligible.

The last route is not an attack: it is the legitimate use Anthropic itself recognizes (the mark indicates that the text *was processed by Claude*, not that the idea is Claude's). The line between "editing Claude content" and "evading the mark" is blurry, and European law does not resolve it: it marks the *processor*, not the *source of the idea*.

## 3. Effectiveness evaluation

| Attack | Effort | Semantic fidelity | Estimated effectiveness vs. MirrorMark | Collateral signal cost |
|--------|--------|-------------------|---------------------------------------|------------------------|
| Paraphrase with another model | Low | Medium | High (≈ TPR 57.8 % in heavy paraphrase; can reach random with radical rewriting) | Leaves the 2nd model's trace |
| Round-trip translation | Low | Medium-low (drift) | High | Loss of nuance |
| Re-sampling (unmarked model) | Medium | High | Very high (no correlation) | Requires model access |
| Selective mechanical editing | Medium-high | Medium-high | Medium (CABS localizes; must attack signal regions) | Laborious |
| Blending / splicing | Medium | High | Medium-high (if marked fraction is low) | — |
| Oracle attack (optimization) | High | High | Potentially total (if detector is public) | Requires the detector |
| Pre-August-2026 / open models | Low | High | Total (no mark) | Not applicable |

## 4. The two asymmetries in the evader's favor

1. **Key-access asymmetry.** The detector needs the secret key or an oracle; the evader does not. Destroying correlation is enough, and for that no correlation target needs to be known.
2. **Objective asymmetry.** The detector must decide with **low false positives** (not to condemn humans); the evader only needs to push the text **below the threshold**, a statistically much cheaper goal. The system defends a territory; the attacker only needs one door.

The second asymmetry is why **no current text watermark is invulnerable to paraphrasing with a quality LLM**. The question is not *whether* the signature can be removed, but *at what fidelity cost*. The literature (and Anthropic itself) admits it: heavy paraphrasing, translation and rewriting are recognized routes of signal degradation.

## 5. How marks are hardened (and why it is not solved)

Modern schemes respond to the attack with three hardening measures:

1. **multi-bit and contextual assignment** (CABS): localize the effect of edits;
2. **cascade marking**: two models, two independent marks, forcing two rewrites;
3. **adaptive thresholds**: the detector demands more evidence based on tampering suspicion.

None of the three defeats the fundamental attack: **rewriting with an independent sampler**. Hardening raises the price (more effort, more semantic drift), it does not close the door. That is why the real regulatory debate is not about *the technique* but about *the social threshold*: what fraction of AI text is tolerated undetected, and who pays the false positives.

## 6. Operational synthesis

To **reduce** the signature (if the goal is to degrade detectability without rewriting):

- selectively edit the highest-signal regions, reorder paragraphs, substitute vocabulary;
- fragment and blend with your own or human text;
- shorten: short passages accumulate less signal.

To **remove** the signature (if a rewrite is accepted):

- paraphrase with another model or with the same model via an unmarked provider;
- round-trip translation;
- regenerate from scratch with an earlier, open, or other-provider model.

To **not carry** the signature at all:

- use models from before August 2, 2026 (while the transition lasts) or open weights;
- use Claude's text as a draft and rewrite it substantially by hand.

## 7. Closing

Claude's text watermark is an honest solution to a real problem —attributing synthetic content— but it is subject to a physical constraint from birth: **the signal lives in the randomness of sampling, and randomness is the easiest thing to destroy**. Rewriting is the blade no text watermark has found a shield against, because rewriting is not an attack: it is the most human way to use language. The system protects the attribution of those who copy and paste; it protects nothing from those who write. And that, perhaps, is the distinction European regulation has not yet wanted to see.

## References

- Kirchenbauer et al. — [arXiv:2301.10226](https://arxiv.org/abs/2301.10226)
- Aaronson & Kirchner — [Watermarking GPT outputs](https://www.scottaaronson.com/blog/?p=8012)
- MCmark — [arXiv:2502.11268](https://arxiv.org/pdf/2502.11268)
- MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
- Nature, *Can Anthropic's invisible watermarks curb 'AI slop'?* — [link](https://www.nature.com/articles/d41586-026-02503-7)
- *'Humanizer' tool can erase signs of AI-written text* — [Nature](https://www.nature.com/articles/d41586-026-02105-3)
