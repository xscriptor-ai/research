---
title: "Regulatory context — the AI Act and Article 50"
description: "The legal framework that obliges Anthropic to mark AI-generated content."
date: "2026-08-14"
tags: ["legislation", "privacy", "technology"]
---

# 01 — Regulatory context

> *Laws are not written for the technology that exists, but for the one imagined by those who draft them. The AI Act imagined a world in which synthetic content could be identified; now the whole industry is rushing to build that world.*

## 1. The European AI Act

The **Artificial Intelligence Act** (Regulation 2024/1689) was formally adopted in 2024 and constitutes the world's first comprehensive regulatory framework for artificial intelligence. It regulates providers according to the **risk** their systems present, with proportionate obligations.

For the topic at hand, the central provision is **Article 50**, on *transparency obligations*. Paragraph (2) requires providers of generative AI systems to ensure that their outputs — text, image, audio, video — can be **identified as generated or manipulated by AI** through machine-readable marking.

## 2. The Code of Practice on Transparency

The operational detail is not in the Regulation itself but in the **Code of Practice on Transparency of AI-Generated Content** (the *Transparency Code*), tied to Article 50(2). It is designed to:

- mark AI-generated or edited content in a way **other systems** can identify (not the human eye);
- cover text, images, audio and video;
- require providers to **cooperate with third parties** on detection.

The Code entered into force on **August 2, 2026**. Signatories include Anthropic, OpenAI, Google, Meta, Microsoft, Black Forest Labs and Synthesia, among others.

## 3. Consequences of non-compliance

Non-compliance is penalized with fines of up to **€15 million** or **3 % of the provider's global annual turnover**, whichever is greater. For Anthropic, the fine would be significant but, more than the money, what is at stake is **authorization to operate in the European market**.

## 4. The timeline

| Date | Milestone |
|-------|-----------|
| 2024 | Formal adoption of the AI Act. |
| August 2, 2026 | The Transparency Code enters into force. Models launched on or after this date must mark content **from day one**. |
| December 2, 2026 | End of the transition period for previously launched models. |
| August 11, 2026 | Anthropic publicly confirms the marking and publishes the technical support page. |

## 5. Anthropic's commitments

On the page *How Claude marks AI-generated content*, Anthropic makes concrete commitments:

1. **New models mark from day one.** Models launched in the EU on or after August 2, 2026 include machine-readable marking at launch: generated text carries an **embedded watermark**, and generated files carry **digitally signed provenance metadata**.
2. **The marking works on every surface.** It applies to output from supported models across the Claude Platform (API), Claude, Claude Code, Claude Cowork and Claude Tag, and through cloud partners (AWS, Google Cloud, Microsoft Foundry). **Worldwide**, not just in the EU.
3. **They will facilitate detection.** Anthropic commits to letting users and third parties detect its marks, as the Code requires, and to publishing the corresponding technical documentation.
4. **Earlier models in progress.** The law includes a transition period for models launched before August 2, 2026, and Anthropic is working to add marking to those models as well.

## 6. Why the scope is global

Although the trigger is European regulation, Anthropic will apply the marking **everywhere**. The reason is engineering: the marking is implemented **at the model level** (during inference), not in the interface or per region. Maintaining two behaviors — marked for the EU, unmarked for the rest — would double deployment complexity and create a trivial route for evasion. The chosen design is coherent: **a single model version, marked for everyone.**

## 7. Critical reading

The asymmetry deserves attention. The European rule pursues *transparency*: being able to know when content is synthetic. But the watermark is **one-sided** — only those who have the key can detect it — which turns it more into an **authorship attribution mechanism** than a mechanism for *informing the reader*. The reader sees nothing. Who detects is a machine, for whatever purposes the detector chooses. This is the tension running through the whole dossier: **the same mechanism that informs some, identifies others.**

## References

- European Commission, *Guidelines on AI transparency obligations* — [link](https://digital-strategy.ec.europa.eu/en/policies/guidelines-ai-transparency-obligations)
- Regulation (EU) 2024/1689, Article 50 — [link](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
- Anthropic Support, *How Claude marks AI-generated content* — [link](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
