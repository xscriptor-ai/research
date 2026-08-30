---
title: "Local models — guaranteed no signature"
description: "Open-weight model families run locally and why, in that scenario, they carry no watermark."
date: "2026-08-14"
tags: ["technology", "privacy", "research"]
---

# Local models — guaranteed no signature

> *The guarantee does not belong to the model: it belongs to the sampler. Any open-weight model you run on your machine, with your own inference engine, carries no mark — not because the mark is "not applied to it", but because the sampler is yours and the mark lives in the sampler.*

## 1. The principle (read before the list)

The text watermark is not **in the weights** of the model. The dossier's schemes (Kirchenbauer, Aaronson–Kirchner, MCmark, MirrorMark) are applied **at sampling time**, in the decoding layer, using a secret key. The weights are inert: they do not carry the signature inside.

From this follows the rule:

$$
\text{mark present} \iff \text{the sampler uses the key } K
$$

**Consequence**: if you download the weights of any open-weight model and run them with your own engine (without enabling any marking scheme), **there is no key to correlate** and, by construction, **there is no signature**. The guarantee is structural, not a vendor promise.

**The caveat that invalidates everything**: if instead of running the weights you access the model through an **API, proxy, router, or agent** (as in this conversation), the sampler belongs to whoever serves the model, and the absence of a mark stops being a guarantee — see [DeepSeek — mitigation](deepseek/mitigation.md). **This list only counts if you actually run the weights locally.**

## 2. Available open-weight families (August 2026)

Models with public downloadable weights, verified against the 2026 landscape:

