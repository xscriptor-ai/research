---
title: "Landscape — marking by company and model"
description: "Company-by-company and model-by-model catalogue of AI content marking: what mark each one uses, in which format, whether it survives copy-paste and whether it is visible. Verified August–September 2026."
date: "2026-09-17"
tags: ["technology", "cryptography", "privacy", "legislation", "research"]
---

# Landscape — marking by company and model

> *Every company tells the same story with different layers: the label is the part the user sees, the metadata the part the file carries, and the watermark the part that survives when both are gone. This catalogue separates the three.*

This document is the exhaustive company/model catalogue of the dossier. The dedicated folders ([Anthropic](anthropic/README.md), [Google](google/README.md), [OpenAI](openai/README.md), [DeepSeek](deepseek/README.md), [Kimi](kimi/README.md), [Others](others/README.md)) keep the detailed analysis for the cases with the most evidence; this catalogue covers the whole ecosystem in one place. The mitigation layer is in [common mitigation](mitigation.md), [Unicode carriers](unicode.md) and [local models](local-models.md).

## 0. How to read this catalogue

Mark classes, ordered by whether they survive copying text and pasting it elsewhere:

| Class | Carrier | Survives copy-paste? | Visible? | Keyed? | Detection |
|-------|---------|----------------------|----------|--------|-----------|
| **Statistical (sampling)** | Randomness of token choice | **Yes** (plain text) | No | Yes (provider key) | Statistical, with key/detector |
| **Unicode carrier** | Invisible characters in the string | **Yes** (unless destination sanitizes) | No | Usually not | Deterministic scan |
| **Weight-based** | Model weights (fine-tuning/instruction marks) | **Yes** | No | Yes | Keyed / trigger-based |
| **Post-hoc editing** | Text rewritten after generation (synonyms, syntax) | **Yes** | No | Usually not | Rule-based / semantic |
| **Perceptual (media)** | Pixels, spectrogram, frames | n/a for text; survives re-encoding | No | Per scheme | Model or keyed |
| **Metadata** | C2PA/EXIF/XMP/document properties | **No** with copied text; yes with files | No | Signature (C2PA) | Cryptographic verification |
| **Visible label** | UI badge, banner, moving mark | No | **Yes** | No | By eye |

**Evidence levels used below**: **[P]** primary documentation (company page, paper, spec); **[S]** secondary coverage; **[U]** no public documentation found as of September 2026 — which means *unknown, not absent* (see [DeepSeek — mitigation](deepseek/mitigation.md) §3).

## 1. Documented statistical text marks (deployed)

### Google — Gemini / SynthID-Text
- **[P]** *Generative* statistical mark in the sampling procedure: **Tournament sampling**, published in Nature (Dathathri et al., *Scalable watermarking for identifying large language model outputs*, Nature 634, 818–823, 2024), deployed in Gemini and Gemini Advanced. Seed = hash of the last 4 tokens + key; detection scores correlations without running the LLM.
- Configurable: non-distortionary (quality preserved; validated on ~20 million live Gemini responses) or distortionary (more detectable, some quality cost).
- Weakens with low entropy (deterministic answers) and with paraphrase/edits, as the paper itself documents.
- Image, audio and video: **SynthID** perceptual variants; adopted as a reference by other vendors (see section 5).
- **[S]** The dossier's sources claim ~100 % of new Google generations and 100+ billion marked items; detection is offered through Google's channel (private keys), not publicly.
- **[S]** Third-party tooling (`watermarks-remover`, vendor notes) reports that Google **retired the SynthID-text detector on its API in August 2026** — pending confirmation; it does not imply the generation-side mark was retired.

### Anthropic — Claude
- **[P]** Embedded watermark in text, announced 11 August 2026 under the EU Code of Practice (Art. 50(2)); applied at the **model level** (generation), travels with copy-paste, may persist through some editing.
- **[P]** Model support (support page, September 2026): text watermarks on first-party surfaces for Claude Fable 5.1 / Fable 5 / Mythos 5.1 / Mythos 5 / Opus 5 / Opus 4.8–4.5 / Sonnet 5 / Sonnet 4.6 / Sonnet 4.5 / Haiku 4.5, with cloud-partner (AWS, Google Cloud, Microsoft Foundry) roll-out in progress — Opus 5 from 14 September 2026; C2PA in generated files.
- **[P]** Detection: **private preview** API for eligible organizations (regulators, law enforcement, media, fact-checkers, researchers, education, EU civil society, and enterprises with their own compliance duties); technical documentation pending.
- Statistical pattern not published; inference points to the distortion-free multi-bit family with **MirrorMark** as the best candidate — full analysis in [Anthropic — 03](anthropic/03-statistical-pattern.md).

