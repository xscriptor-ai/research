---
title: "Agents and prompts — why they do not prevent the mark (and what does mitigate it)"
description: "The watermark is not avoided with instructions or an agent; it is mitigated by changing the sampler the agent accesses."
date: "2026-08-14"
tags: ["technology", "cryptography", "probability", "privacy"]
---

# Agents and prompts — why they do not prevent the mark

> *The mark is not decided in the prompt, nor even in the model: it is decided in the provider's sampler, with a key that neither the model nor the user knows. No instruction disables it; no agent prevents it. What can be done is change the sampler the agent accesses.*

This document answers the recurring question that arises when reading the dossier: *"can't I write a prompt, or build an agent, that prevents the text from coming out marked?"*. The short answer is **no** — and the explanation is structural, not a matter of detail.

## 1. The layers of generation

To see why the prompt cannot prevent the mark, separate what happens in each layer:

```
prompt (you)
   └─► model (predicts token probabilities)
           └─► sampler (chooses the token: key K + context)
                   └─► final text
```

| Layer | Who controls it | What it decides |
|-------|-----------------|-----------------|
| Prompt | You | What the user wants |
| Model | The provider | Which tokens are plausible ($p_\theta$) |
| **Sampler** | **The provider** | **Which concrete token materializes** |

The mark lives **in the third layer**: the sampler, not the model or the prompt.

## 2. Why a prompt cannot prevent the mark

Every text watermark in the dossier is applied **at sampling time**, with a secret key $K$, using reproducible randomness:

$$
u_w = h_K(w,\, x_{<t}), \qquad x_t = \arg\max_{w}\left( \log p_\theta(w \mid x_{<t}) + G_w \right)
$$

Three consequences follow that close the door on the prompt:

1. **The model does not know it is marked.** The mark is imperceptible to the model too; it cannot "decide" not to apply it.
2. **The sampler runs after the model.** Even if the model wrote the most "human" text possible, the sampler still keys the randomness of every choice.
3. **There is no magic phrase.** No instruction —*"do not mark this"*, *"write without a mark"*, *"act like a human"*— touches the layer where the signature is decided. Asking the model to avoid it is like asking it to erase an ink it cannot see.

**Conclusion**: the prompt operates on the model; the mark operates on the sampler. They are different layers, and the prompt does not reach the second one.

## 3. Why an agent does not prevent it either

An agent (Claude Code, a CLI, an assistant, this conversation) is a **wrapper** around the same sampler:

- it takes the user's prompt and turns it into model calls;
- but **every token of its responses still comes out of the provider's sampler**;
- the "deterministic randomness" the user intuits is exactly the mechanism: reproducible values $u_w$ seeded by key + context. **If the key belongs to the provider, the signature is present in everything the agent generates, without exception.**

An agent does not change the sampler: it only invokes it. That is why **building an agent is not mitigation** — at best, it is a way of *chaining* generations that still come from the same well.

## 4. Where the escape is: in the sampler, not in the prompt

Real mitigation has a single condition that can be stated in one line:

> **If the agent accesses a sampler you control, there is no mark. If it accesses a foreign sampler, no prompt can prevent it.**

The routes that work:

| Route | What it does | Guarantee |
|-------|--------------|-----------|
| **Own sampler (local open weights)** | The agent runs on a local model (DeepSeek, Llama, Qwen…) with your own engine | **Total** — no foreign key to correlate |
| **Unmarked provider** | Choosing a host/intermediary that does not mark | Not verifiable — you do not see the sampling layer |
| **Post-processing** | The agent rewrites the output with another sampler (local preferred), round-trip translation, or regeneration | High (degrades/breaks the signature) at a fidelity cost |
| **Pre-August-2026 models** | Using versions in the transition period | Temporary (until Dec 2, 2026) |

None of these routes goes through **writing a better prompt**. They go through **changing where the tokens come from**.

## 5. What an agent CAN do

An agent does not prevent the mark, but it can **choose the sampler** and **apply mitigation as a later step**:

1. **Choose the sampler**: if the agent is configured to run on local weights with its own engine, the exemption is structural — see [local models](local-models.md).
2. **Post-process the output**: rewrite the generated text with a second sampler (another local or unmarked model), or apply round-trip translation. This does not "prevent" the first sampler's signature: it **undoes** it afterwards, with the fidelity cost documented in [common mitigation](mitigation.md).

In other words: the agent acts **downstream** of the sampler, never upstream.

## 6. The concrete case of this conversation

One example is worth a thousand abstractions. If an agent serves `deepseek-v4-flash` through an **API** (as happens here), then:

- the sampler belongs to the infrastructure serving the model, **not to the user**;
- a prompt — neither the user's nor the agent's own — can disable what that infrastructure decides to apply;
- the absence of a documented mark is not a guarantee of real absence (see [DeepSeek — mitigation](deepseek/mitigation.md));
- the only guaranteed route would be **downloading the weights and running them locally**, so the agent accesses its own sampler.

## 7. Closing

The question *"can I avoid the mark with a prompt or an agent?"* confuses two layers that the dossier keeps separate: the **text** (where the prompt acts) and the **sampler** (where the signature lives). The mark is not prevented; it is dodged by changing where the tokens come from, or undone by re-sampling afterwards. **The agent is the key that opens access to the sampler — the problem is knowing whose lock it is.**

## References

- [Common mitigation](mitigation.md) — the general framework
- [Local models](local-models.md) — the structural route of an own sampler
- [Worked example](mitigation-example.md) — text before and after mitigation
- [DeepSeek — mitigation](deepseek/mitigation.md) — the API and intermediary blind spot
- [Anthropic — 02 The technique](anthropic/02-the-technique.md) — the sampling formalism
