---
title: "Codex — TOML subagents, credits and an inherited sandbox"
description: "How delegation works in Codex, what can be configured per agent, and which part of the cost stays outside the file."
date: "2026-10-30"
tags: ["technology", "codex", "agents", "tokens"]
---

# Codex — TOML subagents, credits and an inherited sandbox

> *Codex delegates like a foreman: the worker inherits the permission, it does not negotiate it. That inheritance makes configuration cheap and mistakes expensive.*

## 1. Map of the tool

Codex has native subagents in the CLI, IDE and app. The official documentation presents them as **enabled by default** in current releases; activity appears as agent threads and the main thread receives only the summary. GA is placed at **March 2026 (v0.115.0)** by secondary documentation; the primary source does not fix the date, so treat it as a reference, not a contractual citation.

Three concepts:

| Term | What it is |
|---|---|
| **Subagent workflow** | Several agents in parallel whose results are consolidated |
| **Subagent / agent thread** | A delegated agent with its own thread and context |
| **Custom agent** | A TOML file that defines role, model, effort and instructions |

Codex ships with three built-in roles:

| Agent | Profile | Typical use |
|---|---|---|
| `default` | General purpose | Fallback |
| `worker` | Implementation and fixes | Code changes |
| `explorer` | Read-only exploration | Mapping the codebase |

A custom agent with the same `name` as a built-in replaces it.

## 2. The agent file

One TOML file per agent:

- `~/.codex/agents/*.toml` — personal.
- `.codex/agents/*.toml` — project, loaded for trusted projects.

```toml
name = "repo_scout"
description = "Read-only codebase explorer for locating relevant paths and symbols."
model = "gpt-6-luna"
model_reasoning_effort = "high"
developer_instructions = """
Stay in exploration mode. Do not edit files.
Trace the real execution path, cite files and symbols, and return a compact map.
Prefer targeted reads over broad scans.
"""
```

### 2.1. Fields

| Field | Required | What it does |
|---|---|---|
| `name` | Yes | Identifier Codex uses to spawn it |
| `description` | Yes | Guidance for when to use it |
| `developer_instructions` | Yes | The agent's system prompt |
| `model` | No | Model; if absent, inherits the parent's |
| `model_reasoning_effort` | No | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` depending on model |
| `model_reasoning_summary` | No | Reasoning summary |
| `model_verbosity` | No | Output verbosity |
| `features.shell_tool` | No | Disables the shell |
| `features.apps`, `plugins`, `memories`, `request_permissions_tool` | No | Disables capabilities |
| `skills.config` | No | Enables/disables specific skills |
| `skills.bundled.enabled` | No | Disables bundled skills |
| `skills.include_instructions` | No | Omits the automatic skills instruction block |

What **cannot** go in the file: `sandbox_mode`, `approval_policy`, `model_provider`, `mcp_servers`. Those settings are inherited from the parent. A custom agent can ask “do not edit” in its instructions, but **it does not create a read-only sandbox**. For that, the sandbox is configured on the parent session or the built-in `explorer` role is used.

### 2.2. Global settings

In `config.toml`:

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 4
default_subagent_model = "gpt-6-luna"
default_subagent_reasoning_effort = "high"
interrupt_message = true
```

| Field | Effect |
|---|---|
| `enabled` | Enables/disables multi-agent tools |
| `max_concurrent_threads_per_session` | Cap on concurrent threads (excluding the main one) |
| `default_subagent_model` | Default model for subagents |
| `default_subagent_reasoning_effort` | Default effort |
| `interrupt_message` | Records a model-visible message when an agent is interrupted |

Resolution is: explicit spawn value → `[agents]` default → parent value. A custom file that sets `model` preserves the previously resolved effort; if the model does not support that effort, set `model_reasoning_effort` too.

## 3. How it is invoked

Codex **does not proactively delegate** in local clients. It does when:

1. You ask explicitly: “spawn two agents”, “delegate this in parallel”, “use one agent per review lens”.
2. An `AGENTS.md` or skill asks for it.
3. In ChatGPT Work with Ultra, it may delegate on its own (not the CLI case).

