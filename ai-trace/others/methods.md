---
title: "Others — Marking methods"
description: "State of the methods of Meta, Microsoft, Mistral, Cohere, Suno, Substack and xAI."
date: "2026-08-14"
tags: ["technology", "research"]
---

# Others — Marking methods

> *Signing a code of good practice is easy; deploying a mark in the text of every response is another thing. Between one and the other lies most of the sector.*

## 1. Meta

- **Image**: has published *Stable Signature*, a method that embeds a signature **in the output of generative image models** (the mark is learned during decoder fine-tuning), integrated into part of its generators.
- **Text**: no documented deployment of a statistical mark. In the debates over California's AI bill it opposed mandatory text marking over quality and false-positive concerns.
- **Stance**: oscillating — it researches marks for media, resists text marking.

## 2. Microsoft

- Provenance signals at **product level**: metadata, labels and C2PA in Copilot and Azure OpenAI surfaces.
- No documented model-level embedded text mark.

## 3. Mistral

- Content marking mainly **visible and metadata-based** on its platforms (Le Chat, La Plateforme).
- Also publishes **open-weight** models (*maestro*/*mod* range): run locally, they carry no mark.

## 4. Cohere

- Provenance signals at platform level; no documented embedded text mark.

## 5. Peripheral cases

- **Suno** (music): announced (August 6, 2026) it will mark tracks generated on its platform, amid litigation. It is **audio** marking, not text.
- **Substack**: alliance with **Pangram** (July 2026) to flag AI-generated content in newsletters — third-party detection/labeling, not a generator-embedded mark.
- **xAI / Grok**: no public documentation of a text watermark at the closing of this research.

## References

- European Commission, *Strong backing for the Code of Practice* — [link](https://digital-strategy.ec.europa.eu/en/news/strong-backing-code-practice-transparency-ai-generated-content)
- TechCrunch, *Suno will start watermarking songs* — [link](https://techcrunch.com/2026/08/06/amid-legal-battles-suno-says-it-will-start-watermarking-songs/)
- Substack, *Against Claudefishing* — [link](https://post.substack.com/p/against-claudefishing)
