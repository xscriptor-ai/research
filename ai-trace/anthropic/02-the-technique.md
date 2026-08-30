---
title: "The technique — how a text watermark works"
description: "The sampling moment, the statistical trace, and Claude's dual technique (water + C2PA)."
date: "2026-08-14"
tags: ["technology", "cryptography", "probability"]
---

# 02 — The technique

> *The mark is not added after the text: it is born with it. It is not an ink applied to the page, but a bias that inhabits the very instant the page is written.*

## 1. The core idea

A text watermark pursues a seemingly contradictory goal: **hide a signal that travels with the text** without altering the text the reader perceives. The resolution of the apparent dilemma is that the signal does not live in the *words* but in the **randomness that chose the words**.

A language model does not always choose the most probable word. While generating, it samples: over the probability distribution $p_\theta(\cdot \mid x_{<t})$ that assigns a probability to each token of the vocabulary $V$ given the previous context, decoding introduces a random component (temperature, *top-p*, Gumbel-top-k, etc.). That randomness is the **available space** to hide information.

## 2. The sampling formalism

Let the model's output at step $t$ be:

$$
x_t \sim p_\theta(\,\cdot\,\mid x_{<t})
$$

Using the Gumbel trick, sampling can be reinterpreted as deterministic plus noise: each token $w \in V$ receives a value $u_w \in [0,1)$ generated reproducibly from a secret key $K$ and the context:

$$
u_w = h_K(w,\, x_{<t})
$$

and the token maximizing a combined quantity is chosen:

$$
x_t = \arg\max_{w \in V} \left( \log p_\theta(w \mid x_{<t}) + G_w \right), \quad G_w \sim \text{Gumbel}(0,1)
$$

The values $u_w$ (or their *ranks*) are the **detector-reproducible pseudorandom object**: the detector, knowing $K$, can *replay* the sampling decisions and compare them with what was observed.

## 3. Two historical families

### 3.1. Distribution-biased marks (distortion)

The classic family, inaugurated by **Kirchenbauer et al. (2023)**, splits the vocabulary into two lists —"green" and "red"— using a pseudorandom generator seeded by the last $k$ tokens. At each step:

$$
\ell'(w) =
\begin{cases}
\ell(w) + \delta & \text{if } w \in \text{green list}\\
\ell(w) & \text{if } w \in \text{red list}
\end{cases}
$$

where $\ell(w)$ are the logits. The model shows *preference* for the green list, and the detector counts how many tokens fall into it. Under human text, the green proportion should be $\approx \gamma$ (the fixed green fraction); under marked text, it exceeds $\gamma$. The statistical test is the classic $z$:

$$
z = \frac{n_g - \gamma\, n}{\sqrt{n\, \gamma\, (1-\gamma)}}
$$

**Cost**: the bias alters the real output distribution; there is an unavoidable loss of quality, and an adversary can *detect that there is a mark* by observing the distortion.

### 3.2. Distortion-free marks

The family that **preserves the distribution by construction**. Instead of biasing the logits, it manipulates the **sampling randomness** (the $u_w$) so that the marginal distribution of the chosen token is still $p_\theta$. A transformation that achieves this is **mod-1 reflection**: given a pivot $\tau$ and a symbol $m$, the value $u$ is reflected around $\tau$:

$$
u' = (2\tau - u) \bmod 1
$$

Since $\tau$ and $1-\tau$ pair uniformly, the distribution of $u'$ is identical to that of $u$: **the probability of each token does not change**. But the observed value $u'$ —which the detector can reproduce— encodes $m$ through its distance to $\tau$. The text looks statistically identical to normal generation; the mark lives in the sampling *residuals*.

## 4. Claude's dual technique

Anthropic describes **two complementary techniques**:

### 4.1. Watermark embedded in the text

When a supported model generates text, it *weaves* an imperceptible watermark **directly into the text itself**. Declared properties:

- it cannot be perceived at a glance;
- it does not change the *meaning*, *quality* or *readability*;
- it is applied **at the model level**, so it is present regardless of which product or surface the text comes from;
- since it is part of the text, **it travels with copy-and-paste** and **may survive some editing**.

The decisive phrase is *"applied at the model level"*: the mark is introduced **during generation**, not in a later processing stage. That rules out *post-processing* techniques (adding invisible characters, reordering, etc.) and points to the families in section 3, in particular the **distortion-free** ones.

### 4.2. C2PA provenance metadata

For generated files —`.svg`, `.png`, `.jpg`— Claude attaches **digitally signed provenance metadata**, following the **C2PA** open standard (*Coalition for Content Provenance and Authenticity*), the same one the industry uses to record content provenance.

The C2PA signature is not invisible: it is a metadata field. Its function is different and complementary:

- it signals that the file was **processed by Claude**;
- it allows detecting whether the file has been **tampered with** (the signature breaks if the content changes);
- it is lost with format conversion, re-saving, screenshots or other means that strip metadata.

## 5. Why text is the hard case

Watermarks in images or audio can lean on perceptual redundancy (the eye does not notice a minimal luminance change). Text is discrete and sparse: there are no "pixels" in which to hide something; every token is both *signifier and signified*. That is why the text mark has to live in the **only non-visible place**: the randomness of choice. And that is why marked text is, in a sense, *text carrying the trace of its own production mechanism*: you cannot look at it without the machine that knows how to read it knowing that another machine wrote it.

## 6. Technical summary

| Property | Text mark | C2PA |
|-----------|-----------|------|
| Where it is applied | At generation (token sampling) | On the finished file |
| Signal | Statistical (hidden pattern) | Cryptographic (signature) |
| Visible | No | No (metadata) |
| Survives copy-paste | Yes | No (only if format preserves it) |
| Survives light editing | Partially | Yes (with re-signing) |
| Survives heavy editing | No | No |
| Detection | Statistical (with key) | Signature verification |

## References

- Kirchenbauer et al., *A Watermark for Large Language Models* (2023) — [arXiv:2301.10226](https://arxiv.org/abs/2301.10226)
- Aaronson & Kirchner, *Watermarking GPT outputs* (2023) — [link](https://www.scottaaronson.com/blog/?p=8012)
- Anthropic Support, *How Claude marks AI-generated content* — [link](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
