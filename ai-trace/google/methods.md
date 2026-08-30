---
title: "Google — Marking methods"
description: "How SynthID works in text, image, audio and video."
date: "2026-08-14"
tags: ["technology", "cryptography", "research"]
---

# Google — Marking methods

> *Google does not mark one thing: it marks everything. And it does so in the layer that is not seen — the perceptual signal for media, the statistical one for text.*

## 1. SynthID: the architecture

**SynthID** (Google DeepMind) is a content marking system combining **two families of technique** by format:

| Format | Technique | Signal |
|---------|-----------|--------|
| Text | Statistical in token sampling | Imperceptible, survives copy-paste |
| Image | Perceptual mark in the pixels | Imperceptible, survives crops and format changes |
| Audio | Mark in the spectrogram | Imperceptible |
| Video | Perceptual mark per frame | Imperceptible |

## 2. SynthID-Text: the mark in token choice

The text is marked **during generation**, manipulating the randomness of token selection without altering the perceived distribution. It is the same technical family the dossier attributes to Claude (see [Anthropic — 03 Statistical pattern](../anthropic/03-statistical-pattern.md)):

- the mark is a statistical pattern embedded in sampling, reproducible with Google's secret key;
- detection aggregates evidence over the whole passage; short passages do not accumulate enough signal;
- it survives copy-paste and minor edits; paraphrasing and translation degrade it.

## 3. SynthID in image, audio and video: the perceptual mark

For media, SynthID does not rely on metadata but on **perceptual content**:

- the image carries a mark in the pixels that **survives screenshots, re-sharing on social networks and format changes** — properties metadata (EXIF, C2PA) do not offer on their own;
- detection goes through Google's own channel (private keys).

## 4. Scale and coverage (2026)

- **~100 % of new generations** of Google's AI products (Gemini, Search, Workspace);
- more than **100 billion** marked pieces;
- extended to **Chrome and Search** as a verification layer;
- adopted as a standard by **OpenAI, NVIDIA, ElevenLabs and Kakao**.

## 5. Verification

Detection **is not public**: SynthID's keys are private and verification is offered through Google's channel. That prevents third parties from calibrating attacks against the real detector (the same asymmetry described in [Anthropic — 04](../anthropic/04-detection-and-limitations.md)).

## References

- BuildMvpFast, *SynthID Becomes the Standard* — [link](https://www.buildmvpfast.com/blog/synthid-content-provenance-c2pa-watermarking-ai-2026)
- Perplexity AI Magazine, *SynthID 2026* — [link](https://perplexityaimagazine.com/ai-news/synthid-openai-elevenlabs-nvidia-ai-watermark-standard-2026/)
- TextSight, *Google SynthID watermarking explained* — [link](https://www.textsight.ai/blog/google-synthid-watermarking-explained/)
