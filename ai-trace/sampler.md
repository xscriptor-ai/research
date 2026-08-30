---
title: "The sampler — what it is and what it consists of"
description: "The component that turns the model's probability distribution into a concrete token — and where the watermark lives."
date: "2026-08-14"
tags: ["technology", "probability", "cryptography"]
---

# The sampler

> *The model proposes; the sampler disposes. The mark does not live in what the model knows, but in how the sampler chooses among what it knows.*

This document explains, without smoke, the central component of the whole dossier: the **sampler**. It is where text watermarks live, and understanding it is understanding why the prompt cannot prevent them and why an own sampler is the only guarantee.

## 1. What the model does

An LLM does not "think in words": at each step it computes a **probability for every possible token of the vocabulary** given the previous context. With a vocabulary of $V$ tokens, the model returns a **probability distribution**:

$$
p(\text{"dog"} \mid \text{"the"}) = 0.31, \quad p(\text{"cat"}) = 0.27, \quad p(\text{"sun"}) = 0.22, \quad \dots
$$

That is: a huge list of candidates, each with its weight. **The model only proposes** — it does not decide which word comes out.

## 2. The problem: what to do with that list

If the highest-probability token were always chosen (*greedy*, $\arg\max$), the output would be **deterministic**: the same sentence every time, with a robotic, repetitive tone. For the text to be varied and natural, one must *roll the dice* among plausible candidates.

That "rolling the dice" is the **sampler**: the component that turns the distribution into a concrete token.

> **A distribution goes in; a token chosen randomly according to that distribution comes out.**

## 3. The sampler's knobs

To control *how much* randomness there is, the sampler adjusts the distribution before drawing:

- **Temperature $T$**: divides the logits by $T$. With low $T$, almost everything is greedy (deterministic); with high $T$ the distribution flattens (more surprise, more risk of errors).
- **Top-k**: draws only among the $k$ most probable candidates.
- **Top-p (nucleus)**: draws only within the subset accumulating a probability $p$.
- **Gumbel trick**: to draw without bias, each token receives a random value $u_w \in [0,1)$ and the choice is:

$$
x_t = \arg\max_{w}\left( \log p(w \mid x_{<t}) + G_w \right)
$$

That $u_w$ —a pseudorandom number— is the "face of the coin" of each candidate.

## 4. The property that makes it the mark's home

The sampler's randomness **is not chaotic: it is reproducible**. The sampler generates the $u_w$ with a pseudorandom generator seeded by the secret key $K$ and the context:

$$
u_w = h_K(w,\, x_{<t})
$$

**The same input always produces the same "dice".** The provider exploits this property in two ways:

1. **With distortion** (Kirchenbauer): biases the distribution before drawing (favors a "green list" of tokens), and the detector counts how many greens appear.
2. **Without distortion** (MirrorMark, SynthID-Text): does not touch the distribution, but *manipulates the dice* — e.g., reflects the value $u$ around a pivot:

$$
u' = (2\tau - u) \bmod 1
$$

The probability of each token does not change, but **the observed value encodes a bit**.

The detector, knowing $K$, **reproduces those same dice** and compares: if the observed values correlate with the key, there is a mark. Hence the rule running through the whole dossier:

$$
\text{mark present} \iff \text{the sampler uses the key } K
$$

## 5. The final intuition

Think of a casino roulette: the roulette (the sampler) picks the number. The watermark is a **casino that has subtly marked the wheel** — the ball does not land where the player wants, but the casino can tell afterwards whether the wheel was theirs, because it knows the secret geometry of that specific wheel.

- The **prompt** is the chip you put down: it does not change the wheel.
- The **agent** is the dealer who serves you: it does not change it either.
- You only change the wheel if you **buy it and put it on your own table**: local models with your own sampler.

## 6. Summary

| Question | Answer |
|----------|--------|
| What does the model propose? | A probability distribution over the vocabulary |
| What does the sampler do? | Chooses a concrete token according to that distribution |
| Why does it exist? | Without randomness, the output would be deterministic and robotic |
| Where does the mark live? | In the sampler's *reproducible* randomness (key $K$) |
| How is it detected? | Reproducing the dice with the key and comparing |

## References

- [Anthropic — 02 The technique](anthropic/02-the-technique.md) — the sampling formalism in Claude's context
- [Agents and prompts](agents-and-prompts.md) — why the prompt does not touch this layer
- [Common mitigation](mitigation.md) — how the correlation with the key is destroyed
- [Local models](local-models.md) — the own sampler as structural exemption