## 2. Provenance without a documented statistical text mark

### OpenAI — ChatGPT
- **[P]** 2024 essay documents a researched text mark (accurate, edit-resistant, weak against mass rewriting/translation) — **not deployed** in ChatGPT text.
- Images/audio: C2PA metadata and adoption of the SynthID standard for provenance where supported; Sora video additionally carries a visible moving mark and C2PA metadata. Text: "future direction".

### Meta — Llama and generative media
- **[P]** *Stable Signature* (arXiv:2307.15521): watermark learned into the image decoder's output; deployed in parts of its generators.
- **[P]** *AudioSeal* (arXiv:2401.17264): localized audio watermarking with a fast detector, open-sourced.
- **[P]** *Video Seal* (arXiv:2412.09492): open-sourced neural video watermarking.
- Text: **[U]** no documented statistical text mark; publicly opposed mandatory text watermarking over quality/false-positive concerns during the California AI bill debates.

### Microsoft — Copilot / Azure
- **[P]** Product-level provenance: Content Credentials (C2PA) on AI-generated images in Designer/Bing surfaces and metadata/labels in Copilot and Azure OpenAI offerings.
- Text: **[U]** no documented model-level embedded text mark.

### Amazon — Titan / Nova / Bedrock
- **[P]** Titan Image Generator applies an invisible, tamper-resistant watermark by default, with a detection API in Bedrock; C2PA support.
- Text (Nova): **[U]** no documented statistical text mark.

### Adobe — Firefly
- **[P]** Content Credentials (C2PA) on generated images/video and in editing workflows; Adobe co-founded the Content Authenticity Initiative. No text mark: text is not Firefly's output format.

### Stability AI — Stable Diffusion
- **[P]** Invisible frequency-domain watermark applied to generated images (the `invisible-watermark` library, DWT+DCT, and derivatives); API outputs carry it by default. Not robust to resize/rotation, per the library's own tests.
- Text: **[U]** no mark (no LLM product).

### NVIDIA
- **[S]** Listed among SynthID adopters for content provenance on its platforms; **[U]** no own statistical text mark documented.

### Mistral AI
- **[P/S]** Labels and metadata on Le Chat/La Plateforme; **[P]** open weights (`maestro`/`mod` range). Text: **[U]** no documented statistical mark; locally run weights carry no provider mark ([local models](local-models.md)).

### Cohere
- **[S]** Platform-level provenance signals; **[U]** no documented embedded text mark.

### xAI — Grok
- **[U]** No public documentation of a text watermark at the closing of this research.

### ElevenLabs — audio
- **[P/S]** Inaudible audio marking of generated speech per company announcements; listed among SynthID adopters by the sources in [Google](google/README.md). No text output.

### Suno / Udio — music
- **[S]** Suno announced (6 August 2026) it will mark tracks generated on its platform, amid litigation. Audio marking, not text.

### Substack + Pangram
- **[S]** Alliance (July 2026) to flag AI-generated content in newsletters: third-party **detection/labeling**, not a generator-embedded mark.

### Black Forest Labs / Synthesia / Kakao
- **[S]** Code signatories / SynthID adopters with partial or pending signals; no documented model-level statistical text mark.

## 3. Chinese regime — labels and implicit metadata

The *Measures for the Identification of AI-Generated Synthetic Content* (in force since September 2025) require **explicit** identifiers (visible labels) and **implicit** ones (technical markers, typically in file metadata). It is a labeling regime, not a statistical-marking mandate; providers comply through labels and metadata, and the regime distinguishes them from the European route (mark so third parties can detect).

| Company / model | Hosted products | Open weights | Token-level statistical mark |
|-----------------|-----------------|--------------|------------------------------|
| DeepSeek — V4 / R1 | Visible labels + terms against removal; API disclosure duty | Yes (Hugging Face) | **[U]** not documented; local run = no provider mark ([dossier](deepseek/README.md)) |
| Moonshot — Kimi K3 / K2.6 | Explicit label + implicit metadata marker | Yes | **[U]** not documented ([dossier](kimi/README.md)) |
| Alibaba — Qwen 3.x | Labels on hosted apps | Yes | **[U]** not documented |
| Zhipu / Z.AI — GLM | Labels on hosted apps | Yes (MIT license per [local models](local-models.md)) | **[U]** not documented |
| Baidu — Ernie | Labels on hosted apps | No | **[U]** not documented |
| ByteDance — Doubao / Seed | Labels + implicit markers on video too | Yes (Seed-OSS family) | **[U]** not documented |
| Tencent — Hunyuan | Labels on hosted apps | Yes | **[U]** not documented |
| MiniMax / 01.AI / iFlytek | Labels under the same rule | Partially open | **[U]** not documented |

