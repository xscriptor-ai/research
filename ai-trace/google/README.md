---
title: "Google — SynthID"
description: "The pioneer of AI content marking: image, audio, video and text."
date: "2026-08-14"
tags: ["technology", "cryptography", "research"]
---

# Google — SynthID

> *Google has not made a big regulatory proclamation like Anthropic; it has built infrastructure. The result: Google's mark is already woven into the ecosystem while Claude's has just been born.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

## 1. What SynthID is

**SynthID** is Google DeepMind's content marking system. It covers **image, audio, video and text** and has two components:

- **Perceptual mark** (image, audio, video): a signal embedded in the content itself that survives screenshots, re-sharing on social networks and format changes — something metadata (EXIF, C2PA) does not guarantee.
- **Text mark (SynthID-Text)**: embedded **during generation**, in token choice, without altering the perceived distribution. It is the direct relative of the technique the dossier attributes to Claude.

## 2. Coverage and scale (2026)

- Embedded in **~100 % of new generations** of Google's AI products (Gemini, Search, Workspace).
- Has marked **more than 100 billion** pieces of AI-generated content.
- Extended to **Chrome and Search**: the browser and the search engine check marks and content credentials.

## 3. As a de facto standard

SynthID is being adopted by third parties as a reference mechanism:

| Adopter | Scope |
|---------|-------|
| **OpenAI** | Provenance signals for its supported formats (image/audio) |
| **NVIDIA** | Content marking on its platforms |
| **ElevenLabs** | AI-generated audio |
| **Kakao** | Content in its products |

The sector's convergence on a single method has two faces: interoperability (a common detector) and a **single point of failure** (if the method is compromised, all signatories suffer it).

## 4. Verification

Detection **is not public**: Google keeps the keys private and offers verification through its own channel. It is the same asymmetry described in [04 — Detection and limitations](../anthropic/04-detection-and-limitations.md): without a public key, the evader cannot calibrate attacks against the real detector.

## 5. Critical reading

SynthID proves that content marking is viable and deployable at industrial scale. It also proves the **privatization of verification**: the mark exists so that *Google* can read it. The transparency pursued by the European AI Act depends on opening these mechanisms to third parties; until then, "transparency" is one-sided: the provider sees, the user does not.

## References

- BuildMvpFast, *SynthID Becomes the Standard: OpenAI, NVIDIA, ElevenLabs, Kakao* — [link](https://www.buildmvpfast.com/blog/synthid-content-provenance-c2pa-watermarking-ai-2026)
- Perplexity AI Magazine, *SynthID 2026: OpenAI, ElevenLabs, Nvidia adopt AI watermark* — [link](https://perplexityaimagazine.com/ai-news/synthid-openai-elevenlabs-nvidia-ai-watermark-standard-2026/)
- Presenc AI, *AI Content Watermarking Adoption 2026* — [link](https://presenc.ai/research/ai-content-watermarking-adoption-2026)
