---
title: "OpenAI — Marking methods"
description: "The 2024 research and the current state: C2PA/SynthID for image and audio, text pending."
date: "2026-08-14"
tags: ["technology", "cryptography", "research"]
---

# OpenAI — Marking methods

> *OpenAI has had the technique proven since 2024 and has not deployed it in text. It is the case that shows deploying a mark is a political decision, not a technical one.*

## 1. Current state (August 2026)

OpenAI documents provenance signals for supported **images and audio**, based on:

- **C2PA** (signed provenance metadata);
- Google's **SynthID** standard, adopted as a verification format.

For **ordinary ChatGPT text** it documents no embedded mark. Its help page indicates the goal is **expanding provenance to all formats, including text** — future-direction language, not deployment.

## 2. The method researched in 2024

In *Understanding the source of what we see and hear online* (2024), OpenAI described a text marking method with:

- **high accuracy** in internal tests;
- **resistance to local changes** (minor edits survive);
- **two recognized weaknesses**:
  - **mass rewriting and translation** destroy the signal;
  - the risk of **false positives** on vulnerable populations (e.g., non-native speakers using AI as a writing aid).

The method belongs to the **statistical token-level family** (the same as SynthID-Text and Claude's candidate, MirrorMark): the mark lives in token choice, not in visible bytes.

## 3. Why it has not been deployed

Two complementary readings:

1. **Cost of false positives**: a public detector that flags human text as AI hurts students, writers and non-native speakers. OpenAI flagged this risk in 2024, and deploying now —when Anthropic has just accepted it— would put OpenAI at a perception disadvantage.
2. **Product strategy**: ChatGPT is the mass-consumption product; a watermark perceived as "surveillance" could push users toward alternatives (including the open weights this dossier documents).

## 4. What could change

The European AI Act (Art. 50) and the Code of Practice (which OpenAI signed) pressure text to also be machine-identifiable. It is reasonable to expect OpenAI to eventually deploy a text mark — it has the technique ready — perhaps first on the API and then in the consumer product.

## References

- OpenAI, *Understanding the source of what we see and hear online* (2024) — [link](https://openai.com/index/understanding-the-source-of-what-we-see-and-hear-online/)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