The delegation prompt should say **how to split**, **whether to wait** and **what to return**. Example:

```text
Review this branch with parallel subagents. Spawn one subagent for security risks,
one for test gaps, and one for maintainability. Wait for all three, then summarize
by category with file references.
```

Subagents inherit the parent's sandbox, approval mode, provider and MCP. In the CLI, approvals from inactive threads surface separately; in non-interactive runs, an action that needs approval fails and the error returns to the parent. In the app, threads can be inspected, stopped or closed.

The orchestrator can spawn new subagents; it is not a capability the custom file controls. Depth and concurrency are decided by the runtime.

## 4. Cost

### 4.1. Credits, not dollars

With a subscription, Codex bills in **credits**; with an API key, in dollars. The two invoices are not convertible. For GPT-6.1 Sol:

| Category | Credits / MTok | USD / MTok (API) |
|---|---:|---:|
| Input | 50 | $2.00 |
| Cached input | 2.5 | $0.10 |
| Output | 250 | $10.00 |
| Cache write | **not charged** | $2.50 (30m) / $5.00 (12h) |

For GPT-6 Luna: 2.5 / 0.25 / 12.5 credits per MTok (API $0.10 / $0.01 / $0.50). For GPT-6.1 Terra: 25 / 1.25 / 125 (API $1.00 / $0.05 / $5.00).

A job of 100k input + 10k output costs, in credits:

| Model | Input | Output | Total | vs Sol |
|---|---:|---:|---:|---:|
| GPT-6.1 Sol | 5.0 | 2.5 | **7.5** | 1× |
| GPT-6.1 Terra | 2.5 | 1.25 | **3.75** | 2× cheaper |
| GPT-6 Luna | 0.25 | 0.125 | **0.375** | 20× cheaper |

The pricing page says a typical GPT-5.6 Sol task may use **2–15 credits**. It is a more intuitive unit than the million tokens, but it does not answer “what does my task cost?” without measurement.

### 4.2. Plans and windows

| Plan | Price | What it includes |
|---|---|---|
| Free | $0 | GPT-6 Luna, quick tasks |
| Go | $8/month | Light Codex use |
| Plus | $20/month | GPT-6.1 Sol and Luna; weekly usage |
| Pro | from $100/month | $100/$200/$500; no 5-hour limit |
| Business / Enterprise | per contract | Credits, flexible pricing |
| API key | per token | No cloud features |

Plus and Standard Business estimate, per 5-hour window: **15–160 local messages** with GPT-6.1 Sol and **350–3,000** with GPT-6 Luna. They are estimates, not fixed limits. Subagents consume the same quota; fan-out spends it faster.

### 4.3. Speed modes

| Mode | Included usage | Purchased credits |
|---|---:|---:|
| Standard | 1× | 1× |
| Fast | 2.5× | 2× |
| Ultrafast (Astra/Sol) | 8× | 6× |

Fast and Ultrafast buy latency with quota. For an exploration subagent, Standard is usually enough; for a review against the clock, Fast may make sense.

### 4.4. Where to see it

- `/status` in the CLI: active model and remaining usage.
- Usage dashboard: `chatgpt.com/codex/settings/usage`.
- The app shows subagent activity and the summary they return.
- In the API, the response and organization logs.

There is no “cost per subagent” line equivalent to Claude Code; analysis is by thread and model.

## 5. Recipes

### 5.1. Cheap explorer

```toml
name = "repo_scout"
description = "Maps relevant code paths before implementation. Read-only."
model = "gpt-6-luna"
model_reasoning_effort = "high"
features.shell_tool = false
developer_instructions = """
Do not edit files. Map entry points, data flow and likely files.
Return a compact list with paths and symbols. Stop when the path is clear.
"""
```

Why it saves: Luna is 20× cheaper than Sol per token; without a shell, the agent cannot wander by running commands; output is bounded. Limit: it is not a real sandbox; the parent must run with the right sandbox.

### 5.2. Reviewer