| Family | Company / lab | Reference models | Notes |
|---------|----------------|------------------|-------|
| **DeepSeek** | DeepSeek | V4, V4 Flash, V4 Pro, R1 (reasoning) | Weights on Hugging Face; among the best in agentic coding. |
| **Llama** | Meta | Llama 4 (Scout…), Llama 5 | General purpose; large quantization ecosystem. |
| **Qwen** | Alibaba | Qwen 3.5, Qwen 3.6 (e.g. 27B) | Very good in the small/medium range; coding and general use. |
| **Gemma** | Google | Gemma 4 (e.g. 9B) | Open weights with vision capabilities; edge-oriented. |
| **Mistral** | Mistral AI | Medium 3.5, Large 3, *maestro*/*mod* range | Several sizes and flexible licenses. |
| **Kimi** | Moonshot AI | K2.6, K3 | Open weights; K2.6 leads neutral indexes among open models. |
| **Phi** | Microsoft | Phi-4 | Small, efficient, reasoning-oriented. |
| **GLM** | Zhipu / Z.AI | GLM-5.1 | MIT license (the most permissive of the group). |

Any of these families, **run locally**, meets the condition: own sampler = no mark. The choice between them depends on hardware, task (code, reasoning, vision) and license — not on the watermark, which in this scenario does not exist for any of them.

## 3. Local inference engines

To run these weights with a sampler under your control:

| Engine | Notes |
|--------|-------|
| **llama.cpp** | C++, CPU and GPU, GGUF quantization; the de facto standard for local. |
| **Ollama** | Friendly wrapper over llama.cpp; command-line model management. |
| **vLLM** | Built for servers and production; high throughput. |
| **Hugging Face Transformers** | Python pipelines; maximum control over sampling. |
| **GPT4All** | Simple desktop alternative. |

**Practical rule**: use the engine's **default sampler, without post-processing layers or decoding hooks**. If you do not enable any marking scheme, there is no mark. (Some example repos include watermarking code: just do not use it.)

### Do tools like LM Studio come with a keyless sampler?

**Yes.** LM Studio (and by extension Ollama, llama.cpp, vLLM and Transformers pipelines) ship **their own built-in sampler** — temperature, top-k, top-p, min-p, seed — and **by default it uses no marking key**. The reason is structural and holds for all of them:

- the mark does not live **inside the Hugging Face weights**; the weights are inert;
- the sampler belongs to the **engine**, not to the model;
- the providers' marks are added at their **serving layer**, which does not exist here.

When you download a model from HF and run it in LM Studio, the sampler is the app's: **no key, no tracking signature**. One nuance: if the downloaded repository includes *example code* with watermarking, do not run it — but that is using third-party code, not the engine.

## 4. How to deploy your own sampler (generalized)

There is no need to follow a click, but a **pattern of five decisions**. The concrete steps depend on the chosen tool; the pattern is always the same:

1. **Choose open-weight weights.** From a model registry (Hugging Face or another): family, size and license based on hardware and task. The weights carry no mark — choose by quality and compatibility, not by "marking".
2. **Choose an inference engine.** LM Studio, Ollama, llama.cpp, vLLM or Transformers. **The sampler already ships inside the engine**: it is not downloaded separately, it is not installed apart. This is the core idea: *the sampler is a property of the engine, not of the model*.
3. **Load the weights into the engine.** The engine downloads (or is pointed at) the weights from the registry, usually in quantized form (GGUF/Q4) to fit the VRAM.
4. **Configure the sampler.** Engine-exposed parameters: temperature, top-k, top-p, min-p, seed. The engine's default **is already keyless**; touching these parameters only changes generation style, not the presence of a signature (which remains null).
5. **Verify there is no marking layer.** With an open engine this is trivial: check that sampling uses no key. For total control, write your own sampler (a *LogitsProcessor* in Transformers, or replace the sampling module in llama.cpp) — an exercise of dozens of lines, not a project.

The full pattern:

```
weights (HF, no mark) ──► engine with its own sampler (no key) ──► text without a trace
```

**Direct answer to "how do I create/download it?"**: you do not create the sampler — you *choose weights* and *choose an engine that ships its own sampler*. The keyless sampler is not something you obtain separately: it is what ships by default in local tools, and you only have to make sure you do not activate any marking layer a third party left in the example code.

## 5. What this list guarantees and what it does not

| Scenario | Guaranteed absence of signature? | Why |
|-----------|----------------------------------|-----|
| Open weights + own local engine | **Yes** | The sampler is yours; there is no key to apply. |
| Open weights + Ollama/llama.cpp local | **Yes** | Same principle: inference happens on your machine. |
| Open weights via a host API | **No** | The host controls the sampling layer (may or may not mark). |
| Open weights via router/agent/proxy | **Not verifiable** | An intermediary may add its own marking layer. |
| Proprietary models (Claude, GPT, Gemini) | **Not applicable** | No downloadable weights; API-only = foreign sampler. |

## 6. Honest nuances

1. **No watermark is not no trace.** A local model leaves no *watermark signature*, but it can leave **other signals**: the model's own statistical style (which "AI writing" detectors analyze) or the quantization footprint. These are not keyed marks; they are predictive patterns, and they differ from what this dossier covers.
2. **The weights are not "the guarantee"; the sampler is.** If tomorrow a vendor published weights *alongside* a marking scheme enabled by default, the guarantee would still live in not activating it. The rule does not depend on the model's name.
3. **Local verification is its own reward**: running locally, you can inspect the engine, the sampler and the code — impossible with an API. The guarantee is, in the end, the guarantee of *control*.

## 7. Reading

This list is the reverse of the dossier: where Anthropic, Google and —when they decide— OpenAI embed signatures in the sampler, here the sampler belongs to you. It is the **structural exemption** that recurs in [DeepSeek](deepseek/README.md) and [Kimi](kimi/README.md), extended to the whole open-weight ecosystem. It is not a way of "removing" the signature: it is the way of *not having to remove it* — because it was never generated.

> **Note**: the document describes technical capabilities of local execution. It does not recommend breaking rules; it simply documents where, by design, there is no signal to detect.

## References

- Codersera, *Open-Source LLMs Landscape 2026* — [link](https://codersera.com/blog/open-source-llms-landscape-2026/)
- Inkeybit, *The Local LLM Model Guide 2026* — [link](https://www.inkeybit.com/blog/local-llm-model-guide-2026)
- [Common mitigation](mitigation.md) — the general framework of which this document is the structural route
- [DeepSeek — mitigation](deepseek/mitigation.md) — the API and intermediary blind spot

### Verification (August 2026)

- LM Studio, *Configuring the model* (official parameters: Temperature, Top-K, Top-P, Min-P, Repeat Penalty, XTC, Seed — **no watermarking parameters**) — [link](https://lmstudio.ai/docs/typescript/llm-prediction/parameters) · guide based on official docs: [note.com](https://note.com/takkunblue/n/n302d933bf316?hl=en)
- **vLLM-Watermark** — a *separate* and optional package to add marks to vLLM ("Tiny. Hackable... built on vLLM"); confirms watermarking does not ship in vLLM by default — [link](https://vermaapurv.com/vLLM-Watermark/index.html)
- **MarkLLM** — LLM watermarking toolkit (Tsinghua University), a layer that is added, not default — [GitHub](https://github.com/THU-BPM/MarkLLM)
- Engine licenses (llama.cpp, Ollama, Jan, vLLM, SGLang, mlx-lm: MIT/Apache 2.0) — [dev.to](https://dev.to/sreeraj-sreenivasan/the-complete-guide-to-local-llm-inference-tools-in-july-2026-llamacpp-ollama-vllm-sglang-and-4mh1)
