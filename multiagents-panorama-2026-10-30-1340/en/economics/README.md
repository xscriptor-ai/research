---
title: "Economics — the cost of delegating"
description: "Index of the economics section: tokens, prices, cache, break-even and context."
date: "2026-10-30"
tags: ["economics", "tokens", "agents"]
---

# Economics — the cost of delegating

This section answers how much it costs and when it saves. The [performance section](../performance/README.md) answers how much it accelerates and what quality it leaves behind.

| Document | Content |
|---|---|
| [00 — Executive summary](00-executive-summary.md) | The thesis, key numbers and seven decisions |
| [01 — Token accounting](01-token-accounting.md) | Prices, cache, formulas and worked examples |
| [02 — Subagents and context](02-subagents-and-context.md) | Isolation, break-even, fan-out and limits |

## The idea in one line

A subagent does not save by delegating: it saves context and, above all, it allows a cheaper model. With the same model, isolating 100k tokens takes ~38–50 turns to pay; with a cheap subagent, the break-even drops to a few turns.

**Next:** [Performance](../performance/README.md) · [Main index](../README.md)
