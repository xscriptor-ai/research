---
title: "DeepSeek — Marking methods"
description: "Labeling under the Chinese rule, without a documented statistical mark."
date: "2026-08-14"
tags: ["technology", "privacy", "research"]
---

# DeepSeek — Marking methods

> *DeepSeek complies with a different rule, in a different way: it labels content on its platform and hands over unmarked weights to whoever wants to run them.*

## 1. The regime that obliges it

DeepSeek operates under Chinese regulation: the *Measures for the Identification of AI-Generated Synthetic Content* (in force since September 2025), which distinguish between **visible identification** and **less visible technical identification**.

## 2. What it does on the hosted platform

- Adds **identifiers** of AI-generated content (visible labeling) to comply with the Chinese rules.
- Its terms forbid users from removing, altering, hiding or bypassing those identifiers.
- On the **API** (Open Platform), it obliges developers to **inform end users** that the output is AI-generated — a *disclosure* obligation, not a mark.

## 3. What it does not do

- **Does not document** a hidden statistical mark at the token level in the style of Claude/SynthID that survives copy-paste in ordinary DeepSeek V4 text.
- Does not appear among the European Code of Practice signatories; it complies through its own route.

## 4. The open-weights layer

DeepSeek V4 and **V4 Flash** are distributed as **open weights** (Hugging Face). The layer separation is the key:

```
Hosted DeepSeek:   model → engine → server → interface (visible label) → export
Open weights:      model → YOUR engine → YOUR interface → (no label, no mark)
```

Whoever runs the weights locally controls the **sampler** and the interface; the platform's labeling layer does not participate. **The open model does not produce a mark by itself.**

## 5. The API and intermediary caveat

The absence of a mark from the previous point **is only guaranteed when the sampler is yours** (local weights). On the **API, proxy, router or agent** routes —like a CLI serving `DeepSeek-V4` from a provider— generation happens in the infrastructure of whoever serves the model, and that provider controls the sampling layer:

- it may (or may not) apply a mark without the user knowing;
- DeepSeek's public documentation describes no statistical text mark, but **the absence of a document is not a guarantee of the absence of a mark**;
- an intermediary may add its own marking layer to the output.

The structural exemption belongs to the **sampler**, not the model: whoever runs the weights decides; whoever calls an API does not.

## 6. Reading

DeepSeek is the **structural exemption** of the ecosystem: the model a user can download and run without passing through the provider's layer. It does not need to "remove" a mark because, in local use, **there is no mark to remove**. But that reading is restricted to local weights: on the API or intermediary route, the sampler belongs to the provider and the absence of a mark becomes a non-verifiable unknown.

## References

- DeepSeek, *Open Platform Terms of Service* — [link](https://cdn.deepseek.com/policies/en-US/deepseek-open-platform-terms-of-service.html)
- DeepSeek V4 Flash — [Hugging Face](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash)
- TechGrapple, *Claude Text Watermark vs OpenAI, Kimi K3 & DeepSeek V4* — [link](https://www.techgrapple.com/claude-text-watermark-explained-why-users-are-upset-and-how-openai-kimi-k3-deepseek-v4-compare/)
