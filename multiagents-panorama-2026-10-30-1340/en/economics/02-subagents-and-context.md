---
title: "Subagents and context — the saving that is not in the token"
description: "What a subagent isolates, what it costs, when isolation pays, and why the single agent with skills wins again in many cases."
date: "2026-10-30"
tags: ["technology", "agents", "context", "economics"]
---

# Subagents and context — the saving that is not in the token

> *A subagent does not reduce the work: it reduces what the work leaves in memory. The invoice arrives from both sides.*

## 1. What a subagent is, in cost terms

A subagent is three things at once:

1. **A separate context**: its own window, system prompt and tools. It does not see the parent conversation; it receives a delegation message.
2. **A separate loop**: several model requests, with its own file reads, tool calls and results. Every request resends its growing context.
3. **A return summary**: when it finishes, it returns text to the parent. That text enters the main context and is paid on every subsequent turn.

That separation has one benefit and three costs:

| Benefit | Cost |
|---|---|
| Noisy context does not contaminate the main thread | **Startup**: prompt, `CLAUDE.md`/`AGENTS.md`, git status, tools, skills |
| A different model can be used, cheaper or more specialised | **Work**: the subagent runs its own loop and pays its tokens |
| Work can be parallelised | **Return**: the summary enters the parent and is resent |
| Permissions and tools can be isolated | **Coordination**: choose, launch, wait, synthesise and verify |

The usual mistake is to count only the benefit (“the context is not in the parent”) and forget the three costs. The second mistake is to count only the cost (“subagents use more tokens”) and forget that the parent, without them, would also have paid to read, compact and carry that context.

## 2. The saving condition

Let:

- \(D\): tokens the subagent keeps out of the main thread.
- \(T\): turns remaining in the main session after delegation.
- \(r\): fraction of the input price that a cache read costs (0.05–0.1).
- \(S\): subagent startup tokens.
- \(W\): subagent work tokens (input).
- \(O\): subagent output tokens.
- \(R\): summary tokens returned to the parent.
- \(c_w\): cache-write multiplier (1.25× or 2×).

The net saving is:

$$
\Delta = D \cdot T \cdot r \cdot p_{\text{in,main}}
\;-\;
\left[ (c_w S + W) \cdot p_{\text{in,sub}} + O \cdot p_{\text{out,sub}} + R \cdot p_{\text{in,main}} \right]
$$

If \(\Delta > 0\), the subagent saves **API dollars**. If \(\Delta < 0\), it can still be rational for quality, latency or quota, but not for money. The calculation treats \(W\) as input processed once; a subagent that re-reads its context pays more, and one that caches it pays less.

### 2.1. The table that matters

With \(D=100k\), \(S=30k\), \(W=100k\), \(O=10k\), \(R=2k\), 5-minute cache:

| Main → subagent | 10 turns | 25 turns | 50 turns | 100 turns |
|---|---:|---:|---:|---:|
| Opus 5.5 → Opus 5.5 | −$0.56 | −$0.26 | +$0.24 | +$1.24 |
| Opus 5.5 → Sonnet 5.5 | −$0.18 | **+$0.12** | +$0.62 | +$1.62 |
| Opus 5.5 → Haiku 5.5 (>100k) | **+$0.10** | +$0.40 | +$0.90 | +$1.90 |
| Sonnet 5.5 → Sonnet 5.5 | −$0.28 | −$0.13 | +$0.12 | +$0.62 |
| Sonnet 5.5 → Haiku 5.5 (>100k) | +$0.002 | +$0.15 | +$0.40 | +$0.90 |

Three readings:

1. **With the same model, isolation takes time to pay.** At a 5% cache read, removing 100k tokens from the parent saves $0.01 per turn on Sonnet; the subagent costs $0.38. It takes ~38 turns.
2. **Routing changes the answer.** Opus main + Sonnet subagent pays from ~19 turns; Opus + Haiku pays almost from the start, even with the 100k threshold.
3. **The bad case is an expensive subagent under a cheap main model.** Sonnet → Opus loses money at almost every horizon: you pay for the expensive model to save cheap cache.

