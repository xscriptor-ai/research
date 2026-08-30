---
title: "Kimi — Specific mitigation"
description: "Kimi's signal is in metadata and the interface: deterministic stripping and open weights."
date: "2026-08-14"
tags: ["cryptography", "privacy", "technology"]
---

# Kimi — Specific mitigation

> *The general framework is in the root ([mitigation.md](../mitigation.md)). With Kimi, mitigation is among the simplest in the dossier: the signal is not in the text, but in the interface and the metadata.*

## 1. Explicit identifier (visible label)

It is an **interface** signal, not a content one:

- it appears as a label, mark or banner on the page;
- **it disappears when copying the text outside the platform**;
- Kimi's terms allow, in approved functions, removing it.

## 2. Implicit identifier (metadata)

It is a technical marker in the **file metadata**:

- removed with **deterministic stripping**: format conversion, re-saving, screenshot, or dedicated tools (`watermarks-remover` covers DOCX, ODT, PDF, HTML, Markdown, etc.);
- it is an exact erasure — the bytes are there or they are not —, not a statistical degradation;
- **it does not affect the text itself**, which never carried an embedded mark.

## 3. Local open model

Kimi K3 run locally:

- does not pass through the platform's labeling layer;
- no mark by design (own sampler) — the same structural exemption as DeepSeek.

## 4. Table

| Surface | Signal | Mitigation |
|---------|--------|------------|
| Text copied from the web | None in the content | Not applicable |
| Visible label | UI | Disappears when copying / approved removal |
| Implicit metadata | Bytes | Deterministic stripping |
| Local open weights | None | Not applicable |

## 5. Reading

Kimi is the case of **identification through shallow layers**: it complies with the rule without touching the text. For the evader, it is the cheapest scenario in the dossier — there is no statistical pattern to destroy, only bytes to erase or an interface to avoid.

## References

- [Common mitigation](../mitigation.md)
- Moonshot AI, *Kimi terms* — [link](https://www.kimi.com/user/agreement/modelUse?version=v2)
