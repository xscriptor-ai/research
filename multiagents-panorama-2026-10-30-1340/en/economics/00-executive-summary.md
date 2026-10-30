---
title: "Executive summary — do agents and subagents actually save money?"
description: "The short answer, the arithmetic behind it, and the seven decisions that follow."
date: "2026-10-30"
tags: ["technology", "agents", "economics", "tokens"]
---

# Executive summary — do agents and subagents actually save money?

> *Delegating is not dividing the work: it is deciding which context deserves to survive. The rest is accounting.*

## 1. The short answer

**Yes, they can save; no, they do not save just by being agents.**

An agent or subagent saves when at least one of these is true:

1. **It isolates disposable context** that would otherwise be billed on every subsequent turn and would push the session toward compaction.
2. **It routes the work to a cheaper model** that is good enough for that subtask.
3. **It reduces failed attempts** because the task runs with a clean context, narrower permissions or an independent review.

It does not save when the task is small, sequential, dependent on accumulated context, or requires two agents to make coupled decisions. In those cases the subagent pays a startup context, runs its own loop and returns a summary: three new costs to save a context that was already cached.

The sentence that summarises the dossier:

$$
\text{saving} = \underbrace{\text{context removed from the main thread} \times \text{remaining turns} \times \text{cache rate}}_{\text{what is avoided}}
\;-\;
\underbrace{(\text{startup} + \text{work} + \text{summary} + \text{coordination})}_{\text{what is paid}}
$$

With a warm cache, the read rate is **5–10%** of the input price. That is the detail that breaks the intuition: removing 100,000 tokens from the main thread does not save 100,000 tokens per turn; it saves **5,000–10,000 token-equivalents** per turn. Delegation only pays if many turns remain, the work is very large, or the subagent uses a cheaper model.

## 2. The three markets of cost

The question “does it reduce cost per task?” has three different answers depending on the currency:

| Currency | What is paid | How it is measured | Who feels it |
|---|---|---|---|
| **API dollars** | Input, cache and output tokens | Provider invoice | Company with an API key or cloud |
| **Subscription quota** | Usage window (5 h, weekly, monthly) | `/usage`, `/status`, dashboard | Claude Pro/Max; ChatGPT Plus/Pro; OpenCode Go |
| **Human time** | Developer minutes, review, rework | Clock and labour cost | Any team |

A subagent can be **more expensive in dollars** and **cheaper in human time**. It can also be **cheaper in dollars** and **more expensive in quota**, because five parallel subagents burn the same window five times faster. And it can be **cheaper in quota** if the work moves to a model included in the plan or to a local model.