### 2.2. What the formula does not capture

- **Avoided compaction**: if the parent context crosses the threshold, the session summarises and loses detail. The subagent can prevent that loss. The cost of compaction is a large request plus the risk of forgetting.
- **Quality**: a clean context can produce a better answer; a shared context can produce a more coherent one. The formula does not decide.
- **Latency**: parallelism reduces wall-clock time even if it increases tokens.
- **Quota**: on a subscription, the marginal dollar cost is zero until the window runs out; the scarce resource is the window.
- **Risk**: a subagent with broad permissions can do more damage; one with narrow permissions may not be able to finish.

## 3. The fan-out multiplier

If you launch \(N\) subagents in parallel, with a total workload \(W\) split between them and a startup \(S\) per agent, and the parent synthesises, token cost grows roughly by:

$$
M_{\text{fan-out}} \approx 1 + \frac{N \cdot S}{W}
$$

where \(W\) is the **total** workload, not the work per agent. With \(N=5\), \(W=100k\) and \(S=30k\), each agent does 20k of work and pays 30k of startup: \(1 + 5 \cdot 30/100 = 2.5\times\).

| N | S = 15k, W = 100k | S = 30k, W = 100k | S = 30k, W = 500k |
|---:|---:|---:|---:|
| 2 | 1.30× | 1.60× | 1.12× |
| 5 | 1.75× | 2.50× | 1.30× |
| 10 | 2.50× | 4.00× | 1.60× |

The formula is an approximation: it ignores summary, output and coordination, and assumes work is split without duplication. Even so, it shows the order of magnitude: **fan-out is cheap when the work per agent is large and startup is small; it is expensive when each agent does little**.

The real case that circulates — five parallel subagents hitting the Pro limit in about 15 minutes, versus ~30 minutes sequentially — is consistent with this arithmetic: five startups, five loops and five contexts compete for the same window ([DEV/Tokenade](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0), secondary anecdote). It is not a benchmark, but it points in the right direction.

## 4. The industry numbers

### 4.1. 4×, 15× and 7×

| Source | What it measures | Multiplier | Comment |
|---|---|---|---|
| Anthropic, Research (2025) | Agents vs chat | **~4×** | A tool-using agent |
| Anthropic, Research (2025) | Multi-agent vs chat | **~15×** | Research architecture |
| Anthropic, Claude Code (2026) | Agent teams vs standard session | **~7×** | Teammates in plan mode |
| This dossier | Fan-out of 5 with 30k startup and 100k work | **2.5×** | Formula \(1+NS/W\) |

They are not contradictory: they measure different architectures. A single subagent does not cost 15×; a research system with a lead and many subagents does. An agent team with independent contexts and plan mode, 7×. The right question is **which multiplier matches your topology**, not which number is universal.

### 4.2. The quality data

In Anthropic's internal evaluation, a multi-agent system with Opus 4 as lead and Sonnet 4 as subagents outperformed a single Opus 4 by **90.2%** on research tasks. The analysis found that token usage alone explained **80%** of the variance on BrowseComp, with tool calls and model choice as the other factors ([Anthropic, 2025](https://www.anthropic.com/research/multi-agent-research-system)).

That result is the strongest argument for multi-agent: **more tokens buy more capability on broad exploration tasks**. But the same article warns that most coding tasks have less real parallelism and that agents are still not good at coordinating in real time. The conclusion is not “multi-agent for everything”, but “multi-agent for what looks like research, not for what looks like editing the same file”.

### 4.3. The counterargument: the single agent with skills

In October 2026, Anthropic's Applied AI team published an important correction: across several enterprise deployments, **a single agent with skills outperformed the subagent architecture on quality, and often at lower cost and latency**. One customer saw a runtime **~2× faster** with the single loop. The reason: shared context lets the model connect decisions, and progressive disclosure of skills and tools avoids loading everything at once. The article recommends folding subagents into the main loop and keeping them only when:

