---
title: "Mitigation — common to all companies"
description: "Mitigating invisible marks realistically: statistical watermarks (degradation) and Unicode carriers (exact strip), what actually works, what it costs, and what cannot be certified."
date: "2026-09-17"
tags: ["cryptography", "probability", "privacy", "technology"]
---

# Mitigation (common)

> *A statistical signature is not erased: it is undone. There is no eraser that removes it; there are processes that break the relationship it is made of. And that relationship is always the same — the correlation between the text and the key that wrote it.*

This document is the **general mitigation framework**, valid for every company in the dossier. It covers the two families of marks that survive copy-paste — **statistical marks** (sampling randomness, keyed, provider-side) and **Unicode carriers** (bytes in the string, usually unkeyed, see [unicode.md](unicode.md)) — plus the file-level layers (metadata, perceptual media). Company specifics live in `company/mitigation.md`.

## 1. General principle

Every statistical text watermark relies on a **correlation** between the observed text and a reproducible secret pattern. Detection measures that correlation and compares it against a threshold:

> **Reduce or remove the signature = break the correlation between the final text and the sampler's key.**

The evader does not need to *know* the key; transformations that **disconnect** the text from it are enough. Two forces are in tension: *detectability* and *fidelity*. Unicode carriers are different: they are not statistical but byte-level, so their removal is exact — and so are the false positives it can cause.

## 2. What actually survives copy-paste

| Layer | Survives copied text? | Survives files? | Removal type |
|-------|----------------------|-----------------|--------------|
| Statistical (sampling) mark | Yes (plain text) | — | Degradation (re-sampling) |
| Unicode carriers | Yes (unless destination sanitizes) | Yes | Exact strip |
| Weight-based mark | Yes | — | Retraining/fine-tuning (impractical) |
| C2PA / EXIF / XMP / doc properties | No (not part of the text) | Yes | Exact strip |
| Perceptual media marks | — | Yes (to a degree) | Regeneration / destructive transforms |
| Visible labels | No | Sometimes | Cropping / leaving the platform |

## 3. Attack taxonomy

### 3.1. Rewriting with another model (the standard attack)

Pass the text through another LLM — one without a mark, or from another provider — with a paraphrase instruction:

```
Input:        [watermarked text]
Instruction:  "Rewrite the text in your own words, keeping the meaning
               but changing structure, vocabulary, and sentence order."
Output:       [text whose keyed correlation is broken or degraded]
```

The second model re-samples every token from its own distribution, **without** the first model's key. The original generation-time correlation is destroyed by construction; what remains is the *second* model's own trace (which may or may not be marked, depending on the provider).

**Measured effectiveness — realistic numbers**:

- against **MCmark**, detection TPR drops to **11–48 %** at 1 % FPR;
- against **MirrorMark**, heavy paraphrasing reduces *detection* TPR to **≈ 57.8 %** (best configuration, 36-bit/400-token setting) and collapses payload recovery to **near random** (bit accuracy ≈ 0.54);
- against **SynthID-Text**, the Nature paper documents that edits and paraphrasing weaken the mark — with the caveat that a strong paraphrase "usually does change the text significantly".

**The nuance that the dossier insists on**: *detection* and *payload* fall separately. A paraphrase can leave enough residual signal for a yes/no detector (57.8 % is still a majority) while destroying the embedded message. Two limits must be stated honestly: (i) the MirrorMark experiment used a weak paraphraser (a 2020 model), so the numbers are an order of magnitude, not a law — a modern rewriter may clear more, but it costs more fidelity; (ii) paraphrasing never *guarantees* falling below the threshold.

**Re-stamping hazard**: rewriting with the *same* provider's model replaces one mark with another. A clean pipeline uses a **non-origin** model for the rewrite.

### 3.2. Round-trip translation

$$
T \xrightarrow{\text{trans}} T_{\text{other}} \xrightarrow{\text{trans}^{-1}} T'
$$

Each translation step re-samples the text completely, and translation is one of the strongest known attacks. The cost is **semantic drift**: lost nuance, altered register, mangled named entities and terminology. Preferred when reasonable fidelity is wanted with little technical effort.

### 3.3. Controlled re-sampling