The scale-setting number: Claude Code costs, across enterprise deployments, **~$13 per developer per active day**, **$150–250 per month**, and 90% of users stay below **$30 per active day** ([Anthropic, Claude Code costs](https://code.claude.com/docs/en/costs)). Compared with the cost of an engineering hour, the token is cheap; what is expensive is being wrong, reviewing and repeating. That is why cost per accepted task — not cost per million tokens — is the right metric.

## 3. What the three tools do

| | **Claude Code** | **Codex** | **OpenCode** |
|---|---|---|---|
| **Subagents** | Yes; `.claude/agents/*.md` | Yes; TOML in `~/.codex/agents/` or `.codex/agents/` | Yes; Markdown in `~/.config/opencode/agents/` or `.opencode/agents/` |
| **Automatic delegation** | By `description` | When asked, or when `AGENTS.md`/skill asks | By `description`, or `@mention` |
| **Nesting** | Yes, 3 layers by default; configurable | Yes; the orchestrator can spawn new subagents | v2: controlled by permissions; built-in `general` cannot nest |
| **Model per agent** | Yes (`haiku`, `sonnet`, `opus`, `fable`, ID, `inherit`) | Yes (`model`, `model_reasoning_effort`) | Yes (`provider/model#variant`) |
| **Isolated context** | Yes; fork shares cache, the rest do not | Yes; own thread | Yes; child session |
| **Background** | Yes | Yes | Yes (v2) |
| **Cost per agent** | Visible in `/usage` and transcripts | `/status`, dashboard, logs | Session cost; not always per subagent |
| **Practical ceiling** | 20 concurrent subagents; 15k description tokens | `max_concurrent_threads_per_session` | Permissions and `steps` |

All three answer **yes** to “can an agent delegate to a subagent?”. The difference is semantics: Claude Code and Codex can nest; OpenCode v2 lets you configure which subagents each agent may launch, but its built-in general agent is explicitly limited. In all three, delegation has a real cost: each subagent is a model loop with its own context.

## 4. The arithmetic of saving

### 4.1. The same job, four models

100,000 input tokens and 10,000 output tokens, no cache, at list prices on 30 October 2026:

| Model | Input | Output | Total |
|---|---:|---:|---:|
| Claude Haiku 5.5 (≤100k) | $0.010 | $0.005 | **$0.015** |
| GPT-6 Luna | $0.010 | $0.005 | **$0.015** |
| Claude Sonnet 5.5 | $0.200 | $0.100 | **$0.30** |
| GPT-6.1 Sol | $0.200 | $0.100 | **$0.30** |
| Claude Opus 5.5 | $0.400 | $0.200 | **$0.60** |
| Claude Fable 5.1 | $1.000 | $0.500 | **$1.50** |

The gap between the cheap and the expensive model is **40×** for the same number of tokens. The gap between a subagent and the main thread, at the same model, is **5–10% of cached input**. Routing matters far more than delegating.

### 4.2. Subagent startup

A fresh Claude Code subagent does not start empty: it loads its system prompt, the `CLAUDE.md` hierarchy, git status, tool definitions and preloaded skills. In one third-party measurement, **19 subagents** wrote between **12,427 and 40,763 tokens** on their first request, with **zero cache reads** ([Tokenade, 25 Oct 2026](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0)). That startup is billed at the cache-write rate: **1.25×** input for a 5-minute TTL, **2×** for 1 hour.

At Sonnet 5.5 prices, 30,000 startup tokens cost **$0.075**; at Opus 5.5, **$0.15**; at Haiku 5.5 (≤100k), **$0.004**. Nineteen subagents cost, in startup alone, **$1.43–$2.85** with Sonnet/Opus. It is not a fortune, but it is money paid before the subagent reads a file.

### 4.3. The break-even point

Model: the main thread stops carrying \(D=100,000\) tokens; the subagent costs \(S=30,000\) startup and \(W=100,000\) work; it returns \(R=2,000\) summary and produces \(O=10,000\) output. The main cache is warm and read at 5% of input.

| Main → subagent | 10 turns left | 25 turns | 50 turns | 100 turns |
|---|---:|---:|---:|---:|
| Opus 5.5 → Opus 5.5 | −$0.56 | −$0.26 | +$0.24 | +$1.24 |
| Opus 5.5 → Sonnet 5.5 | −$0.18 | **+$0.12** | +$0.62 | +$1.62 |
| Opus 5.5 → Haiku 5.5 (>100k) | **+$0.10** | +$0.40 | +$0.90 | +$1.90 |
| Sonnet 5.5 → Sonnet 5.5 | −$0.28 | −$0.13 | +$0.12 | +$0.62 |
| Sonnet 5.5 → Haiku 5.5 (>100k) | +$0.002 | +$0.15 | +$0.40 | +$0.90 |

Reading: **with the same model, delegating 100k tokens only starts to pay after roughly 50 remaining turns**. With an expensive main model and a cheap subagent, it pays almost immediately. The table does not include the value of quality or the cost of avoided compaction; if the subagent prevents a single rework, the balance moves.

### 4.4. The fan-out multiplier

If you launch \(N\) subagents in parallel, with a total workload \(W\) split between them and a startup \(S\) per agent, token cost grows roughly by:

$$
M_{\text{fan-out}} \approx 1 + \frac{N \cdot S}{W}
$$

| N | S = 15k, W = 100k | S = 30k, W = 100k | S = 30k, W = 500k |
|---:|---:|---:|---:|
| 2 | 1.30× | 1.60× | 1.12× |
| 5 | 1.75× | 2.50× | 1.30× |
| 10 | 2.50× | 4.00× | 1.60× |

The table uses \(W\) as total workload; with \(N=5\) and \(W=100k\), each agent does 20k of work and pays 30k of startup. Parallelism buys **wall-clock time**, not tokens. Five subagents that finish in a fifth of the time may cost 2.5× the tokens. If quota is the scarce resource, parallelism is a luxury; if the clock is the scarce resource, it is a rational purchase.

## 5. Agnostic conversion

**It can be done, but you have to choose what “working” means.**

- **Common layer**: `AGENTS.md` for instructions (Claude Code reads it if there is no `CLAUDE.md`; Codex and OpenCode use it natively), `SKILL.md` for skills (Agent Skills standard) and MCP for tools.
- **Non-common layer**: the agent file. Claude Code: Markdown + YAML (`description`, `tools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `omitClaudeMd`, `isolation`). Codex: TOML (`name`, `description`, `developer_instructions`, `model`, `model_reasoning_effort`). OpenCode: Markdown + YAML (`mode`, `model`, `permissions` in v2; `permission` and `temperature` in v1).
- **Layer that cannot be copied**: permission semantics, `description`-based routing, the exact model, cache and hooks. A converter can emit a valid file and lose behaviour.

Parity levels proposed by this dossier:

| Level | What it guarantees | How it is checked |
|---|---|---|
| **L0** | Inventory | Parse and list agents |
| **L1** | Valid file | The tool loads it |
| **L2** | Capabilities | Tools, permissions, model and skills mapped |
| **L3** | Behaviour | Same delegation decision on a task set |
| **L4** | Economics | Same cost per accepted task within tolerance |

Most converters reach **L1–L2**. `rulesync` already generates subagents for Claude Code, Codex and OpenCode; Codex has `/import` from Claude Code; xscriptor's converter (`agents/claude/tools/xscriptor-convert.py`) is a real **one-way** OpenCode→Claude case. None guarantees L3–L4 without an evaluation.

## 6. The xscriptor paradox

The `agents` repo contains **205 agents** (181 specialised + 24 senior) and **~17,768 characters of descriptions** (about **4,400 tokens**). Installing all 205 into Claude Code, Codex or OpenCode has two effects:

1. **Fixed cost**: descriptions load into the main context so the model knows whom to delegate to. 4,400 tokens per request is small in dollars (cents per session), but it occupies window and competes for attention.
2. **Routing cost**: 205 options make the choice less reliable. Anthropic says it explicitly: “flooding Claude with options makes automatic delegation less reliable; most teams settle on a handful of well-scoped agents” ([How and when to use subagents](https://claude.com/blog/subagents-in-claude-code)). The library is an excellent catalogue; the default roster should be something else.

The practical recommendation: **full catalogue, tiered installation**. A core of 8–12 agents per project, domain packs on demand, and the rest as skills or definitions that are available but not loaded. The difference between a catalogue and a roster is the difference between a library and a backpack.

## 7. Seven decisions

1. **Measure per accepted task, not per token.** \(C_{\text{accepted}} = C_{\text{run}} / P_{\text{success}}\). A $1.50 agent with 90% success ($1.67) is cheaper than a $0.80 agent with 60% ($1.33) only if the second generates no human rework. Add review cost.
2. **Route before delegating.** If Haiku/Luna can do the task, the saving is 20–40×; context isolation saves 5–10%. Start with the model.
3. **Delegate only disposable, voluminous work.** Wide searches, log triage, tests, independent review, exploration of dozens of files. Do not delegate small edits, sequential chains or files that are already open.
4. **Count the startup.** If the subagent will do less than ~30k tokens of work, startup eats the saving. Use a fork when you need the same context; use the main thread when the context is the task.
5. **Parallelise for time, not money.** Fan-out multiplies tokens. Use it when the clock is worth more than the quota, and cap concurrency.
6. **Convert with a manifest, not faith.** A converter must emit what it mapped, what it lost and what needs verification. Without L3/L4, “it works” means “it loads”.
7. **Prune the roster.** Every unused agent is a description you pay for and an option that distracts. 205 agents are not 205 capabilities; they are 205 decisions.

## 8. Conclusion

The economics of agents is not the economics of the token. It is the economics of **attention**: which context is kept, which is discarded, and what is paid to read it again. A subagent is a way to forget on purpose. Used well, forgetting is cheap; used badly, it is paying twice for the same context and calling it architecture.

The right question is not *“how many agents can I launch?”* but *“which part of this work does not need to exist in the main thread?”*. When the answer is clear, the subagent saves. When it is not, the subagent only adds a layer of indirection between the problem and the invoice.

---

**Next:** [Token accounting](01-token-accounting.md) — the formulas, prices and examples behind these numbers.
