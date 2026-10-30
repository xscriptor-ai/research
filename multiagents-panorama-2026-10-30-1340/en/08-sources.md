---
title: "Sources — what we know, how we know it and what is missing"
description: "Bibliography, evidence quality, claim table and the limits of this research."
date: "2026-10-30"
tags: ["technology", "sources", "research", "agents"]
---

# Sources — what we know, how we know it and what is missing

> *A number without a source is an opinion with decimals.*

## 1. How this dossier was built

- **Primary documentation** from Anthropic, OpenAI and OpenCode, read on **30 October 2026**.
- **List prices** from the official pages, in USD per million tokens, with the reading date.
- **Independent measurements** from third parties, labelled as such and treated as orders of magnitude, not benchmarks.
- **Local xscriptor repositories**, cloned on 30/10/2026, for the converter case study.
- **Our own calculations** from published rates; the formulas and assumptions are in documents 01 and 02.

The rule was: no cost claim without a token category and a price; no behaviour claim without a source or a local test.

## 2. Evidence quality

| Level | Type | Use in the dossier |
|---|---|---|
| **A** | Official vendor documentation | Features, fields, prices, limits |
| **B** | Vendor technical article with methodology | Multi-agent 15×, 90.2%, context rot |
| **C** | Independent measurement with a small sample | 12.4k–40.8k startup, quota anecdotes |
| **D** | Press/third parties | GA dates, OpenCode migration, comparisons |
| **E** | Our own calculation | Break-even, fan-out, cost per task |

A and B support the conclusions; C and D illustrate; E is reproducible with the formulas.

## 3. Primary sources

### Anthropic — Claude Code

1. **Create custom subagents.** https://code.claude.com/docs/en/sub-agents — fields, context loading, fork, depth, concurrency, models.
2. **Manage costs effectively.** https://code.claude.com/docs/en/costs — $13/day, $150–250/month, 7× teams, attribution, recommendations.
3. **How Claude Code uses prompt caching.** https://code.claude.com/docs/en/prompt-caching — prefixes, TTL, invalidation, subagents and fork.
4. **Orchestrate teams of Claude Code sessions.** https://code.claude.com/docs/en/agent-teams — teams, 7×, limits, messages.
5. **Extend Claude with skills.** https://code.claude.com/docs/en/skills — `SKILL.md`, frontmatter, `context: fork`, progressive loading.
6. **How Claude remembers your project.** https://code.claude.com/docs/en/memory — `CLAUDE.md`, `AGENTS.md`, reading order.
7. **Pricing.** https://platform.claude.com/docs/en/about-claude/pricing — price table, cache, batch, long context, tokenizer.

### Anthropic — research and practice

8. **How we built our multi-agent research system.** https://www.anthropic.com/research/multi-agent-research-system — 4×/15×, 80% of variance, 90.2%, limits in coding. Published 13/06/2025.
9. **Building agents: When to fold subagents into your main agent loop.** https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop — single agent + skills, 2× faster, when to keep subagents. 22/10/2026.
10. **How and when to use subagents in Claude Code.** https://claude.com/blog/subagents-in-claude-code — patterns, when not to delegate, “too many specialist agents”. 07/04/2026.

### OpenAI — Codex

11. **Subagents.** https://learn.chatgpt.com/docs/agent-configuration/subagents — roles, TOML, `[agents]`, inheritance, concurrency.
12. **Pricing.** https://learn.chatgpt.com/docs/pricing — plans, credits, rates, Fast/Ultrafast, limits.
13. **Import from another agent.** https://developers.openai.com/codex/import — import from Claude Code/Cursor, subagents, warnings.
14. **API pricing.** https://platform.openai.com/docs/pricing — per-token prices, cache, batch, long context.

### OpenCode

15. **Agents (v1).** https://opencode.ai/docs/agents/ — primary/subagent, `permission`, `mode`, `steps`, `hidden`, built-ins.
16. **Agents (v2).** https://opencode.ai/v2/docs/agents/ — `agents`, `permissions`, `system`, `steps`, `disabled`, built-ins.
17. **Rules.** https://opencode.ai/docs/rules/ — `AGENTS.md`, `CLAUDE.md` fallback in v1, precedence.
18. **Console pricing.** https://opencode.ai/docs/zen/ — model table, cache, Go, free models.

### Standards and ecosystem

19. **AGENTS.md.** https://agents.md/ — format, 60k+ projects, Agentic AI Foundation/Linux Foundation, compatible agents.
20. **Rulesync.** https://github.com/dyoshikawa/rulesync — conversion CLI, subagent support for Claude Code, Codex CLI, OpenCode and OpenCode V1.
21. **Cognition — Don't Build Multi-Agents.** https://www.cognition.ai/blog/dont-build-multi-agents — principles of shared context and implicit decisions. 12/06/2025.
22. **Chroma — Context Rot.** https://research.trychroma.com/context-rot — 18 models, length degradation, LongMemEval. 14/07/2025.

## 4. Secondary sources