The structural consequence repeats across the table: on the hosted service the regime adds **labels and metadata**; whoever downloads the weights and runs them with their own engine gets **no provider mark** — the exemption of the sampler, valid exactly as long as the sampler is theirs ([local models](local-models.md)).

## 4. Open weights — the structural exemption

Families with downloadable weights (DeepSeek V4/V4 Flash/V4 Pro/R1, Llama 4/5, Qwen 3.5/3.6, Gemma 4, Mistral, Kimi K2.6/K3, Phi-4, GLM-5.1, and the Chinese open releases above): run locally with llama.cpp, Ollama, vLLM, LM Studio or Transformers, they carry **no provider mark**, because the mark lives in the provider's sampler and the weights are inert. Two caveats, detailed in [local models](local-models.md): engines that ship watermarking layers (e.g. keyed-Gumbel marks in self-hosted engines such as `arbi-serve`), and marks baked into weights by fine-tuning (rare, not documented in these families).

## 5. Cross-company summary

| Company / model | Text mark (copy-paste) | Carrier | Visible? | Verification |
|-----------------|------------------------|---------|----------|--------------|
| Google — Gemini | **Yes (statistical)** | Sampling (Tournament) | No | Private, Google channel |
| Anthropic — Claude | **Yes (statistical)** | Sampling (multi-bit family) | No | Private preview API |
| OpenAI — ChatGPT | No (text pending) | C2PA/SynthID on media | No | C2PA tools / provider |
| Meta — Llama/media | No text mark (media marks yes) | Learned decoder / neural | No | Provider/OSS detectors |
| Microsoft / Amazon / Adobe / Stability | No text mark (media: metadata/learned) | C2PA / invisible watermark | No | C2PA tools / provider |
| DeepSeek / Kimi / Qwen / GLM / Ernie / Doubao / Hunyuan | No documented text mark (labels/metadata) | Unicode/UI/metadata in hosted apps | Label: yes | Deterministic (label/metadata) |
| Platform marks (any) | **Possible (Unicode carriers)** | Bytes (zero-width, tags, spaces) | No | Deterministic scan ([unicode.md](unicode.md)) |

Read alongside the two implicit columns of this dossier: **statistical marks** (keyed, provider-side, only degradable) and **Unicode carriers** (bytes, usually unkeyed, exactly removable with typographic care).

## 6. Method note

"[P]" entries were verified against primary sources during August–September 2026: Nature 634, 818–823 (SynthID-Text), Anthropic support page, arXiv:2307.15521 / 2401.17264 / 2412.09492, the `invisible-watermark` library, and the dossier's own primary fetches (TechCrunch, Search Engine Journal, MirrorMark arXiv:2601.22246). "[S]" entries rest on the secondary coverage cited in the dedicated folders. "[U]" is a statement about public documentation, not about reality: the absence of a document is not a guarantee of absence, and it can change without notice because the sampler belongs to the provider.

## References

- Dathathri et al., *Scalable watermarking for identifying large language model outputs*, Nature 634, 818–823 (2024) — [Nature](https://www.nature.com/articles/s41586-024-08025-4)
- Anthropic Support, *How Claude marks AI-generated content* — [link](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
- OpenAI, *Understanding the source of what we see and hear online* (2024) — [link](https://openai.com/index/understanding-the-source-of-what-we-see-and-hear-online/)
- Meta: Stable Signature — [arXiv:2307.15521](https://arxiv.org/abs/2307.15521); AudioSeal — [arXiv:2401.17264](https://arxiv.org/abs/2401.17264); Video Seal — [arXiv:2412.09492](https://arxiv.org/abs/2412.09492)
- `invisible-watermark` (DWT+DCT image watermarking) — [GitHub](https://github.com/ShieldMnt/invisible-watermark)
- `watermarks-remover` (Unicode vs statistical layers; vendor notes) — [GitHub](https://github.com/guillaumemeyer/watermarks-remover)
- TechCrunch, *Anthropic says it will watermark text generated by its AI models* — [link](https://techcrunch.com/2026/08/11/anthropic-says-it-will-watermark-text-generated-by-its-ai-models/)
- [Dossier index](README.md) · [Unicode carriers](unicode.md) · [Common mitigation](mitigation.md) · [Local models](local-models.md)
