---
title: "Token accounting — the formula nobody teaches"
description: "Input, cache, output, TTL and model: how an agent session becomes an invoice, with worked numbers."
date: "2026-10-30"
tags: ["technology", "tokens", "economics", "cache"]
---

# Token accounting — the formula nobody teaches

> *A token costs different things the first time it is read, the second time it is remembered and the third time it is rewritten. The invoice is the story of those three readings.*

## 1. What is billed

Providers do not bill “a conversation”. They bill four token categories, each with its own price:

| Category | What it is | When it is paid |
|---|---|---|
| **Uncached input** | New tokens the model processes | On every request, for the part not in cache |
| **Cache write** | Tokens stored for reuse | The first time, at a premium (1.25× or 2×) |
| **Cache read** | Tokens retrieved from a valid cache | On later requests, at a discount (0.05×–0.1×) |
| **Output** | Tokens the model generates, including reasoning | Always, at the output price |

The general formula for one request:

$$
C = p_{\text{in}} \cdot U + p_{\text{cw}} \cdot W + p_{\text{cr}} \cdot R + p_{\text{out}} \cdot O
$$

where \(U\) is uncached input, \(W\) cache write, \(R\) cache read and \(O\) output. For a session, sum over all requests:

$$
C_{\text{session}} = \sum_{t=1}^{T} \left( p_{\text{in}} U_t + p_{\text{cw}} W_t + p_{\text{cr}} R_t + p_{\text{out}} O_t \right)
$$

This sum is what turns an architecture decision into money. A subagent adds requests; a model change changes all four prices; a misaligned cache changes \(W\) and \(R\).

## 2. Reference prices (30 October 2026)

### 2.1. Anthropic

USD per million tokens (MTok). The 5-minute cache is written at 1.25× input; the 1-hour cache at 2×.

| Model | Input | Output | Cache read | 5m write | 1h write |
|---|---:|---:|---:|---:|---:|
| Claude Haiku 5.5 (≤100k) | $0.10 | $0.50 | $0.01 | $0.125 | $0.20 |
| Claude Haiku 5.5 (>100k) | $0.50 | $2.50 | $0.05 | $0.625 | $1.00 |
| Claude Sonnet 5.5 | $2.00 | $10.00 | $0.10 | $2.50 | $4.00 |
| Claude Opus 5.5 | $4.00 | $20.00 | $0.20 | $5.00 | $8.00 |
| Claude Fable 5.1 | $10.00 | $50.00 | $0.25 | $12.50 | $20.00 |

Notes that change the comparison:

- **Haiku 5.5 has two rates**: if the prompt exceeds 100,000 tokens, input is multiplied by 5 and output by 5. A 130k-token subagent does not pay the cheap rate.
- **Cache read is not 0.1× on every model**: it is **0.05×** on Opus 5.5 and Sonnet 5.5, and **0.025×** on Fable 5.1 and Mythos 5.1. The table already reflects this.
- **Claude 4.7 and later use a new tokenizer** that produces ~30% more tokens for the same text. Comparing per-token prices across generations without correcting for this overstates the saving of newer models.
- **Batch API**: 50% off input and output, but it is not interactive; it does not replace a subagent in a session.
- **Fast mode**: doubles the price of Opus 5.5 ($8/$40) and Opus 5/4.8 ($10/$50); it stacks with caching and data residency.
- **Data residency** (`inference_geo: "us"`): 1.1× on all categories for Claude 4.6+.

Source: [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing), read on 30/10/2026.

### 2.2. OpenAI

USD per MTok, short context (≤272k):

| Model | Input | Output | Cached input | 30m write | 12h write |
|---|---:|---:|---:|---:|---:|
| GPT-6.1 Sol | $2.00 | $10.00 | $0.10 | $2.50 | $5.00 |
| GPT-6.1 Terra | $1.00 | $5.00 | $0.05 | $1.25 | $2.50 |
| GPT-6 Luna | $0.10 | $0.50 | $0.01 | $0.125 | $0.25 |
| GPT-6 Astra | $10.00 | $50.00 | $1.00 | $12.50 | $25.00 |

Notes:

- **Long context** (>272k): GPT-6.1 Sol goes to $4.00 input and $15.00 output; cached input to $0.20.
- **Codex with a subscription** does not bill dollars per token: it bills **credits**. For GPT-6.1 Sol, 50 credits per MTok input, 2.5 per MTok cached and 250 per MTok output. **There is no separate cache-write charge** in Codex billing.
- **API prices cannot estimate tasks included in a plan.** Codex's pricing page says it explicitly: “API token prices are separate from subscription usage; don't use them to estimate included tasks”.
- **Fast** multiplies included usage by 2.5 and purchased credits by 2; **Ultrafast** by 8 and 6, respectively.