- the single loop degrades measurably from context bloat;
- tasks are truly parallel with disposable contexts;
- the agent struggles to choose tools or there are conflicting instructions.

([Anthropic, “When to fold subagents into your main agent loop”, 22/10/2026](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop))

That does not contradict subagents: it places them. A subagent is an **isolation** tool, not a default architecture. When the problem is “too much noisy context”, the subagent helps. When the problem is “the decisions of one step must inform the next”, the subagent gets in the way.

### 4.4. Context rot: why clean context matters

Chroma evaluated 18 models and found that performance degrades non-uniformly as input grows, even on simple tasks. On LongMemEval, full prompts of ~113k tokens performed much worse than “focused” prompts of ~300 tokens with the same relevant information; the effect appeared across families ([Chroma, Context Rot, 2025](https://research.trychroma.com/context-rot)). The conclusion: **it is not enough for information to be in the window; how much noise surrounds it matters**.

That favours isolation when the subagent can receive a small, relevant context. But it also explains why a badly designed subagent — vague prompt, blind exploration — can produce a worse summary than the parent. The quality of the input context is the deciding variable.

## 5. The decision matrix

| Situation | Recommended architecture | Why |
|---|---|---|
| Search 50 files and return 5 findings | **Exploration subagent** | Disposable context, small output, cheap model |
| Edit three coupled files | **Main thread** | Shared decisions; a summary would lose detail |
| Review a PR with fresh eyes | **Reviewer subagent** | Clean context, read-only permissions |
| Implement a small change | **Main thread** | Startup costs more than the task |
| Research a broad topic in parallel | **Several subagents / team** | Real parallelism, independent contexts |
| Debug with competing hypotheses | **Agent team** | Discussion and contrast; 7× tokens |
| Long task with noisy tool output | **Subagent + summary** | Avoids context rot and compaction |
| Task depends on 20 turns of decisions | **Main thread or fork** | The fork preserves context and cache |
| Need the same context but isolated | **Fork** | Shares cache; cheaper than a fresh subagent |
| Need different permissions | **Subagent with tools/permissions** | Capability isolation |

## 6. Fork, subagent and team

| | **Fresh subagent** | **Fork** | **Agent team** |
|---|---|---|---|
| Context | New, only the delegation prompt | Copy of the parent | New, independent |
| Cache | Own, starts cold | Shares the parent's | Own |
| Model | Configurable | The parent's | Configurable per teammate |
| Communication | Returns a summary | Returns a result | Peer messages and task list |
| Cost | Startup + work | Work + cache reuse | Startup + work + coordination |
| Best for | Disposable work | Same task, same context | Discussion, review, divided ownership |

The fork is the underrated option: it shares the prompt, tools and conversation, so its first request reads the parent cache. For a side task that needs the accumulated context, it is often cheaper than a fresh subagent. Its limit: it carries all the context, including the noise.

## 7. How to decide in practice

A four-question sequence:

1. **Is the context the task or the noise?** If the subagent needs the history to decide, do not isolate it; use the main thread or a fork. If the history is noise, isolate.
2. **Is the work independent?** If two subagents touch the same file or depend on each other's result, coordination eats the saving.
3. **Is there a cheaper model that can do it?** If yes, routing is the lever; isolation is secondary.
4. **How many turns remain?** If the session ends in five turns, startup does not amortise. If fifty remain, isolated context matters.

If all four answers are “yes”, delegate. If any is “no”, measure before delegating.

## 8. Closing

The subagent is a technology of attention, not of discount. It buys the ability to look at a problem without dragging everything before it; it charges for creating a new world and for translating what it found into a summary. Like every translation, it loses something. The question is whether what it loses matters more than the noise it avoids.

When the answer is no, the subagent saves. When it is yes, the subagent only makes the problem look tidier.

---

**Next:** [Acceleration and latency](../performance/00-acceleration-and-latency.md) — what fan-out really accelerates and what integration slows down.
