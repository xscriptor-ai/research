---
title: "Measurement and playbook — how to know the saving is real"
description: "Metrics, instrumentation, experiment design and decision rules for delegating without self-deception."
date: "2026-10-30"
tags: ["technology", "agents", "measurement", "economics"]
---

# Measurement and playbook — how to know the saving is real

> *The token counts itself; the saving has to be proven. The difference between the two is method.*

## 1. Why anecdotes contradict each other

In the same week, two opposite stories appear:

- “Delegating grunt work to subagents cut my bill by 40%.”
- “Five parallel subagents burned through my Pro limit in 15 minutes.”

Both can be true. The first usually measures **one long session** where the subagent avoided carrying a huge context; the second, **a fan-out** where five startups competed for the same window. The difference is not the tool: it is the topology, the session length and the metric.

Without method, the debate is settled by the anecdote that confirms prior intuition. With method, it is settled by a table.

## 2. The metrics

| Metric | Definition | Why it matters |
|---|---|---|
| \(C_{\text{run}}\) | Cost of one run | The immediate invoice |
| \(P_{\text{success}}\) | Accepted tasks / attempted tasks | Reliability |
| \(C_{\text{accepted}}\) | \(C_{\text{run}} / P_{\text{success}}\) | The real cost per outcome |
| \(Q\) | Rubric score (0–100) | Quality |
| \(T_{\text{wall}}\) | Time to result | Human cost |
| \(M\) | Delegated tokens / direct tokens | Token multiplier |
| \(H\) | Cache hit rate | Context efficiency |
| \(R\) | Rework (human minutes) | The cost not on the invoice |
| \(E\) | \(Q / C_{\text{accepted}}\) | Economic efficiency |

The decision metric is \(C_{\text{accepted}}\), not \(C_{\text{run}}\). A $1.50 agent with 90% success ($1.67) can be better than a $0.80 agent with 60% ($1.33) if the second generates 20 minutes of review. At $60/hour, 20 minutes is $20: the “cheap” one is 12× more expensive.

## 3. Instrumentation

### 3.1. Claude Code

- **`/usage`**: session cost, usage per model, attribution to skills/subagents/plugins/MCP on subscription plans, cache line.
- **Transcripts**: `~/.claude/projects/<project>/<session>/` and `subagents/agent-<id>.jsonl`. Each assistant message carries `usage` with `input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`.
- **Trap**: streaming writes several lines with the same `message.id` and the same `usage`. **Deduplicate by `message.id`** or you will triple the count.
- **OpenTelemetry**: exports tokens and cost per user/session; the only option that reaches your own stack in real time.
- **`/context`**: what occupies the window; useful for seeing the weight of `CLAUDE.md`, tools and skills.

Minimal command to read one subagent:

```bash
for f in ~/.claude/projects/*/SESSION_ID/subagents/agent-*.jsonl; do
  jq -s -c --arg f "$(basename "$f")" '
    map(select(.message.usage)) | unique_by(.message.id) | map(.message.usage)
    | {agent: $f, requests: length,
       input: (map(.input_tokens)|add),
       cache_write: (map(.cache_creation_input_tokens)|add),
       cache_read: (map(.cache_read_input_tokens)|add),
       output: (map(.output_tokens)|add)}' "$f"
done
```

### 3.2. Codex

- **`/status`** in the CLI: model and remaining usage.
- **Dashboard**: `chatgpt.com/codex/settings/usage`.
- **API**: organization logs and the response; cost is calculated with the price table.
- **Subagents**: the app shows threads and summaries; there is no public per-subagent attribution equivalent to Claude Code. For fine analysis, instrument the provider or the gateway.

### 3.3. OpenCode

- **Session cost** in the TUI; `opencode stats` aggregates tokens and estimated cost.
- **Console**: workspace and member budgets; auto-recharge.
- **Child sessions**: cost is aggregated into the session; per-subagent attribution requires export or instrumentation.
- **Free models**: cost is not zero if the price is your data; the economic metric must include the data policy.

### 3.4. Common denominator

To compare across tools, normalise:

- **Same model or same tier** (not Opus vs Luna without saying so).
- **Same effort/reasoning**.
- **Same system prompt and tools**.
- **Same task set and acceptance criterion**.
- **Same time window** (quota changes over time).

## 4. Experiment design

### 4.1. Task set

20–30 representative tasks, stratified by type:

| Type | Examples | Why |
|---|---|---|
| Exploration | “Where is the token validated?” | Disposable context |
| Logs/tests | “Which tests fail and why?” | Voluminous output |
| Review | “Review this diff” | Clean context |
| Small implementation | “Fix this typo” | Should stay in the parent |
| Coupled implementation | “Change this interface in three modules” | Context matters |
| Research | “Compare these two libraries” | Parallelism |

Each task needs:

- a **verifiable expected result** (test, diff, file list, rubric);
- a **binary acceptance criterion** or a 0–100 rubric;
- a **reference cost** (direct run);
- a **timeout** and a **maximum budget**.

### 4.2. Paired design

For each task, run:

- **A**: main thread, no delegation.
- **B**: with subagent(s), same task and same main model.
- **C** (optional): with a cheap subagent (Haiku/Luna) and an expensive main model.

