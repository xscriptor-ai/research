---
title: "Index — Invisible AI traces in language models"
description: "Research dossier on AI-generated content watermarking, company by company: methods, detection, and mitigation."
date: "2026-09-17"
tags: ["technology", "cryptography", "privacy", "legislation", "research"]
---

# Invisible AI traces in language models

> *A text that can be attributed is a text that no longer fully belongs to whoever wrote it. The question is not whether the mark is visible, but whether authorship is still free.*

Research dossier on **AI-generated content marking** following the European AI Act (Art. 50) and Chinese regulation: what technique each company uses, what they have disclosed, and **how the signature is reduced or removed**. Verified August–September 2026.

## Index

| Section | Content |
|---------|---------|
| [**Mitigation (common)**](mitigation.md) | The framework for both families of copy-paste-surviving marks: statistical watermarks (degradation) and Unicode carriers (exact strip). Attack taxonomy, realistic effectiveness, fidelity costs, and verification discipline. |
| [**Unicode carriers**](unicode.md) | The other invisible layer: zero-width characters, bidi controls, tag characters, exotic spaces, homoglyphs — how they encode, how they are detected and removed without breaking legitimate text. |
| [**Landscape by company/model**](companies.md) | Exhaustive catalogue: every company and model family with its mark, format, copy-paste survival and visibility, including the Chinese labeling regime and open weights. |
| [**Worked example**](mitigation-example.md) | A text before and after mitigation: original, light editing, and rewriting with another sampler, with what happens in the invisible layer explained. |
| [**Local models without a trace**](local-models.md) | Open-weight families (DeepSeek, Llama, Qwen, Gemma, Mistral, Kimi, Phi, GLM) run locally, inference engines, and why there is no mark in that scenario — with the API and intermediary caveat. |
| [**Agents and prompts**](agents-and-prompts.md) | Why neither a prompt nor building an agent prevents the mark (it lives in the provider's sampler), and how it is mitigated by changing the sampler the agent accesses. |
| [**The sampler**](sampler.md) | What the sampler is and what it consists of: the component that turns the model's distribution into a concrete token — and why it is the home of the watermark. |
| [**Anthropic — Claude**](anthropic/README.md) | Full dossier: regulatory context, technique, statistical pattern (MirrorMark), detection and limitations, mitigation. |
| [**Google — Gemini / SynthID**](google/README.md) | Methods (`methods.md`) and specific mitigation (`mitigation.md`): mark on text, image, audio and video. |
| [**OpenAI — ChatGPT**](openai/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): 2024 research, C2PA/SynthID, no text deployment yet. |
| [**DeepSeek — V4**](deepseek/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): Chinese labeling, open weights without a mark. |
| [**Kimi — K3 (Moonshot)**](kimi/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): visible identifiers and metadata-based ones. |
| [**Others — Meta, Microsoft, Amazon, Adobe, Stability…**](others/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): media marks (Meta, Amazon, Adobe, Stability, ElevenLabs), Code signatories and peripheral cases. |

## Folder structure

Dedicated folders exist for the cases with a deployed or documented mark and the most evidence; the rest of the ecosystem lives in the [landscape catalogue](companies.md).

```
company/
  README.md       — summary card and internal links
  methods.md      — the company's own marking methods
  mitigation.md   — mitigation specifics (the common part lives in the root)
```

Cross-cutting documents (mitigation, Unicode carriers, catalogue, worked example, local models, agents, sampler) live at the root.

## Core idea

Two families of marks survive copying text and pasting it elsewhere, and they are not the same thing:

1. **Statistical marks** — woven into the randomness of token sampling by the provider's inference layer. They are keyed, invisible, and only *degradable*. For this family the rule is:

$$
\text{mark present} \iff \text{the sampler uses the key } K
$$

**Whoever controls the sampler decides**: providers mark in their inference layer; whoever runs open weights locally is their own sampler and carries no provider mark.

2. **Unicode carriers** — invisible characters written into the bytes of the string (zero-width, joiners, bidi controls, tag characters, exotic spaces). They are usually unkeyed, detectable without any secret, and *exactly removable* — with typographic care for the legitimate ones ([unicode.md](unicode.md)).

A single text can carry both at once, plus file-level metadata if it travels as a file. Mitigation must name the layer it is attacking.

## Landscape summary

The exhaustive company-by-company and model-by-model catalogue is in [**Landscape by company/model**](companies.md). Quick view:

| Company / model | Text mark (copy-paste) | Method | Visibility | Public verification |
|------------------|------------------------|--------|------------|----------------------|
| Google (SynthID-Text) | Yes (~100 % of new generations, per dossier sources) | Statistical in sampling (Tournament) | Invisible | Private, Google channel |
| Anthropic (Claude) | Yes (confirmed) | Likely MirrorMark | Invisible | Detection API in private preview |
| OpenAI (ChatGPT) | Not deployed in text | C2PA/SynthID (image/audio) | Invisible | Supported formats only |
| Meta / Microsoft / Amazon / Adobe / Stability | No text mark documented | Media marks: learned or C2PA | Invisible | C2PA tools / provider |
| DeepSeek V4 / Kimi K3 / Qwen / GLM / Ernie / Doubao / Hunyuan | Not documented (labels + metadata) | Chinese regime: explicit + implicit | Label visible; marker invisible | No (metadata is deterministic) |
| Any platform (Unicode carriers) | Possible | Bytes in the string | Invisible | Deterministic scan, no key |

## Main sources

1. Anthropic Support — [How Claude marks AI-generated content](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
2. Dathathri et al. — [Scalable watermarking for identifying large language model outputs](https://www.nature.com/articles/s41586-024-08025-4) (Nature 634, 818–823, 2024) — SynthID-Text
3. TechGrapple — [Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
4. Nature — [Can Anthropic's invisible watermarks curb 'AI slop'?](https://www.nature.com/articles/d41586-026-02503-7)
5. MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
6. European Commission — [EU AI Act Transparency Code](https://digital-strategy.ec.europa.eu/en/policies/guidelines-ai-transparency-obligations)
7. `watermarks-remover` — [GitHub](https://github.com/guillaumemeyer/watermarks-remover) (Unicode layer vs statistical layer, per-vendor notes)

## Verification status (17 September 2026)

**Primary-verified in this revision**: Anthropic support page (embedded watermark, C2PA, model support list, detection in private preview); MirrorMark (arXiv:2601.22246) and MCmark (arXiv:2502.11268); SynthID-Text (Nature 634, 818–823, 2024) and the official SynthID page (detection via Gemini and the Detector portal for media, none for text); EU Code of Practice timeline (TechCrunch, 11 August 2026); Meta's Stable Signature, AudioSeal and Video Seal; the `invisible-watermark` library; `watermarks-remover` (Layer A/B, vendor notes).

**Secondary-only**: Google's "~100 % of new generations" and "100+ billion" figures and the SynthID adopter list (OpenAI, NVIDIA, ElevenLabs, Kakao); Suno and Substack announcements; Chinese vendors' hosted labeling details (no direct vendor page fetched).

**Unconfirmed / to watch**: the retirement of the SynthID-text detector on Google's API (reported only by `watermarks-remover`; consistent with the absence of user-facing text detection, but not confirmed by Google); Anthropic's technical detection documentation (pending publication); whether any Chinese vendor adds a statistical mark beyond labels — unknown, not absent.

**Known external deadlines that will change this dossier**: end of the EU transition period for pre-August-2026 models (2 December 2026); OpenAI's text-mark decision; Google's SynthID Detector general availability.