23. **Tokenade / DEV — Claude Code Subagents Token Usage: What They Really Cost.** https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0 — 19 subagents, 12,427–40,763 startup tokens, 66k–84k per run (citing a July 2026 report), deduplication by `message.id`. 25/10/2026.
24. **Reveneau — Giving a Claude Code Subagent a Smaller Model.** https://reveneau.com/guides/claude-code-agent-costs/giving-a-subagent-a-smaller-model — price arithmetic, +30% tokenizer warning. 04/10/2026.
25. **Shattered — OpenCode 2 Upgrade: 12 Steps, 50 Min.** https://shattered.io/opencode-2-upgrade-from-1x-12-steps-2026/ — v1→v2 migration, permissions, AGENTS.md, plugins, v2 date. 27/10/2026.
26. **The Vibelog — OpenCode 2.0 vs Claude Code.** https://thevibelog.dev/blog/opencode-2-vs-claude-code-2026 — feature and cost comparison. 16/10/2026.
27. **Daniel Vaughan — Codex CLI in 2026.** https://codex.danielvaughan.com/2026/03/27/codex-cli-in-2026-whats-new/ — subagents GA in v0.115.0, 16/03/2026. 27/03/2026.

## 5. Local repositories

28. `xscriptor-ai/agents` — 205 agents, 142 with `edit: allow`, 105 with `task: allow`, 202 with `webfetch: allow`, ~17,768 characters of descriptions; converter at `claude/tools/xscriptor-convert.py`.
29. `xscriptor-ai/skills` — senior and web-fullstack skills; converter destination.
30. `xscriptor-ai/scripts` — `install-agents.sh`, installation into `~/.config/opencode/agents`.
31. `xscriptor-ai/research` — this dossier.

Counts were made with `find`/`grep` on the repositories cloned on 30/10/2026. The full converter was not executed because the environment did not have `PyYAML` installed; the analysis of its behaviour is based on reading the code.

## 6. Claim table

| Claim | Value | Source | Level |
|---|---|---|---|
| Agents vs chat | ~4× tokens | Anthropic 8 | B |
| Multi-agent vs chat | ~15× tokens | Anthropic 8 | B |
| Teams vs session | ~7× tokens | Anthropic 2, 4 | A |
| 90.2% multi-agent improvement | 90.2% | Anthropic 8 | B |
| Variance explained by tokens | 80% | Anthropic 8 | B |
| Subagent startup | 12,427–40,763 tokens | Tokenade 23 | C |
| Subagent run | 66k–84k tokens | Tokenade 23 (secondary) | C/D |
| Claude Code average cost | $13/day; $150–250/month | Anthropic 2 | A |
| Haiku 5.5 ≤100k price | $0.10/$0.50 | Anthropic 7 | A |
| Sonnet 5.5 price | $2.00/$10.00 | Anthropic 7 | A |
| Opus 5.5 price | $4.00/$20.00 | Anthropic 7 | A |
| Fable 5.1 price | $10.00/$50.00 | Anthropic 7 | A |
| GPT-6.1 Sol price | $2.00/$10.00 | OpenAI 14 | A |
| GPT-6 Luna price | $0.10/$0.50 | OpenAI 14 | A |
| Codex Sol credits | 50/2.5/250 | OpenAI 12 | A |
| Codex subagents GA | 16/03/2026 | Vaughan 27 | D |
| Claude reads AGENTS.md | Yes, if no CLAUDE.md | Anthropic 6 | A |
| OpenCode v2 only AGENTS.md | Yes | Shattered 25 | D |
| Rulesync supports subagents | Claude/Codex/OpenCode | Rulesync 20 | A |
| Codex imports subagents | Yes | OpenAI 13 | A |
| xscriptor 205 agents | 205 | Local 28 | A |
| xscriptor descriptions | ~17,768 chars | Local 28 | A |
| xscriptor converter | OpenCode v1 → Claude | Local 28 | A |

## 7. What we could not verify

- **Cost per subagent in Codex.** There is no public view equivalent to Claude Code transcripts; analysis is by thread and model.
- **Exact `CLAUDE.md` behaviour in OpenCode v2.** v1 documentation supports it as a fallback; the v2 migration guide says only `AGENTS.md` is discovered. The official v2 rules page was not available on the reading date. Verify on your version.
- **Actual loading of all 205 xscriptor agents in Claude Code.** The converter requires `PyYAML`; it was not run end to end. The permission critique is based on reading the code.
- **Subscription quota to dollars.** Not published; any figure that presents it as dollars is an estimate.
- **Long-term quality effects.** There is no large randomised trial; Anthropic's data is internal and Chroma's is laboratory work.
- **Future prices.** The tables are a snapshot of 30/10/2026; models and prices change.
- **The 66k–84k figure.** It is secondary (Tokenade citing a July 2026 report); use it as an order of magnitude.

## 8. How to cite this dossier

```bibtex
@techreport{xscriptor2026multiagents,
  title   = {Multiagents Panorama: The price and speed of delegating},
  author  = {Xscriptor},
  year    = {2026},
  month   = {October},
  note    = {Research dossier, xscriptor-ai/research},
  url     = {https://github.com/xscriptor-ai/research}
}
```

## 9. Closing

The evidence is enough for a conclusion and not enough for a law. We know that subagents isolate context, that they cost startup, that the model weighs more than isolation and that conversion loses semantics. We do not know how much of that translates to your repository without measuring it.

That ignorance is the honest part of the dossier. The other part is the formula.

---

**Back:** [Index](README.md) · [Executive summary](economics/00-executive-summary.md)
