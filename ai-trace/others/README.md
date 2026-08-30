---
title: "Others — Meta, Microsoft, Mistral, Cohere, Suno, Substack, xAI"
description: "Code signatories and peripheral cases with partial or pending signals."
date: "2026-08-14"
tags: ["technology", "legislation", "research"]
---

# Others — the rest of the ecosystem

> *Signing a code of good practice is easy; deploying a mark in the text of every response is another thing. Between one and the other lies most of the sector.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

## 1. Code signatories

Besides Anthropic, OpenAI and Google, the Section 1 list of the Code of Practice includes **Meta, Microsoft, Mistral and Cohere**. Signing commits to making content machine-identifiable within the AI Act deadlines, but does **not** oblige a specific method or an embedded text mark.

## 2. Company by company

### Meta
- Code signatory.
- Has published content marking research for **images** (*Stable Signature*), integrated into part of its generators.
- Its public stance on **text** marking has been oscillating: in debates over California's AI bill it opposed mandatory text watermarking over quality and false-positive concerns. No documented deployment of a statistical mark in its models' text.

### Microsoft
- Code signatory.
- Provenance signals mainly at **product level** (metadata, labels, C2PA in Copilot and Azure OpenAI surfaces).
- Does not document a model-level embedded text mark in the style of Claude.

### Mistral
- Content marking mainly **visible and metadata-based** on its platforms (Le Chat, La Plateforme).
- Also publishes **open-weight** models (*maestro*/*mod* range): run locally, they carry no mark.

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
- TechCrunch, *Suno will start watermarking songs* — [link](https://techcrunch.com/2026/08/06/amid-legal-battles-suno-says-it-will-start-watermarking-songs/)
- Substack, *Against Claudefishing* — [link](https://post.substack.com/p/against-claudefishing)
