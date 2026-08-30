---
title: "DeepSeek — V4"
description: "Chinese-rule labeling; no documented statistical mark; open weights."
date: "2026-08-14"
tags: ["technology", "privacy", "research"]
---

# DeepSeek — V4

> *DeepSeek's position is the ecosystem's structural exemption: open model + own regulatory regime = no statistical watermark in the text the user runs locally. But that exemption belongs to the sampler, not the model: through the API or an intermediary, the sampler belongs to the provider and the absence of a mark stops being a guarantee.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

## 1. Regulatory regime

DeepSeek operates under Chinese regulation, not the European AI Act. The relevant rule is the *Measures for the Identification of AI-Generated Synthetic Content* (in force since **September 2025**), which distinguish between **visible** identification and less visible technical identification.

## 2. What it does and what it does not do

- **Yes**: its hosted platform adds **identifiers** of AI-generated content (visible labeling) to comply with the Chinese rules. Its terms tell users not to remove, alter, hide, forge or bypass those identifiers.
- **Yes**: the **API** terms (Open Platform) oblige developers to inform end users that the output is AI-generated.
- **No**: there is no public documentation of a **hidden statistical mark at the token level** in the style of Claude that survives copy-paste in ordinary DeepSeek V4 text.
- **No**: DeepSeek **does not appear** on the European signatories list (Section 1); it complies through its own route.

## 3. Open weights

DeepSeek V4 and **DeepSeek V4 Flash** are distributed as **open weights** (for example, on Hugging Face). Whoever runs them locally:

- does not pass through the platform's labeling layer;
- controls their **own sampler** and interface;
- therefore, the open model **does not produce a watermark by itself** — unless the inference software adds one.

This is the *layer* difference that structures the whole analysis: the model, the inference engine, the server, the interface and the export system are separate layers. The mark lives in the sampling layer; DeepSeek controls it in its hosted service (with visible labels) but **not** in the weights the user downloads.

## 4. Reading

DeepSeek demonstrates that AI content marking is not a single phenomenon but **three parallel regimes**: the European one (mark so third parties can detect), the Chinese one (label to comply with the state rule) and the open one (no mark, because the sampler belongs to the user). For those who want text without a signature, DeepSeek's open models are one of the most accessible lateral routes in the ecosystem.

## References

- DeepSeek, *Open Platform Terms of Service* — [link](https://cdn.deepseek.com/policies/en-US/deepseek-open-platform-terms-of-service.html)
- DeepSeek V4 Flash — [Hugging Face](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
