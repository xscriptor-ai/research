---
title: "Index — Invisible AI traces in language models"
description: "Research dossier on AI-generated content watermarking, company by company: methods, detection, and mitigation."
date: "2026-08-14"
tags: ["technology", "cryptography", "privacy", "legislation", "research"]
---

# Invisible AI traces in language models

> *A text that can be attributed is a text that no longer fully belongs to whoever wrote it. The question is not whether the mark is visible, but whether authorship is still free.*

Research dossier on **AI-generated content marking** following the European AI Act (Art. 50) and Chinese regulation: what technique each company uses, what they have disclosed, and **how the signature is reduced or removed**. Verified as of August 2026.

## Index

| Section | Content |
|---------|---------|
| [**Mitigation (common)**](mitigation.md) | Mitigation is essentially the same for every company: destroy the correlation between the text and the sampler's key. Attack taxonomy, evaluation, and the structural exemption of open-weight models. |
| [**Worked example**](mitigation-example.md) | A text before and after mitigation: original, light editing, and rewriting with another sampler, with what happens in the invisible layer explained. |
| [**Local models without a trace**](local-models.md) | Open-weight families (DeepSeek, Llama, Qwen, Gemma, Mistral, Kimi, Phi, GLM) run locally, inference engines, and why there is no mark in that scenario — with the API and intermediary caveat. |
| [**Agents and prompts**](agents-and-prompts.md) | Why neither a prompt nor building an agent prevents the mark (it lives in the provider's sampler), and how it is mitigated by changing the sampler the agent accesses. |
| [**The sampler**](sampler.md) | What the sampler is and what it consists of: the component that turns the model's distribution into a concrete token — and why it is the home of the watermark. |
| [**Anthropic — Claude**](anthropic/README.md) | Full dossier: regulatory context, technique, statistical pattern (MirrorMark), detection and limitations, mitigation. |
| [**Google — Gemini / SynthID**](google/README.md) | Methods (`methods.md`) and specific mitigation (`mitigation.md`): mark on text, image, audio and video. |
| [**OpenAI — ChatGPT**](openai/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): 2024 research, C2PA/SynthID, no text deployment yet. |
| [**DeepSeek — V4**](deepseek/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): Chinese labeling, open weights without a mark. |
| [**Kimi — K3 (Moonshot)**](kimi/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): visible identifiers and metadata-based ones. |
| [**Others — Meta, Microsoft, Mistral, Cohere…**](others/README.md) | Methods (`methods.md`) and mitigation (`mitigation.md`): Code signatories with partial signals. |

## Folder structure

Each company has its own folder at the same level, with three documents:

```
company/
  README.md       — summary card and internal links
  methods.md      — the company's own marking methods
  mitigation.md   — mitigation specifics (the common part lives in the root)
```

## Core idea

All text watermarks that survive copy-and-paste live **in the sampler**:

$$
\text{mark present} \iff \text{the sampler uses the key } K
$$

From this follows the rule running through the whole dossier: **whoever controls the sampler decides**. Providers mark in their inference layer; whoever runs open weights locally is their own sampler and carries no mark.

## Landscape summary

| Company / model | Text mark | Method | Public verification |
|------------------|-----------|--------|----------------------|
| Google (SynthID-Text) | Yes (~100 % of new generations) | Statistical in sampling | Private |
| Anthropic (Claude) | Yes (confirmed) | Likely MirrorMark | Announced, not published |
| OpenAI (ChatGPT) | Not deployed | C2PA/SynthID (image/audio) | Supported formats only |
| DeepSeek V4 | Not documented | Labeling (Chinese rule) | No |
| Kimi K3 | Not documented | Metadata + label | No |

## Main sources

1. Anthropic Support — [How Claude marks AI-generated content](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
2. TechGrapple — [Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
3. Nature — [Can Anthropic's invisible watermarks curb 'AI slop'?](https://www.nature.com/articles/d41586-026-02503-7)
4. MirrorMark — [arXiv:2601.22246](https://arxiv.org/abs/2601.22246)
5. European Commission — [EU AI Act Transparency Code](https://digital-strategy.ec.europa.eu/en/policies/guidelines-ai-transparency-obligations)
