---
title: "Mitigation — common to all companies"
description: "Mitigating the watermark is essentially the same across the ecosystem: destroy the correlation between the text and the sampler's key."
date: "2026-08-14"
tags: ["cryptography", "probability", "privacy", "technology"]
---

# Mitigation (common)

> *A statistical signature is not erased: it is undone. There is no eraser that removes it; there are processes that break the relationship it is made of. And that relationship is always the same — the correlation between the text and the key that wrote it.*

This document is the **general mitigation framework**, valid for every company in the dossier. The reason is structural: although each provider uses a different method (SynthID-Text, MirrorMark, literature schemes), all text watermarks live in the sampler and **are destroyed the same way**. Company specifics live in `company/mitigation.md`.

## 1. General principle

Every text watermark relies on a **correlation** between the observed text and a reproducible secret pattern. Detection measures that correlation and compares it against a threshold:

> **Reduce or remove the signature = destroy the correlation between the final text and the sampler's key.**

The evader does not need to *know* the key; transformations that **disconnect** the text from it are enough. The cost of each transformation is **semantic fidelity**. The real problem is optimizing between two forces: *detectability* and *fidelity*.

## 2. Attack taxonomy

### 2.1. Rewriting with another model (the standard attack)

Pass the text through another LLM — one without a mark, or from another provider — with a paraphrase instruction:

```
Input:        [watermarked text]
Instruction:  "Rewrite the text in your own words, keeping the meaning
               but changing structure, vocabulary, and sentence order."
Output:       [text with no correlation to the original key]
```

The second model re-samples every token from its own distribution, **without** the first model's key. The original mark cannot survive by definition; what remains is the second model's own trace (which may or may not be marked, depending on the provider).

**Measured effectiveness**: against MCmark the TPR drops to 11–48 %; against MirrorMark heavy paraphrasing reduces TPR to ≈ 57.8 %; radical rewriting brings it close to random.

### 2.2. Round-trip translation

$$
T \xrightarrow{\text{trans}} T_{\text{other}} \xrightarrow{\text{trans}^{-1}} T'
$$

Each translation step re-samples the text completely. The original mark disappears; the cost is **semantic drift** (loss of nuance and register).

### 2.3. Controlled re-sampling

If access to an equivalent model **without** the mark is available — an earlier version, another provider, open weights — regenerate the text with the same thematic seed. This is the cleanest attack: it comes from the same distribution, but from an independent sampling, with no correlation to the marked version's key.

### 2.4. Mechanical editing (insertion, deletion, substitution)

Changing synonyms, reordering clauses, deleting paragraphs. Its effectiveness depends on the scheme's **localization**:

- rigid token→position assignment: breaks with any insertion or deletion;
- contextual assignment (CABS): *localizes* the effect; an edit corrupts only the affected positions.

The sophisticated evader **edits selectively the highest-signal regions** (score-based attack, section 2.6).

### 2.5. Blending and splicing

Combine the marked text with human material or text from another model: the detector aggregates over the whole text, and if the marked fraction is low, the statistic falls below the threshold.

### 2.6. Adversarial optimization attack

If the detector is public (the European AI Act *requires* it to be), treat it as constrained optimization:

$$
\min_{T'} \; \bigl| \text{score}(T') - \mu_0 \bigr| \quad \text{subject to} \quad \text{sim}(T', T) \geq \tau
$$

with $\mu_0$ the mean of the statistic under unmarked text and $\text{sim}$ a semantic-similarity measure. The public detector becomes an **oracle to train the attack** — the most serious threat to any scheme.

### 2.7. Deterministic metadata stripping (for C2PA/EXIF signals)

*Metadata* signals (C2PA, EXIF, XMP, document properties) are not statistical: they are **bytes**. They are removed exactly with format conversion, re-saving, screenshots, or dedicated tools (e.g., the open-source `watermarks-remover` project, covering PNG, JPEG, SVG, PDF, DOCX, ODT, HTML and Markdown). There is no ambiguity: the bytes are there or they are not.

> **Note**: metadata is not "degraded", it is erased. A text watermark, by contrast, can only be *degraded*; it is never located or erased like a byte.

### 2.8. The structural route: not generating marked text

The simplest mitigation is not to produce marked text at all:

- models released before August 2, 2026 (transition period);
- **open-weight models run locally** (DeepSeek, Kimi K3, Mistral, etc.) — no mark by design;
- using the generated text only as a draft and rewriting it substantially by hand.

## 3. Comparative evaluation

| Attack | Effort | Fidelity | Effectiveness vs. MirrorMark-type marks | Collateral cost |
|--------|--------|----------|----------------------------------------|-----------------|
| Paraphrase with another model | Low | Medium | High (TPR ≈ 57.8 % → random) | Second model's trace |
| Round-trip translation | Low | Medium-low | High | Semantic drift |
| Re-sampling (unmarked model) | Medium | High | Very high | Requires access |
| Selective mechanical editing | Medium-high | Medium-high | Medium | Laborious |
| Blending / splicing | Medium | High | Medium-high | — |
| Oracle attack | High | High | Potentially total | Requires detector |
| Metadata stripping | Low | High (does not touch text) | Total against C2PA/EXIF | N/A for text marks |
| Local open weights | Low | High | Total (no mark) | N/A |

## 4. The two asymmetries in the evader's favor

1. **Key-access asymmetry.** The detector needs the key or an oracle; the evader does not. Destroying correlation is enough.
2. **Objective asymmetry.** The detector must decide with **low false positives** (not to condemn humans); the evader only needs to push the text **below the threshold**, which is statistically much cheaper.

This is why **no current text watermark is invulnerable to paraphrasing with a quality LLM**. The question is not *whether* the signature can be removed, but *at what fidelity cost*.

## 5. What does NOT work

- **Looking for invisible Unicode characters** (zero-width spaces, etc.): the statistical mark does not use hidden characters. Finding them does not prove a mark; not finding them does not prove the absence of one.
- **"AI writing" detectors** (linguistic-pattern analyzers): they lack the provider's key and are predictions, not verifications.
- **Removing "one marked paragraph"**: the signal is spread across the whole text; there is no byte to locate.

## 6. How marks are hardened (and why it is not solved)

Modern schemes respond with:

1. **multi-bit and contextual assignment** (CABS): localize the effect of edits;
2. **cascade marking**: two models, two marks, forcing two rewrites;
3. **adaptive thresholds**: the detector demands more evidence when tampering is suspected.

None of them defeats the fundamental attack — **rewriting with an independent sampler**. Hardening raises the price; it does not close the door.

## 7. Operational summary

- **Reduce** (without rewriting): edit the highest-signal regions, reorder paragraphs, substitute vocabulary, fragment and blend with your own text, shorten passages.
- **Remove** (accepting a rewrite): paraphrase with another model, round-trip translation, regenerate with an unmarked model.
- **Do not carry** a mark: local open weights, models from before August 2026, or substantial manual rewriting.
- **Metadata**: deterministic stripping (C2PA/EXIF) — as long as the signal is metadata and not text.

## 8. Closing

The text watermark is an honest solution to a real problem — attributing synthetic content — but it is subject to a physical constraint: **the signal lives in the randomness of sampling, and randomness is the easiest thing to destroy**. Rewriting is the blade no watermark has found a shield against, because rewriting is not an attack: it is the most human way to use language.

> **See also**: [worked example](mitigation-example.md) — a text before and after each mitigation phase. · [local models](local-models.md) — open-weight families with their own sampler, the route of never generating a mark. · [agents and prompts](agents-and-prompts.md) — why the prompt does not prevent the mark and what does mitigate it. · [the sampler](sampler.md) — the component where the signature lives.

> **Note**: this document is technical analysis (the literature calls it *watermark robustness/attacks*) and does not recommend breaking the law. In the EU the marking is the provider's obligation, not the user's.

## References

- [Anthropic — 05 Mitigation](anthropic/05-mitigation.md) — Claude-centered version
- Kirchenbauer et al. — [arXiv:2301.10226](https://arxiv.org/abs/2301.10226)
- Aaronson & Kirchner — [Watermarking GPT outputs](https://www.scottaaronson.com/blog/?p=8012)
- MCmark — [arXiv:2502.11268](https://arxiv.org/pdf/2502.11268)
- MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
- `watermarks-remover` (Guillaume Meyer) — [GitHub](https://github.com/guillaumemeyer/watermarks-remover)
