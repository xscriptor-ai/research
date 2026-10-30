---
title: "Performance — acceleration, latency and quality"
description: "Index of the performance section: what fan-out accelerates, what integration slows down, and what happens to quality."
date: "2026-10-30"
tags: ["performance", "latency", "quality", "agents"]
---

# Performance — acceleration, latency and quality

This section answers how much it accelerates and what quality it leaves behind. The [economics section](../economics/README.md) answers how much it costs.

| Document | Content |
|---|---|
| [00 — Acceleration and latency](00-acceleration-and-latency.md) | Amdahl, coordination, speed tax and 1→2→4 viability |
| [01 — Quality and project performance](01-quality-and-project-performance.md) | Quality, reliability, delivery, maintainability, webapp vs distro |
| [02 — Cases and variables](02-cases-and-comparisons.md) | Languages, application types, documentation, papers and decision matrix |

## The idea in one line

Fan-out accelerates only independent branches. If integration is 30% of the work, the maximum is 3.3× even with infinite agents; if it is 50%, the maximum is 2×. More agents do not break Amdahl: they distribute the wait.

**Next:** [Economics](../economics/README.md) · [Main index](../README.md)