```toml
name = "reviewer"
description = "Reviews a diff for correctness, security and missing tests."
model = "gpt-6.1-sol"
model_reasoning_effort = "high"
developer_instructions = """
Review like an owner. Do not edit files. Prioritize exploitable issues,
behavior regressions and missing tests. Lead with findings and file references.
"""
```

Why it saves: the expensive model is used only for judgement; the implementation context stays in the parent.

### 5.3. Implementation worker

```toml
name = "ui_fixer"
description = "Implements a small, well-scoped fix after the failure is understood."
model = "gpt-6.1-terra"
model_reasoning_effort = "medium"
developer_instructions = """
Make the smallest defensible change. Touch only the files named in the task.
Run the narrowest verification command and report the exact result.
"""
```

Why it saves: Terra is 2× cheaper than Sol; the narrow scope avoids long exploration.

### 5.4. Project defaults

```toml
[agents]
max_concurrent_threads_per_session = 4
default_subagent_model = "gpt-6-luna"
default_subagent_reasoning_effort = "high"
```

A cheap default stops every spawn from inheriting Sol by inertia. Agents that need Sol declare it.

## 6. The savings playbook

1. **Luna to explore, Terra to implement, Sol to review.** It is the most cost-effective assignment at current prices.
2. **Cap concurrency.** `max_concurrent_threads_per_session` is a budget, not decoration. Four threads are usually enough.
3. **Be explicit about the split.** A vague prompt produces duplication; duplication is the biggest hidden cost multiplier.
4. **Avoid write-heavy fan-out.** Two agents editing the same file create conflicts and rework; use parallelism for reading, tests and review.
5. **Use inheritance.** Sandbox, approval mode and MCP are configured once on the parent; do not try to replicate them in every TOML.
6. **Use nested AGENTS.md.** Codex reads the nearest `AGENTS.md`; a monorepo with per-package instructions avoids loading the whole world.
7. **Measure in credits and accepted tasks.** A 0.375-credit subagent that fails is more expensive than a 7.5-credit one that succeeds.
8. **Do not confuse Fast with cheap.** Fast multiplies quota; use it when the clock is worth more.

## 7. Traps

- **Thinking the TOML isolates permissions.** It does not: it inherits sandbox, approval, provider and MCP. The file can disable features, not create a sandbox.
- **Asking for “read-only” in instructions.** It is an intention, not a control. Use the `explorer` role or the parent sandbox.
- **Assuming Codex delegates on its own.** In local clients it does not proactively; ask for it or put it in `AGENTS.md`/a skill.
- **Ignoring the cost of concurrency.** Each thread is a loop; four threads do not cost the same as one.
- **Using API prices to estimate the plan.** The documentation itself advises against it.
- **Forgetting long context.** In the API, >272k doubles input and multiplies output; on a subscription, GPT-6.1 Sol has no long-context surcharge, but usage still counts.
- **Confusing credits with quality.** Luna is cheap and capable for exploration; it is not Sol for decisions.

## 8. Relevance for xscriptor

xscriptor's converter does not emit TOML, so Codex is currently the destination with the most manual work. If an adapter is added, three decisions matter:

1. **Map tiers, not IDs**: `fast` → `gpt-6-luna`, `balanced` → `gpt-6.1-terra`, `frontier` → `gpt-6.1-sol`. That way the catalogue survives model changes.
2. **Do not promise a per-agent sandbox**: Codex inherits it. An audit agent should use the `explorer` role or the parent sandbox, not a TOML that says “do not edit”.
3. **Cap concurrency per project**: 205 agents do not imply 205 threads; `max_concurrent_threads_per_session` is the economic brake.

## 9. Closing

Codex is the most opaque tool in money and the clearest in inheritance: the subagent does not negotiate permissions, it receives them. That simplicity is a configuration virtue and a security risk. The real saving comes from model assignment — Luna, Terra, Sol — and concurrency discipline, not from a magic field in the TOML.

A Codex subagent is a thread with a short contract. The shorter the contract, the cheaper the thread; the vaguer it is, the more it looks like hiring a second team to repeat the meeting.

---

**Next:** [OpenCode](05-opencode.md) — the open version: two formats, permissions and child sessions.
