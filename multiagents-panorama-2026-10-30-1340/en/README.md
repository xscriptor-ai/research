---
title: "Multiagents Panorama — The price and speed of delegating"
description: "Dossier on the economics, acceleration and project performance of agents and subagents in Claude Code, Codex and OpenCode."
date: "2026-10-30"
tags: ["technology", "agents", "economics", "performance", "research"]
---

# Multiagents Panorama — The price and speed of delegating

> *No subagent is free: it pays a context bill before it reads the first line. The question is not whether it delegates, but which part of the work deserves to exist twice — and which part is worth waiting for.*

**English** · [Español](../es/README.md)

Research dossier on **agents and subagents in Claude Code, OpenAI Codex and OpenCode**, organised around two axes:

- **Economics** — tokens, cache, model routing, break-even and cost per accepted task.
- **Performance** — acceleration, latency, coordination, quality and project delivery.

Prices, model names and product behaviour were verified against primary documentation on **30 October 2026**. Token prices change; the formulas do not.

## The short answer

1. **Subagents are not a discount coupon.** They are a context-isolation and model-routing mechanism. They reduce the *cost per accepted task* only when the isolated work is large, disposable and independent enough to repay its startup, its own loop and the summary returned to the parent.
2. **Total tokens usually go up.** Anthropic measured agents at ~4× the tokens of chat and multi-agent research at ~15×; Claude Code agent teams at ~7× a standard session. A fresh Claude Code subagent was measured writing **12,427–40,763 tokens** to its cache before doing any work.
3. **The big financial lever is the model, not the delegation.** The same 100k-input / 10k-output job costs **$0.015 on Haiku 5.5**, **$0.30 on Sonnet 5.5**, **$0.60 on Opus 5.5** and **$1.50 on Fable 5.1**. A cheap subagent can save 20–40× per token; context isolation alone saves only the cached-input rate, often 5–10% of that.
4. **Acceleration is local.** Fan-out accelerates only independent branches. If integration is 30% of the work, Amdahl caps the speedup at 3.3× even with infinite agents; if it is 50%, the cap is 2×. More agents distribute the wait; they do not remove it.
5. **Quality does not scale with the number of agents.** Anthropic's own Applied AI team found that a single agent with skills often beat the subagent architecture on quality and cost; subagents remain for isolation, parallelism and permissions.
6. **Agnostic conversion is possible at the structural layer.** `AGENTS.md`, `SKILL.md` and MCP are common ground. Agent files are not: Claude Code uses Markdown + YAML, Codex TOML, OpenCode Markdown + YAML with another permission model. A converter reaches **capability parity**; **behavioural and economic parity** need an IR, a manifest and an evaluation set.

## Key numbers

| Claim | Number | Source |
|---|---|---|
| Agents vs chat | ~4× tokens | Anthropic, multi-agent research system (2025) |
| Multi-agent vs chat | ~15× tokens | Same |
| Claude Code agent teams vs standard session | ~7× tokens (plan mode) | Claude Code cost docs (2026) |
| Fresh Claude Code subagent startup | 12,427–40,763 cache-write tokens | Independent transcript measurement (2026) |
| Model routing, same job | 40× between Haiku 5.5 and Opus 5.5 | Anthropic list prices (2026) |
| Research time with parallel agents | Up to **90% less** on complex queries | Anthropic multi-agent research (2025) |
| Single loop vs multi-agent harness | **~2× faster** in one enterprise case | Anthropic Applied AI (2026) |
| xscriptor agent library | 205 agents, ~17,768 chars of descriptions | Local `agents` repo (2026) |
| Claude Code average spend | ~$13/developer/active day; $150–250/month | Anthropic cost docs (2026) |

## Sections

| Section | English | Español |
|---|---|---|
| **Economics** | [Index](economics/README.md) | [Índice](../es/economia/README.md) |
| **Performance** | [Index](performance/README.md) | [Índice](../es/rendimiento/README.md) |
| **Tools** | [Claude Code](03-claude-code.md) · [Codex](04-codex.md) · [OpenCode](05-opencode.md) | [Claude Code](../es/03-claude-code.md) · [Codex](../es/04-codex.md) · [OpenCode](../es/05-opencode.md) |
| **Conversion** | [Agnostic conversion](06-agnostic-conversion.md) | [Conversión agnóstica](../es/06-conversion-agnostica.md) |
| **Playbook** | [Measurement](07-measurement-and-playbook.md) | [Medición](../es/07-medicion-y-playbook.md) |
| **Sources** | [Sources](08-sources.md) | [Fuentes](../es/08-fuentes.md) |

## Documents

| # | English | Español |
|---|---|---|
| 00 | [Executive summary](economics/00-executive-summary.md) | [Resumen ejecutivo](../es/economia/00-resumen-ejecutivo.md) |
| 01 | [Token accounting](economics/01-token-accounting.md) | [Contabilidad de tokens](../es/economia/01-contabilidad-de-tokens.md) |
| 02 | [Subagents and context](economics/02-subagents-and-context.md) | [Subagentes y contexto](../es/economia/02-subagentes-y-contexto.md) |
| 03 | [Acceleration and latency](performance/00-acceleration-and-latency.md) | [Aceleración y latencia](../es/rendimiento/00-aceleracion-y-latencia.md) |
| 04 | [Quality and project performance](performance/01-quality-and-project-performance.md) | [Calidad y rendimiento del proyecto](../es/rendimiento/01-calidad-y-rendimiento-del-proyecto.md) |
| 05 | [Cases and variables](performance/02-cases-and-comparisons.md) | [Casos y variables](../es/rendimiento/02-casos-y-comparativas.md) |
| 06 | [Claude Code](03-claude-code.md) | [Claude Code](../es/03-claude-code.md) |
| 07 | [Codex](04-codex.md) | [Codex](../es/04-codex.md) |
| 08 | [OpenCode](05-opencode.md) | [OpenCode](../es/05-opencode.md) |
| 09 | [Agnostic conversion](06-agnostic-conversion.md) | [Conversión agnóstica](../es/06-conversion-agnostica.md) |
| 10 | [Measurement and playbook](07-measurement-and-playbook.md) | [Medición y playbook](../es/07-medicion-y-playbook.md) |
| 11 | [Sources](08-sources.md) | [Fuentes](../es/08-fuentes.md) |

**Data:** [model prices](../data/prices-2026-10-30.csv) · [delegation scenarios](../data/delegation-scenarios.csv) · [notes](../data/README.md).

## How to read it

- Ten minutes: **00** and the decision matrix in **02**.
- If cost matters: **01** and **02**.
- If speed or quality matters: **03**, **04** and **05**.
- Configuring one tool: **06**, **07** or **08**.
- Writing the converter: **09**.
- Measuring or defending the change: **10**.
- Auditing a number: **11**.

## Method

- **Primary sources first**: vendor documentation and pricing pages, read on 30 October 2026.
- **Secondary sources labelled**: independent measurements and press coverage are marked as such; no anecdote is presented as a benchmark.
- **Formulas over folklore**: every cost or speed claim is derived from token categories, Amdahl's law or measured runtime, with assumptions stated.
- **Reproducible**: the measurement protocol and the conversion parity levels are specified so the reader can run the same experiment on their own repository.

This is a research dossier, not a buying recommendation. The correct architecture depends on the value of the task, the cost of a wrong answer, the length of the session and the critical path of the project — the variables the token bill does not show.

## License

MIT. Text and tables may be reused with attribution.
