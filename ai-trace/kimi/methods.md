---
title: "Kimi — Marking methods"
description: "Explicit and implicit identifiers of Moonshot AI: visible ones and metadata-based ones."
date: "2026-08-14"
tags: ["technology", "privacy", "research"]
---

# Kimi — Marking methods

> *Kimi distinguishes between the label that is seen and the marker that travels in the metadata. Neither of them lives in the text that is copied and pasted.*

## 1. The two Moonshot AI identifiers

Kimi's terms (*AI-Generated Content Identification Rules*) define two types of identifiers applied to text, images, audio and video:

- **Explicit identifier**: something the user sees or hears — "AI-generated" label, visible mark, page banner.
- **Implicit identifier**: a **technical marker in the file metadata** for traceability.

Moonshot adds both by default to generated content, except for approved functions that allow removing the explicit identifier.

## 2. The difference with Claude

The resemblance to Anthropic is superficial:

| | Kimi | Claude |
|---|---|---|
| Implicit identifier | Marker in **file metadata** | **Mark in the text itself** |
| Survives copy-paste | No | Yes |
| Survives conversion | No | Partially |

There is no official Kimi K3 documentation describing a **statistical model-level mark** embedded in normal text generation.

## 3. The open model

Kimi K3 is also distributed as an open model (MoonshotAI's official repo). Run locally:

- it does not pass through kimi.com's interface or its labeling layer;
- the inference software controls whether labels or metadata are added;
- no mandatory mark is documented for local generation.

## 4. Regime

Kimi complies with the Chinese framework for synthetic content identification (September 2025) and **does not appear** among the European Code of Practice signatories.

## References

- Moonshot AI, *Kimi terms — AI-Generated Content Identification Rules* — [link](https://www.kimi.com/user/agreement/modelUse?version=v2)
- Kimi K3 — [GitHub MoonshotAI](https://github.com/MoonshotAI/Kimi-K3)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