Source: [OpenAI API pricing](https://platform.openai.com/docs/pricing) and [Codex pricing](https://learn.chatgpt.com/docs/pricing), read on 30/10/2026.

### 2.3. OpenCode

OpenCode is not a model provider: it is a client and, optionally, a gateway (Console/Zen). The cost depends on the provider you choose. OpenCode Console publishes a per-model table; examples from 30/10/2026:

| Model | Input | Output | Cache read |
|---|---:|---:|---:|
| Claude Opus 5.5 | $4.00 | $20.00 | $0.20 |
| Claude Sonnet 5.5 | $2.00 | $10.00 | $0.10 |
| Claude Haiku 5.5 (≤100k) | $0.10 | $0.50 | $0.01 |
| GPT-6.1 Sol (≤272k) | $2.00 | $10.00 | $0.10 |
| DeepSeek V4.1 Flash | $0.30 | $1.20 | $0.006 |
| GLM 5.3 Flash | $0.15 | $0.50 | $0.03 |

The DeepSeek V4.1 Flash detail is instructive: its cache read is **$0.006/M**, 98% cheaper than its input. That makes an exploration subagent with repeated context almost free in that combination. But the model, latency and quality are not Opus; the saving is real and the comparison is not equivalent.

Source: [OpenCode Console pricing](https://opencode.ai/docs/zen/), read on 30/10/2026.

## 3. The cache: where it is won and lost

### 3.1. How it works

The prompt cache stores a **prefix** of the request. If the next request starts exactly the same way, the provider reads that prefix from cache instead of reprocessing it. The prefix includes, in order:

1. The system prompt and tool definitions.
2. Project context (`CLAUDE.md`, `AGENTS.md`, rules, loaded skills).
3. The conversation: messages, tool results, responses.

A change anywhere invalidates everything after it. That is why switching models mid-session, connecting an MCP or editing the system prompt can cost a full re-read.

### 3.2. The multipliers that matter

| Operation | Anthropic | OpenAI | Practical effect |
|---|---|---|---|
| Cache read | 0.05×–0.1× | 0.05× | Repeated context is almost free |
| 5m/30m write | 1.25× | 1.25× | The first time costs 25% more |
| 1h/12h write | 2× | 2.5× | Long cache is more expensive; only pays across pauses |
| Model switch | Invalidates | Invalidates | A mid-task switch can cost the whole session |
| Compaction | Rewrites the conversation | Equivalent | The next request shares no prefix |

The golden rule: **cache rewards stability**. An agent that changes model, tools or prompt mid-task pays for a context it had already paid for.

### 3.3. The subagent case

A subagent **does not share the parent cache** unless it is a *fork*. Its system prompt, tools and context are different, so its first request writes its own prefix and does not read the parent's. In Claude Code:

- A normal subagent starts with **zero cache reads** and pays the write for its startup.
- A **fork** inherits the parent's prompt, tools and conversation; its first request reuses the parent cache. It is cheaper for tasks that need the same context.
- Subagents have their own TTL (5 minutes by default, even on a subscription); it can be raised with `subagentPromptCacheTtl` or the `cacheTtl` frontmatter field.

Source: [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching), [subagents](https://code.claude.com/docs/en/sub-agents).

## 4. Worked examples

### 4.1. The same job on four models

100k input + 10k output, no cache:

| Model | Input | Output | Total | vs Haiku |
|---|---:|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0.010 | $0.005 | **$0.015** | 1× |
| GPT-6 Luna | $0.010 | $0.005 | **$0.015** | 1× |
| Sonnet 5.5 | $0.200 | $0.100 | **$0.30** | 20× |
| GPT-6.1 Sol | $0.200 | $0.100 | **$0.30** | 20× |
| Opus 5.5 | $0.400 | $0.200 | **$0.60** | 40× |
| Fable 5.1 | $1.000 | $0.500 | **$1.50** | 100× |

If those 100k were cache reads and only 10k were new input:

| Model | New input | Cache | Output | Total |
|---|---:|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0.001 | $0.001 | $0.005 | **$0.007** |
| Sonnet 5.5 | $0.020 | $0.010 | $0.100 | **$0.13** |
| Opus 5.5 | $0.040 | $0.020 | $0.200 | **$0.26** |
| GPT-6.1 Sol | $0.020 | $0.010 | $0.100 | **$0.13** |

Caching reduces input cost by an order of magnitude. That explains why “keeping context in the main thread” is not as expensive as it looks — and why removing it does not save as much as it looks either.

### 4.2. Subagent startup

30,000 startup tokens, 5-minute cache write:

| Model | Startup cost |
|---|---:|
| Haiku 5.5 (≤100k) | $0.00375 |
| Sonnet 5.5 | $0.075 |
| Opus 5.5 | $0.15 |
| Fable 5.1 | $0.375 |

If the subagent does 10,000 tokens of work and returns 1,000, total cost (excluding output) is:

| Model | Startup | Work | Total |
|---|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0.00375 | $0.001 | **$0.00475** |
| Sonnet 5.5 | $0.075 | $0.020 | **$0.095** |
| Opus 5.5 | $0.15 | $0.040 | **$0.19** |

At Sonnet, a subagent that does 10k of work costs **$0.095** before output. If the work could be done in the main thread for $0.02, the subagent is a luxury. If the work produces 100k of logs that would stay in context, the math changes.

### 4.3. The break-even point, with numbers

Let \(D=100,000\) tokens kept out of the main thread, \(T\) remaining turns, \(r=0.05\) the cache-read fraction, \(p_{\text{in}}\) the main model's input price. The saving is:

$$
\text{saving}(T) = D \cdot T \cdot r \cdot p_{\text{in}}
$$

The subagent cost, with startup \(S=30,000\), work \(W=100,000\), output \(O=10,000\) and summary \(R=2,000\), is:

$$
\text{cost}_{\text{sub}} = (1.25S + W) \cdot p_{\text{in,sub}} + O \cdot p_{\text{out,sub}} + R \cdot p_{\text{in,main}}
$$

With Sonnet 5.5 on both sides:

- Saving per turn: \(100,000 \times 0.05 \times \$2/M = \$0.01\).
- Subagent cost: \((37,500+100,000)\times\$2/M + 10,000\times\$10/M + 2,000\times\$2/M = \$0.275 + \$0.10 + \$0.004 = \$0.379\).
- Break-even: \(T = 0.379/0.01 \approx 38\) turns.

With Opus 5.5 main and Sonnet 5.5 subagent:

- Saving per turn: \(100,000 \times 0.05 \times \$4/M = \$0.02\).
- Subagent cost: **the same**, $0.379.
- Break-even: \(T \approx 19\) turns.

With Opus 5.5 on both:

- Saving per turn: $0.02.
- Cost: \((37,500+100,000)\times\$4/M + 10,000\times\$20/M + 2,000\times\$4/M = \$0.55 + \$0.20 + \$0.008 = \$0.758\).
- Break-even: \(T \approx 38\) turns.

The lesson: **routing to a cheaper model halves the break-even; isolation alone does not**. And if the subagent is more expensive than the main model, the break-even moves further away.

## 5. Subscription is not dollars

In Claude Code Pro/Max, Codex Plus/Pro and OpenCode Go, the user pays a fixed fee and consumes a **usage window**. There is no public token-to-dollar conversion for those windows. What is known:

- **Claude Code**: every subagent consumes the same quota as the main conversation. `/usage` shows attribution by skills, subagents, plugins and MCP; the window is 5-hour and weekly. Five parallel subagents can exhaust a window much faster than a sequential session.
- **Codex**: subagents consume the same plan credits. `/status` shows the model and usage; the dashboard shows limits. The credit table lets you compare models, but it does not convert quota into dollars.
- **OpenCode Go**: limits are per model, with 20% of the monthly limit per 5-hour window, 50% weekly and 100% monthly. A large fan-out on one model can lock that model for the rest of the day.

The practical consequence: **if you pay a subscription, the marginal cost of a token is zero until it is not**. The scarce resource is not money; it is the window. That changes the decision: a subagent that saves 20 minutes may be rational even if it consumes 3× the tokens, if the window holds.

## 6. Cost per accepted task

The metric that matters is not \(C_{\text{session}}\), but:

$$
C_{\text{accepted}} = \frac{C_{\text{run}} + C_{\text{rework}} \cdot P_{\text{fail}}}{P_{\text{success}}}
$$

If a retry costs the same as a run, \(C_{\text{accepted}} = C_{\text{run}} / P_{\text{success}}\). Examples:

| Cost per run | Success rate | Cost per accepted task |
|---:|---:|---:|
| $0.50 | 70% | **$0.71** |
| $0.80 | 60% | **$1.33** |
| $1.50 | 90% | **$1.67** |
| $3.00 | 95% | **$3.16** |

A more expensive but more reliable agent can be cheaper per accepted task. And if failure implies 30 minutes of human review, the math explodes: at $60/hour, 30 minutes is $30, twenty times the token cost in the example.

## 7. What the price table does not say

- **Quality**: the cheap model may fail more; cost per accepted task captures it, price per token does not.
- **Latency**: a parallel subagent can reduce wall-clock time without reducing tokens; a cheap model can be slower.
- **Compaction**: a large context forces summarisation; compaction is a large request and can lose information.
- **Quota**: a subscription does not convert to dollars; the window does run out.
- **Reasoning tokens**: billed as output; high effort can multiply output without changing input.
- **Tokenizer**: the same text can be 30% more tokens on newer models.
- **Taxes, discounts and agreements**: list prices are not the final invoice.

## 8. Closing

Token accounting is uncomfortable because it forces you to separate three things intuition mixes: what it costs to **read**, what it costs to **remember** and what it costs to **read again**. A subagent is a bet on the third. Sometimes it wins; sometimes it just pays twice for the same context.

The formula does not decide for you. But it stops the decision from being made by an anecdote.

---

**Next:** [Subagents and context](02-subagents-and-context.md) — why isolation sometimes saves and sometimes only moves the cost.
