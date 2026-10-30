---
title: "Acceleration and latency — the other side of the invoice"
description: "What a subagent actually accelerates, what it only parallelises, and why 1→2→4 can be slower than a single thread."
date: "2026-10-30"
tags: ["technology", "agents", "performance", "latency"]
---

# Acceleration and latency — the other side of the invoice

> *Parallelising is not accelerating: it is distributing the wait. If the wait is in integration, distributing it does not remove it.*

## 1. Three metrics that are not the same

| Metric | Definition | What it measures |
|---|---|---|
| **Latency** | Time from request to accepted result | The user's wait |
| **Throughput** | Accepted tasks per unit time | The system's capacity |
| **Cost of speed** | Tokens (or quota) per unit of time saved | What you pay to go faster |

A subagent can improve one task's latency and worsen the project's throughput; it can accelerate exploration and slow integration. The right question is not “is it faster?”, but **“which part of the critical path got shorter?”**.

## 2. Amdahl's law, without escape

If a fraction \(p\) of the work is parallelisable and the rest is sequential, the speedup with \(N\) workers is:

$$
S(N) = \frac{1}{(1-p) + p/N}
$$

| Parallel fraction \(p\) | N = 2 | N = 4 | N = 8 | Theoretical limit |
|---:|---:|---:|---:|---:|
| 0.50 | 1.33× | 1.60× | 1.78× | 2.00× |
| 0.70 | 1.54× | 2.11× | 2.58× | 3.33× |
| 0.90 | 1.82× | 3.08× | 4.71× | 10.00× |
| 0.95 | 1.90× | 3.48× | 5.93× | 20.00× |

The consequence for 1→2→4: **the 8 leaves do not accelerate what is not parallelisable**. If integration, contracts, final tests or the build consume 30% of the time, the maximum is 3.3× even with infinite agents. If they consume 50%, the maximum is 2×. Fan-out does not break Amdahl; it hides it until integration arrives.

## 3. Coordination also takes time

A more realistic model:

$$
T(N) = T_s + \frac{T_p}{N} + cN
$$

where \(T_s\) is sequential work, \(T_p\) parallel work, and \(c\) the cost of coordinating each agent (messages, waits, conflicts, review). The optimum is not “all the agents”, but:

$$
N^* = \sqrt{\frac{T_p}{c}}
$$

Example: \(T_s=1\) h, \(T_p=8\) h, \(c=0.25\) h per agent. \(N^* \approx 5.7\); with 6 agents, \(T \approx 3.8\) h versus 9 h for one thread: **2.4×**. If coordination rises to \(c=1\) h, \(N^* \approx 2.8\) and speedup falls to **1.35×**. More agents, less acceleration.

That is the mathematical argument against 1→2→4 by default: **11 agents are not 11 times faster; they are 11 more coordination points**.

## 4. The evidence

| Source | What it measures | Result | Reading |
|---|---|---|---|
| Anthropic, Research (2025) | Multi-agent research | Up to **90% less time** on complex queries | Real parallelism accelerates exploration |
| Anthropic, Research (2025) | Cost of the same system | ~15× tokens vs chat | Speed is paid for |
| Anthropic, Applied AI (2026) | Single loop vs multi-agent | **~2× faster** with one agent | Fewer agents can be faster |
| Claude Code costs (2026) | Agent teams | ~7× tokens vs standard session | Coordination has an invoice |
| Public anecdote (2026) | 5 parallel subagents | ~15 min vs ~30 min sequential | 2× speed, more quota |
| This dossier | 1→2→4 with 8 leaves | Limited by slowest leaf + integration | The tree does not remove the critical path |

The first two rows explain why multi-agent is attractive for research: the work is breadth-first, there are many independent branches and the result is a summary. The next two explain why it does not translate automatically to coding: the work is sequential, shares files and integration is the bottleneck.

## 5. The speed tax

If fan-out multiplies tokens by \(M\) and accelerates by \(S\), the cost per unit of speed is:

$$
\text{tax} = \frac{M}{S}
$$

| Scenario | M | S | Tax | Reading |
|---|---:|---:|---:|---|
| 5 subagents, 30k startup, 100k work | 2.5× | 2.0× | 1.25× | You pay 25% more tokens per unit of speed |
| Agent team in plan mode | 7.0× | 2.0× | 3.5× | Speed is expensive |
| 1→2→4 with 8 leaves | ~3–4× | 2.0× | 1.5–2.0× | Depends on integration |
| Fast model (Codex) | 2.0× quota | <2.0× latency | >1.0× | Quota rises faster than speed |

The tax does not say “don't do it”; it says **how much you are paying for the clock**. If the clock is worth more than the quota, it is rational. If quota is the scarce resource, fan-out is a luxury.

## 6. Where each tool actually accelerates

| Tool | Mechanism | Accelerates well | Accelerates badly |
|---|---|---|---|
| **Claude Code** | Subagents, fork, background, teams | Exploration, review, parallel tests | Same-file edits, coupled tasks |
| **Codex** | Agent threads, `max_concurrent_threads_per_session` | Exploration, review lenses, tests | Write fan-out, merges |
| **OpenCode v2** | Child sessions, background, `steps` | Independent work, exploration | Deep nesting, shared-state tasks |

Claude Code has fork: it shares the parent cache and accelerates without paying the full startup. Codex inherits sandbox and permissions, which reduces configuration but not coordination. OpenCode v2 allows background, but its `general` does not nest: 1→2→4 must be built with custom agents and permissions, and coordination cost rises.

## 7. 1→2→4: when it accelerates and when it does not

| Project | Accelerates? | Why |
|---|---|---|
| **Greenfield webapp** | **Yes, contract-first** | Frontend/backend and modules are disjoint; 8 leaves can work on different files |
| **Existing webapp** | **Partial** | Shared contracts and files; integration dominates |
| **UI/UX design** | **Partial** | 4 leaves inventing styles produce inconsistency; one design-system owner is better |
| **Linux distro: research** | **Yes** | Packages, licences, compatibility, artwork and docs are independent branches |
| **Linux distro: build** | **No** | PKGBUILD, kernel, initramfs, installer, signatures and ISO share state; integration is serial |
| **Debugging with hypotheses** | **Yes** | Independent competing branches; the team contrasts them |
| **Coupled refactor** | **No** | Each change depends on the previous one; the summary loses detail |

The practical rule: **1→2→4 accelerates when the 8 leaves write to different files and integration is cheap**. If they share files, contracts or build state, the tree is a conflict machine.

## 8. How to measure it

- **Latency**: time to accepted result, not to first token.
- **Throughput**: accepted tasks per hour, not tasks launched.
- **Integration time**: merges, conflicts, final tests, review.
- **Coordination time**: messages, waits, retries, idle agents.
- **Tax**: \(M/S\), tokens per unit of speed.
- **Quality**: if speed rises and acceptance falls, there is no real acceleration.

A minimal experiment: same task, same branch, 3 repetitions; measure \(T_1\), \(T_N\), tokens and acceptance. If \(S(N) < 1.2\) and the tax is >1.5, fan-out is not buying speed; it is buying activity.

## 9. Closing

Acceleration is real, but local: it lives in independent branches, not in the whole tree. A 1→2→4 can halve exploration and double integration time; the net result depends on which weighs more on the critical path.

The question is not “how many agents can I launch in parallel?” but **“which leaf is the slowest, and how long does integration take?”**. If the answer is integration, more agents only distribute the wait.

---

**Next:** [Quality and project performance](01-quality-and-project-performance.md) — what happens to quality, reliability and delivery when you accelerate.
