---
title: "DeepSeek — Specific mitigation"
description: "In DeepSeek mitigation is barely needed: open weights do not carry a mark. But only when the sampler is yours."
date: "2026-08-14"
tags: ["cryptography", "privacy", "technology"]
---

# DeepSeek — Specific mitigation

> *The general framework is in the root ([mitigation.md](../mitigation.md)). With DeepSeek the mitigation question is inverted: it is not about erasing a mark, but about not passing through the layer that puts it. But beware: "no mark" is only a guarantee when the sampler is yours.*

## 1. The structural route: open weights

DeepSeek requires no mitigation in a single scenario: **open weights run locally**.

- **Open model + your inference engine** = own sampler = **no mark by design**. There is no signature to destroy because it was never generated.
- It is the direct application of the dossier's rule: *whoever controls the sampler decides*.

**This is the only surface where the absence of a mark is structural and verifiable by construction.** In all the others, a documented absence of a mark is not the same as a real absence.

## 2. The hosted platform and the API

- On the **hosted** service, the documented signal is a **visible label** (Chinese compliance): an interface label, not a text signature.
- On the **API**, the documented obligation is **disclosure** to the end user, not technical marking.
- The Chinese **implicit identifiers** are defined in **file metadata**: removed with deterministic stripping (conversion, re-saving).

## 3. The blind spot: API, intermediaries and agents

Here is the important caveat usually forgotten. On the **API, proxy or intermediary** routes, the sampler **is not yours**:

- When a developer calls DeepSeek's API —or when an agent/CLI serves `DeepSeek-V4` through a provider or router (as happens in this very conversation)— generation happens **in the infrastructure of whoever serves the model**, not on your machine.
- That provider **controls the sampling layer** and may or may not apply a mark without you knowing. The structural rule inverts: *you do not control the sampler, so you cannot guarantee the absence of a signature*.
- That DeepSeek's public documentation describes no statistical mark **does not prove there is none**: the absence of a document is not a guarantee, it is an unknown. And it can be added at any time without notice, because the sampler belongs to the provider.
- The same applies to the **intermediary**: a router that forwards to DeepSeek may —on its own or by obligation— add its own marking layer to the output.

**Practical conclusion**: with a DeepSeek model via API/intermediary you may have a signature and be unable to verify it. The only way to *know* there is no mark is to run the weights yourself.

## 4. Table

| Surface | Who controls the sampler? | Documented signal | Guaranteed absence of mark? |
|---------|---------------------------|-------------------|----------------------------|
| Local open weights | You | None | **Yes** (by construction) |
| Official API | DeepSeek | Disclosure (no mark) | Not verifiable |
| Intermediary / router / agent | The intermediary | Indeterminate | Not verifiable |
| Hosted platform (web) | DeepSeek | Visible label (UI) | No (the label does exist) |
| Implicit metadata (files) | DeepSeek | Bytes | Deterministic stripping |

## 5. Reading

DeepSeek is the case where mitigation costs "nothing" **only in one scenario**: local weights. In the rest of the ecosystem —and especially in the agents and intermediaries that serve the model without the user seeing the inference layer— the classification *"no mark"* becomes *"mark not documented"*, which is a different thing. The lesson extends to the whole dossier: **the exemption does not belong to the model, it belongs to the sampler**. If you do not control it, you are not exempt.

## References

- [Common mitigation](../mitigation.md)
- DeepSeek, *Open Platform Terms of Service* — [link](https://cdn.deepseek.com/policies/en-US/deepseek-open-platform-terms-of-service.html)