Repeat each cell 3 times, with the same model and effort, and randomise the order to avoid cache warm-up effects. If possible, use two repositories or two branches to avoid contamination.

### 4.3. Analysis

- **Median and interquartile range**, not mean: agent costs have heavy tails.
- **Paired bootstrap** for the confidence interval of the cost and success difference.
- **Report effect size**: “B costs 1.8× tokens and succeeds 8 points more” is more useful than a p-value.
- **Split by task type**: the global average hides that delegation wins on exploration and loses on editing.
- **Include human cost**: review minutes × hourly rate.

### 4.4. Sample size

With 20 tasks you can detect large effects (≥30% difference) if the design is paired. For 10–15% effects, you need 50–100 tasks or more repetitions. Do not conclude with 3 tasks and an anecdote.

## 5. Decision rules

### 5.1. Delegate if…

- the work produces **a lot of disposable output** (searches, logs, tests);
- the work is **independent** and can be parallelised;
- you need **clean context** to review or verify;
- there is a **cheaper model** that can do it;
- the main session has **many turns ahead**;
- the cost of a mistake is low or the subagent has narrow permissions.

### 5.2. Do not delegate if…

- the task is **small** (less than ~30k tokens of work);
- the work is **sequential** and each step depends on the previous one;
- two agents would touch **the same file**;
- the subagent needs **the whole history** to decide;
- **startup** costs more than the task;
- you cannot **verify** the result.

### 5.3. Practical thresholds

| Signal | Threshold | Action |
|---|---|---|
| Expected output | >50k tokens | Delegate to an explorer |
| Subagent work | <30k tokens | Keep in the parent |
| Remaining turns | <20 | Do not delegate for savings |
| Model difference | ≥2× | Route |
| Model difference | ≥20× | Route even if startup hurts |
| Independent tasks | ≥3 | Parallelise, with a cap |
| Shared files | ≥1 | Do not parallelise writes |

The thresholds are heuristics, not laws. Use them to decide what to measure, not to replace measurement.

## 6. Playbook by scenario

| Scenario | Architecture | Model | Why |
|---|---|---|---|
| “Where is the auth logic?” | Explorer subagent | Fast | Large output, small answer |
| “Why does this test fail?” | Triage subagent | Fast/Balanced | Voluminous logs, short summary |
| “Review this PR” | Reviewer subagent | Balanced/Frontier | Clean context; judgement matters |
| “Implement this small feature” | Main thread | Balanced | Startup > task |
| “Refactor three coupled modules” | Main thread or fork | Frontier | Shared decisions |
| “Compare three libraries” | 3 parallel subagents | Fast/Balanced | Real parallelism |
| “Research a broad topic” | Team or multi-agent | Frontier + Balanced | Breadth-first; 7–15× tokens |
| “Fix all type errors” | 2–3 subagents per package | Balanced | Independent per file |
| “Security review before commit” | Read-only subagent | Frontier | High risk; clean context |
| “Document this module” | Main thread + skill | Balanced | Shared context |

## 7. Anti-patterns

1. **Agent sprawl.** 205 agents installed, 205 descriptions loaded, 205 ways to choose badly. Prune.
2. **Write fan-out.** Five agents editing the same repo: conflicts, merges and rework.
3. **“Give me a full report.”** The report enters the parent and is paid on every turn. Ask for lists, paths, findings.
4. **A subagent for a typo.** Startup costs more than the fix.
5. **Haiku for a 130k prompt.** The 100k threshold multiplies the price by 5.
6. **Switching models mid-task.** Invalidates the cache; you pay the re-read.
7. **No `maxTurns`/`steps`.** A subagent can explore until the window is exhausted.
8. **Measuring total tokens and calling it savings.** The total can rise while cost per accepted task falls.
9. **Ignoring quota.** On a subscription, the scarce resource is the window, not the dollar.
10. **Converting without a manifest.** An agent that loads but widens permissions is not a conversion; it is a risk.

## 8. A 30-day plan

### Week 1 — baseline

- Install only 8–12 relevant agents.
- Measure 10 tasks without delegation: cost, time, success.
- Record the quota consumed per session.
- Measure the size of `CLAUDE.md`/`AGENTS.md` and the descriptions.

### Week 2 — one delegation

- Pick the most voluminous, disposable work (search or logs).
- Create a subagent with a cheap model, `omitClaudeMd`/minimal tools and bounded output.
- Compare A/B on 10 tasks.
- Adjust the delegation prompt first, not the model.

### Week 3 — routing

- Identify tasks that can use a model 2–20× cheaper.
- Create two or three agents with `fast`/`balanced` tiers.
- Measure cost per accepted task, not just tokens.
- Check quota: does fan-out exhaust it?

### Week 4 — prune and convert

- Remove agents you did not use.
- Measure L1/L2 of the converter (loading and capabilities).
- If you convert, emit a manifest and review agents that widen permissions.
- Document the three tasks that will stay in the main thread.

## 9. Closing

Measuring is how you distinguish saving from movement. A subagent can reduce the invoice, the latency, the window or the risk; rarely all four at once. Method lets you choose which of the four you are buying.

The final question is not “do I delegate or not?”. It is “what am I willing to pay not to remember this?”. If the answer is clear, the architecture follows.

---

**Next:** [Sources](08-sources.md) — references, evidence quality and what we could not verify.
