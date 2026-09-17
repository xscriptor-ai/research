---
title: "Others — Meta, Microsoft, Mistral, Cohere, Suno, Substack, xAI"
description: "Code signatories and peripheral cases with partial or pending signals."
date: "2026-08-14"
tags: ["technology", "legislation", "research"]
---

# Others — the rest of the ecosystem

> *Signing a code of good practice is easy; deploying a mark in the text of every response is another thing. Between one and the other lies most of the sector.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

The exhaustive company-by-company catalogue is in [Landscape by company/model](../companies.md); this folder keeps the group analysis.

## 1. Code signatories

Besides Anthropic, OpenAI and Google, the Section 1 list of the Code of Practice includes **Meta, Microsoft, Mistral and Cohere**. Signing commits to making content machine-identifiable within the AI Act deadlines, but does **not** oblige a specific method or an embedded text mark.

## 2. Company by company

### Meta
- Code signatory.
- **Media marks researched and open-sourced**: *Stable Signature* for images (learned into the decoder, integrated into part of its generators), *AudioSeal* for speech (localized audio watermarking with a fast detector), *Video Seal* for video (neural video watermarking).
- Its public stance on **text** marking has been oscillating: in debates over California's AI bill it opposed mandatory text watermarking over quality and false-positive concerns. No documented deployment of a statistical mark in its models' text.

### Microsoft
- Code signatory.
- Provenance signals mainly at **product level** (metadata, labels, C2PA in Copilot, Designer/Bing image surfaces and Azure OpenAI).
- Does not document a model-level embedded text mark in the style of Claude.

### Amazon
- Titan Image Generator applies a **tamper-resistant invisible watermark** by default, with a detection API in Bedrock; C2PA support.
- Text (Nova): no documented statistical mark.

### Adobe
- **Content Credentials (C2PA)** on generated images and video and across editing workflows; co-founder of the Content Authenticity Initiative. Firefly does not generate text, so there is no text mark.

### Stability AI
- Invisible frequency-domain watermark applied to generated images by default (the `invisible-watermark` library: DWT+DCT, plus learned variants); per its own tests it does not survive resize or rotation well.
- No LLM product; no text mark.

### ElevenLabs
- **Audio**: inaudible marking of generated speech per company announcements; listed among SynthID adopters by the sources cited in the Google folder. No text output.

### Mistral
- Content marking mainly **visible and metadata-based** on its platforms (Le Chat, La Plateforme).
- Also publishes **open-weight** models (*maestro*/*mod* range): run locally, they carry no provider mark.

### Cohere
- Provenance signals at platform level; no documented embedded text mark.

## 3. Peripheral cases

- **Suno** (AI music): announced (August 6, 2026) it will mark tracks created on its platform, amid a wave of litigation. It is **audio** marking, not text.
- **Substack**: alliance with **Pangram** (July 2026) to flag AI-generated content in newsletters — third-party detection/labeling, not a generator-embedded mark.
- **xAI / Grok**: no public documentation of a text watermark at the closing of this research.

## 4. Reading

Most of the sector is **at the entry gate**: it has signed the code and deploys labels and metadata, but does not yet embed statistical marks in text. Pressure to do so will come from three sides: the AI Act (making marking mandatory), the SynthID standard (already adopted by OpenAI, NVIDIA and ElevenLabs) and competition — when a mark becomes the trust criterion of editors, employers and clients, a company's absence of a mark will turn into an argument against it.

## References

- European Commission, *Strong backing for the Code of Practice on Transparency of AI-Generated Content* — [link](https://digital-strategy.ec.europa.eu/en/news/strong-backing-code-practice-transparency-ai-generated-content)
- Meta, *Stable Signature* — [arXiv:2307.15521](https://arxiv.org/abs/2307.15521)
- Meta, *AudioSeal — Proactive Detection of Voice Cloning with Localized Watermarking* — [arXiv:2401.17264](https://arxiv.org/abs/2401.17264)
- Meta, *Video Seal: Open and Efficient Video Watermarking* — [arXiv:2412.09492](https://arxiv.org/abs/2412.09492)
- `invisible-watermark` (image watermarking library) — [GitHub](https://github.com/ShieldMnt/invisible-watermark)
- TechCrunch, *Suno will start watermarking songs* — [link](https://techcrunch.com/2026/08/06/amid-legal-battles-suno-says-it-will-start-watermarking-songs/)
- Substack, *Against Claudefishing* — [link](https://post.substack.com/p/against-claudefishing)
- [Landscape by company/model](../companies.md) — exhaustive catalogue
