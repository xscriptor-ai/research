---
title: "OpenAI — ChatGPT"
description: "State of text marking at OpenAI: researched in 2024, not deployed."
date: "2026-08-14"
tags: ["technology", "legislation", "privacy"]
---

# OpenAI — ChatGPT

> *OpenAI has the technique ready and the research done. It has not deployed it in ChatGPT's text —perhaps out of fear of false positives— but it has explicitly said that text provenance is a future direction.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

## 1. State as of August 11, 2026

- **Does not document** a watermark embedded in ordinary ChatGPT text.
- Its current provenance system (based on **C2PA** and Google's **SynthID** standard) covers supported **images and audio**.
- Its help page indicates the goal is to **expand provenance signals to all formats, including text** — language of a *future direction*, not of an ongoing deployment.

## 2. The 2024 research

In its public essay *Understanding the source of what we see and hear online* (2024), OpenAI described a text marking method with:

- **high accuracy** in its tests;
- **resistance to local changes** (minor edits);
- and two recognized weaknesses that sound prophetic today:
  - **mass rewriting and translation** destroy the signal;
  - **false positives hit vulnerable populations** — for example, non-native speakers who use AI as a writing aid.

This second concern is likely the main reason OpenAI has **not deployed** the method in ChatGPT: a public detector with false positives on human text carries a high reputational and ethical cost. Anthropic, with the same *distortion-free* design, has accepted that risk in the name of AI Act compliance.

## 3. Regulatory position

- **Signatory** of the European Code of Practice on transparency of AI-generated content (Section 1 list), alongside Anthropic, Google, Meta, Microsoft, Mistral and Cohere.
- Has adopted **SynthID** for its image and audio provenance signals.
- European compliance does not require copying Anthropic's method: each company may use a different technique as long as content is machine-identifiable.

## 4. Reading

OpenAI is the case that confirms the rule: text marking technology **exists and works** (they proved it in 2024), but its deployment is a **political** decision, not a technical one. The moment it deploys will be set by the AI Act itself and by competitive pressure: if paying customers begin to demand "unmarked text" as a differentiator, or if verification becomes an industry standard, OpenAI will have to choose between losing that advantage or giving it up.

## References

- OpenAI, *Understanding the source of what we see and hear online* (2024) — [link](https://openai.com/index/understanding-the-source-of-what-we-see-and-hear-online/)
- European Commission, *Strong backing for the Code of Practice on Transparency of AI-Generated Content* — [link](https://digital-strategy.ec.europa.eu/en/news/strong-backing-code-practice-transparency-ai-generated-content)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