If access to an equivalent model **without** the mark is available — an earlier version, another provider, open weights — regenerate the text with the same thematic seed. The cleanest attack: it comes from the same distribution, from an independent sampling, with no correlation to the marked version's key. Requires access, and carries the same re-stamping caveat if the "unmarked" model is not actually unmarked.

### 3.4. Mechanical editing (insertion, deletion, substitution)

Changing synonyms, reordering clauses, deleting paragraphs. Effectiveness depends on the scheme's **localization**:

- position-assignment schemes (multi-bit schedulers): rigid token→position mappings break with insertions/deletions;
- contextual assignment (**CABS** in MirrorMark, sliding-window seeding in SynthID/Kirchenbauer): *localizes* the effect — only nearby evidence is corrupted, the detector aggregates the rest;
- against a CABS-type mark, isolated edits are weak; the evader must attack the highest-signal regions, which requires estimating the signal (i.e. an oracle, §3.6).

### 3.5. Blending and splicing

Combine the marked text with human material or text from another model: the detector aggregates over the whole text, and if the marked fraction is low, the statistic falls below the threshold. Costs coherence and provenance (the result is a collage).

### 3.6. Adversarial optimization attack

When the evader has access to the detector —public, leaked, or by being one of the organizations admitted to it—, treat it as constrained optimization:

$$
\min_{T'} \; \bigl| \text{score}(T') - \mu_0 \bigr| \quad \text{subject to} \quad \text{sim}(T', T) \geq \tau
$$

with $\mu_0$ the mean of the statistic under unmarked text and $\text{sim}$ a semantic-similarity measure. The European Code of Practice obliges providers to *facilitate* third-party detection, but that does not mean a public detector: Anthropic keeps its detection API in **private preview** for eligible organizations, and Google verifies through its own channel. Any accessible detector becomes an **oracle to train the attack** — the most serious threat to any scheme. For everyone else, the "oracle" in practice is a **research harness** (MarkLLM, same-key Gumbel replay) that only certifies *same-configuration* clearance, never a vendor detector.

### 3.7. Deterministic stripping: metadata (C2PA/EXIF/XMP)

*Metadata* signals are not statistical: they are **bytes**. They are removed exactly with format conversion, re-saving, screenshots, or dedicated tools (the open-source `watermarks-remover` covers PNG, JPEG, SVG, PDF, DOCX, XLSX, PPTX, ODT, HTML, Markdown and media containers). Three realistic caveats:

- **PDF is a container, not a field**: a real strip requires a structural rebuild (`qpdf` in the reference tooling), not just deleting a label;
- **some editors re-add** Content Credentials on save (a stripped image can be re-stamped by the next tool in the chain);
- metadata does not travel with **copied text**, so stripping is only relevant for files.

### 3.8. Deterministic stripping: Unicode carriers

Carriers (zero-width, joiners, bidi controls, tag characters, exotic spaces) are **bytes in the string**, so removal is exact — no statistics involved. The `watermarks-remover` tooling formalizes it as *Layer A*, separate from the statistical *Layer B*. The realistic rules:

- strip the payload-ish carriers (U+200B, U+2060, U+FEFF outside file start, U+E0000–U+E007F, deprecated format controls);
- normalize exotic spaces **only if** the typography allows it (French `« »` and `;:!?` need U+00A0/U+202F; multilingual text needs ZWJ/ZWNJ, variation selectors and bidi controls);
- never apply a blind "remove all `Cf`" filter: it breaks legitimate emoji, Indic, Persian, Arabic and CJK text.

Full inventory and detection snippets in [unicode.md](unicode.md).

### 3.9. The structural route: not generating marked text

The simplest mitigation is not to *produce* marked text:

- **open-weight models run locally** (DeepSeek, Llama, Qwen, Gemma, Mistral, Kimi, GLM, Phi…) with your own engine — no provider mark by design ([local models](local-models.md));
- models released before August 2, 2026 during the transition period (until December 2, 2026);
- using generated text only as a draft and rewriting it substantially by hand.

Two caveats: engines can ship watermarking layers (keyed-Gumbel marks already exist in self-hosted engines such as `arbi-serve`), and a mark **baked into weights** by fine-tuning would survive local deployment. Neither is documented in the open-weight families of this dossier; both are why the guarantee is "no *provider sampler* mark", not "no mark of any kind".

## 4. How effective is this, really

| Route | Effort | Fidelity cost | Reliability of evasion | Can it be verified? |
|-------|--------|---------------|------------------------|---------------------|
| Strong LLM paraphrase | Low | Medium | **Medium–high, uncertain**: detection TPR can stay ≈ 50–60 % while payload dies | No vendor certification |
| Iterative paraphrasing (same-key harness) | Medium | Medium–high | High *in same-config tests* (often clears on 2nd attempt) | Only against the harness's own detector |
| Round-trip translation | Low | Medium-high | High | No |
| Re-sampling with unmarked model | Medium | High | Very high | No |
| Mechanical editing | Medium-high | Medium-high | Low against CABS; high against Unicode | Layer A: exact |
| Blending / splicing | Medium | High | Medium-high (dilution) | No |
| Oracle attack | High | High | Potentially total | Only if detector access |
| Metadata strip | Low | None (files) | Exact for that layer | Yes (deterministic) |
| Unicode strip | Low | None–low (typography) | Exact for that layer | Yes (deterministic) |
| Local open weights | Low | None | Total for sampler marks | By construction |

**The three realities that any honest analysis must state**:

1. **No local tool can certify "not detected" by a vendor detector.** Vendor detectors are private; research harnesses are same-key only. A clean report means "this detector, this key, this configuration no longer fires" — nothing more.
2. **Detection is not payload.** A detector may still say "possibly AI" after the embedded message is unrecoverable, because the statistic degrades before it vanishes (MirrorMark: 57.8 % detection vs bit accuracy ≈ random).
3. **Short and low-entropy texts carry less signal to begin with.** Statistical marks accumulate evidence with length and with the entropy of the distribution (SynthID-Text's own analysis): a deterministic answer or a three-line snippet may be weak or undetectable even before any mitigation — which cuts both ways, since it also limits what an attacker can measure.

## 5. Comparative evaluation

| Attack | Effort | Fidelity | Effectiveness vs. MirrorMark-type marks | Collateral cost |
|--------|--------|----------|----------------------------------------|-----------------|
| Paraphrase with another model | Low | Medium | Medium-high (detection TPR ≈ 57.8 %; payload ≈ random) | Second model's trace; re-stamping if same provider |
| Round-trip translation | Low | Medium-low | High | Semantic drift |
| Re-sampling (unmarked model) | Medium | High | Very high | Requires access |
| Selective mechanical editing | Medium-high | Medium-high | Medium | Laborious; needs signal estimate |
| Blending / splicing | Medium | High | Medium-high | Coherence |
| Oracle attack | High | High | Potentially total | Requires detector access |
| Metadata stripping | Low | High (does not touch text) | Total against C2PA/EXIF | N/A for text marks |
| Unicode stripping | Low | High (with typographic care) | Total against carriers | Breaks legit joiners/spaces if blind |
| Local open weights | Low | High | Total (no sampler mark) | Marks in weights/engines are separate risks |

## 6. The two asymmetries in the evader's favor

1. **Key-access asymmetry.** The detector needs the key or an oracle; the evader does not. Destroying correlation is enough.
2. **Objective asymmetry.** The detector must decide with **low false positives** (not to condemn humans); the evader only needs to push the text **below the threshold**, which is statistically much cheaper.

This is why **no current statistical text watermark is invulnerable to paraphrasing with a quality LLM**. The question is not *whether* the signature can be removed, but *at what fidelity cost* — and whether someone can prove it happened.

## 7. What does NOT work

- **Assuming a mark is visible or byte-like.** Statistical marks leave no bytes; finding nothing in the hexdump proves nothing about them.
- **Assuming carriers prove a statistical mark.** A zero-width character is a carrier, not evidence of a keyed watermark — and legitimate text is full of them.
- **"AI writing" detectors** (linguistic-pattern analyzers): they lack the provider's key and are predictions, not verifications.
- **Removing "one marked paragraph"**: the signal is spread across the whole text; there is no byte to locate.
- **Blind Unicode filtering**: it removes the mark and also legitimate ZWJ, NBSP, bidi controls and combining marks.

## 8. How marks are hardened (and why it is not solved)

Modern schemes respond with:

1. **multi-bit and contextual assignment** (CABS): localize the effect of edits;
2. **spectral/diversity-aware sampling** (SynthID-Text): preserve quality while keeping detectability, including through speculative decoding;
3. **adaptive thresholds and selective prediction**: the detector demands more evidence or abstains when uncertain.

The concept of *cascade marking* (two models, two marks, forcing two rewrites) and multi-layer thresholds circulates as a proposed direction, but is not a documented deployed standard and raises its own false-positive costs. None of these measures defeats the fundamental attack — **rewriting with an independent sampler**. Hardening raises the price; it does not close the door.

## 9. Verification discipline

Before claiming a mitigation "worked":

1. state **which detector, which key, which configuration** was tested, and whether it is a vendor detector or a same-config research harness;
2. separate **detection** (binary) from **payload** (message recovery);
3. report the **false-positive baseline** of the detector (typically 1 % FPR in the literature);
4. state the **fidelity cost** with a same-size sample, not an anecdote;
5. assume the **origin model is not the rewrite model**, or the result is invalid by re-stamping.

## 10. Operational summary

- **Reduce** (without rewriting): edit the highest-signal regions (requires an estimate), reorder paragraphs, substitute vocabulary, fragment and blend with your own text, shorten passages.
- **Remove** (accepting a rewrite): paraphrase with another model, round-trip translation, regenerate with an unmarked model. Use a non-origin model.
- **Strip bytes** (exact, per layer): Unicode carriers with typographic care ([unicode.md](unicode.md)); metadata in files (C2PA/EXIF), noting the PDF container caveat.
- **Do not carry** a mark: local open weights, models from before August 2026, or substantial manual rewriting. Check what the engine itself adds.
- **Do not confuse layers**: a clean hexdump, a clean metadata check and a below-threshold statistic are three different claims about three different marks.

## 11. Closing

The text watermark is an honest solution to a real problem — attributing synthetic content — but it is subject to a physical constraint: **the signal lives in the randomness of sampling, and randomness is the easiest thing to destroy**. Rewriting is the blade no statistical watermark has found a shield against, because rewriting is not an attack: it is the most human way to use language. The Unicode layer adds the opposite lesson: when the signal is bytes, it can be removed exactly — and what pays the price is not cryptography but typography, in languages that needed those invisible characters to be written correctly.

> **See also**: [Unicode carriers](unicode.md) — the other invisible layer, byte by byte. · [worked example](mitigation-example.md) — a text before and after each mitigation phase. · [local models](local-models.md) — the structural route of never generating a sampler mark. · [agents and prompts](agents-and-prompts.md) — why the prompt does not prevent the mark and what does mitigate it. · [the sampler](sampler.md) — the component where the statistical signature lives. · [landscape by company/model](companies.md) — who uses which mark.

> **Note**: this document is technical analysis (the literature calls it *watermark robustness/attacks*) and does not recommend breaking the law. In the EU the marking obligation falls on the provider; users may have their own transparency duties depending on what they publish, and this dossier is not legal advice.

## References

- [Anthropic — 05 Mitigation](anthropic/05-mitigation.md) — Claude-centered version
- Kirchenbauer et al. — [arXiv:2301.10226](https://arxiv.org/abs/2301.10226)
- Aaronson & Kirchner — [Watermarking GPT outputs](https://www.scottaaronson.com/blog/?p=8012)
- MCmark — [arXiv:2502.11268](https://arxiv.org/pdf/2502.11268)
- MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
- Dathathri et al., *Scalable watermarking for identifying LLM outputs* — [Nature 634, 818–823 (2024)](https://www.nature.com/articles/s41586-024-08025-4)
- `watermarks-remover` (Guillaume Meyer) — Layer A/B, deterministic vs best-effort — [GitHub](https://github.com/guillaumemeyer/watermarks-remover)
- MarkLLM (same-config verification harness) — [GitHub](https://github.com/THU-BPM/MarkLLM)
- vLLM-Watermark (optional watermarking layer for vLLM) — [link](https://vermaapurv.com/vLLM-Watermark/index.html)
