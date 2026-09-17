---
title: "Kimi — K3 (Moonshot AI)"
description: "Explicit and implicit identifiers of Moonshot AI: visible ones and metadata-based ones — no statistical mark documented in the text."
date: "2026-08-14"
tags: ["technology", "privacy", "research"]
---

# Kimi — K3 (Moonshot AI)

> *Kimi distinguishes between the label that is seen and the marker that travels in the metadata. Neither of them lives in the text that is copied and pasted.*

**Documents in this folder:** [methods](methods.md) · [specific mitigation](mitigation.md) · [common mitigation](../mitigation.md)

## Executive summary

- **What it is**: compliance through **identifiers**, not a statistical watermark. Moonshot's *AI-Generated Content Identification Rules* define two types: an **explicit identifier** (label, visible mark or banner) and an **implicit identifier** (technical marker in **file metadata**).
- **Where it lives**: interface and metadata — **never in the text itself**. Copied text carries nothing from Kimi.
- **Copy-paste survival**: the visible label disappears when the text leaves the platform; the metadata marker travels with files, not with copied text.
- **Statistical mark**: no official Kimi K3 documentation describes a model-level token-sampling mark. Not documented — *unknown, not absent* (see [DeepSeek — mitigation](../deepseek/mitigation.md) §3).
- **Open model**: Kimi K3/K2.6 weights are published; run locally they carry no provider mark (own sampler, [local models](../local-models.md)).
- **Regime**: Chinese framework for synthetic content identification (September 2025); Moonshot does not appear among the EU Code of Practice signatories.

## The difference with Claude

| | Kimi | Claude |
|---|---|---|
| Implicit identifier | Marker in **file metadata** | **Mark in the text itself** |
| Survives copy-paste | No | Yes |
| Survives conversion | No | Partially |

## Reading

Kimi is the case of **identification through metadata layers**: the provenance layer exists, but it is tied to the file and dies with conversion, re-saving or screenshots. It is exactly the kind of signal Claude's text mark tries to surpass — and the kind a mark-removal tool can clean deterministically, because metadata are bytes. Note that a platform can also add **Unicode carriers** on top of the text: a different layer, with different rules ([Unicode carriers](../unicode.md)).

## References

- Moonshot AI, *Kimi terms — AI-Generated Content Identification Rules* — [link](https://www.kimi.com/user/agreement/modelUse?version=v2)
- Kimi K3 — [GitHub MoonshotAI](https://github.com/MoonshotAI/Kimi-K3)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
