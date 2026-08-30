---
title: "Google — Specific mitigation"
description: "Mitigation specifics against SynthID, by format."
date: "2026-08-14"
tags: ["cryptography", "privacy", "technology"]
---

# Google — Specific mitigation

> *The general framework is in the root ([mitigation.md](../mitigation.md)). Here only what changes with Google is detailed: mitigation is no longer a single thing, but one per format.*

## 1. What is common (text)

For **SynthID-Text** the whole general framework applies: paraphrasing with another model, round-trip translation, re-sampling, open weights. There is no technical specificity in text; the only difference is that **the public detector does not exist** (private keys), so the evader cannot calibrate attacks against the real oracle. Paraphrasing remains the reliable route.

## 2. The decisive difference: media (image, audio, video)

SynthID's mark in media is **perceptual**, not metadata:

- **It is not removed by metadata stripping** — there are no bytes to delete; the signal is in the pixels or the spectrogram.
- `watermarks-remover` and similar tools **can score its presence but not remove it**.
- The only reliable route is **regeneration**: regenerating or processing the medium from scratch with another tool, or applying strong destructive transformations (aggressive re-compression, cropping, re-sampling) that degrade the signal at the cost of quality.

## 3. Table by format

| Format | Signal type | Metadata stripping? | Reliable route |
|---------|-------------|---------------------|----------------|
| Text (SynthID-Text) | Statistical | Not applicable | Paraphrase / re-sampling |
| Image | Perceptual | No | Regeneration / strong transformations |
| Audio | Perceptual | No | Regeneration / re-sampling |
| Video | Perceptual | No | Regeneration per frame |
| C2PA/EXIF metadata | Bytes | **Yes** | Deterministic stripping |

## 4. Reading

Google is the only case in the dossier where mitigation **bifurcates by format**: the text behaves like the rest of the ecosystem, but the perceptual mark on image/audio/video is structurally more expensive to remove. That asymmetry is deliberate: the perceptual mark survives what metadata does not — social traffic. Whoever wants to remove the mark from a Google medium has no "erasing": they have to rebuild the medium.

## References

- [Common mitigation](../mitigation.md)
- explainx.ai, *A Watermark Removal Tool Just Added OpenAI and Gemini Support* — [link](https://www.explainx.ai/blog/ai-watermark-removal-tool-openai-gemini-c2pa-august-2026)
