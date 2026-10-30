---
title: "Claude Code — subagents, forks and the price of a clean context"
description: "How a Claude Code subagent is defined, invoked and paid for; which fields matter and which recipes actually save."
date: "2026-10-30"
tags: ["technology", "claude-code", "agents", "tokens"]
---

# Claude Code — subagents, forks and the price of a clean context

> *Claude Code does not delegate to work less: it delegates so the main thread does not remember what it does not need. Memory, here, is billed per turn.*

## 1. Map of the tool

Claude Code has four mechanisms that are often confused:

| Mechanism | What it is | Context | Relative cost |
|---|---|---|---|
| **Subagent** | Specialised agent with its own window, invoked with the `Agent` tool | New, except forks | Startup + work |
| **Fork** | Subagent that inherits the parent conversation | Copy of the parent | Shares cache; cheaper |
| **Agent team** | Several Claude Code sessions coordinated by a lead | Independent each | ~7× a standard session in plan mode |
| **Skill** | Instructions loaded on demand in the current thread | The same thread | Load cost, no agent startup |

The key difference: a subagent **does not see the conversation** (except a fork), and a team **does not share context**: it shares messages and a task list. If the problem is “this thread has too much noise”, subagent. If it is “I need several people to discuss”, team. If it is “I need a reusable instruction without creating an agent”, skill.

## 2. The agent file

Subagents are Markdown with YAML frontmatter:

- `~/.claude/agents/` — personal, all projects.
- `.claude/agents/` — project, versionable.
- Plugins, managed settings and `--agents` JSON — other scopes.

```markdown
---
name: repo-scout
description: Explores a repository and returns file paths, symbols and risks. Use for wide searches before implementation.
tools: Read, Grep, Glob
model: haiku
maxTurns: 20
omitClaudeMd: true
color: cyan
---

You are a read-only repository scout. Search broadly, read only what is necessary,
and return a compact list of findings with file paths and line references.
Do not propose patches. Do not edit files.
```

### 2.1. Fields that change cost

| Field | What it does | Economic effect |
|---|---|---|
| `description` | The parent uses it to decide whether to delegate | Loaded in the main context; keep it short |
| `tools` | Tool allowlist | Fewer schemas = fewer tokens per request; less wandering |
| `disallowedTools` | Denylist | Alternative when you want to inherit almost everything |
| `model` | `haiku`, `sonnet`, `opus`, `fable`, ID or `inherit` | The biggest lever: 2–40× per token |
| `permissionMode` | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`, `manual` | Controls autonomy and risk, not tokens directly |
| `maxTurns` | Caps the loop | Prevents endless exploration |
| `autoCompactWindow` | Lowers the compaction threshold | Less context, more compactions |
| `skills` | Preloads full skills | Saves exploration, but loads tokens at startup |
| `mcpServers` | MCPs for this agent only | Keeps their schemas out of the main context |
| `hooks` | Agent-scoped hooks | Can filter output before it reaches the model |
| `memory` | Persistent memory (`user`, `project`, `local`) | Avoids re-exploration; adds context |
| `background` | Keeps the agent in the background | Useful for independent work |
| `omitClaudeMd` | Skips user/project/local `CLAUDE.md` | Saves thousands of tokens per request |
| `effort` | `low`…`max` | More effort = more reasoning tokens (output) |
| `isolation: worktree` | Isolated copy of the repo | Safety and parallelism, not direct savings |
| `experimental.cacheTtl` | `5m` or `1h` | 1-hour cache costs 2× the write |

Only `name` and `description` are required. All subagent descriptions load into the main context; when they exceed **15,000 tokens**, Claude Code warns at startup. It is a sign of sprawl, not just cost.

### 2.2. What a subagent loads

A non-fork subagent starts with:

- its system prompt (the file body);
- the delegation message Claude writes;
- the `CLAUDE.md` hierarchy (except `Explore`, `Plan` and `omitClaudeMd: true`);
- the `git status` snapshot;
- preloaded skills;
- the definitions of allowed tools.

That payload is written to cache from scratch. In one independent measurement of 19 subagents, the first request wrote between **12,427 and 40,763 tokens** and read **zero** from cache ([Tokenade, 25/10/2026](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0)). It is not a law, but it gives the order of magnitude: **subagent startup is tens of thousands of tokens**.

### 2.3. Fork

A fork inherits the parent's prompt, tools and conversation. Its first request **reuses the parent cache**, so it is cheaper than a fresh subagent for tasks that need the same context. It is launched with the `Agent` tool and `isolation`/fork depending on the version, or from a skill with `context: fork`. Limit: it drags all the context, including noise, and cannot spawn more forks.

## 3. How it is invoked

1. **Automatic**: Claude reads descriptions and decides. A description with a clear trigger (“use before commits”, “when exploring a large codebase”) routes better than a generic one (“security expert”).
2. **Explicit**: “Use the repo-scout agent to…”, or `@agent` in the UI.
3. **Via command/skill**: a skill with `context: fork` and `agent` runs the work in a subagent.
4. **Via `--agents` JSON**: ephemeral definitions for automation.
5. **Nested**: a subagent can spawn another up to **3 layers** by default (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`). At the limit, it loses the `Agent` tool.
6. **Concurrent**: by default, **20** subagents at once (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`). The limit is not a budget.

## 4. Cost, with names and numbers

### 4.1. Startup

30,000 startup tokens, 5-minute cache write:

| Model | Cost |
|---|---:|
| Haiku 5.5 (≤100k) | $0.00375 |
| Sonnet 5.5 | $0.075 |
| Opus 5.5 | $0.15 |

### 4.2. Break-even

With \(D=100k\), \(S=30k\), \(W=100k\), \(O=10k\), \(R=2k\):

| Main → subagent | 10 turns | 25 turns | 50 turns |
|---|---:|---:|---:|
| Opus → Sonnet | −$0.18 | +$0.12 | +$0.62 |
| Sonnet → Sonnet | −$0.28 | −$0.13 | +$0.12 |
| Opus → Haiku (>100k) | +$0.10 | +$0.40 | +$0.90 |

The practical conclusion: **if the subagent will do little work, do not launch it for money**. If the main thread is Opus and the subagent can be Sonnet/Haiku, routing pays before isolation.

### 4.3. Where to see it

- `/usage` shows session cost, usage per model and, on subscription plans, attribution to skills, subagents, plugins and MCP.
- The `Prompt cache (main)` line shows hits, misses and whether the cache is warm.
- Transcripts are in `~/.claude/projects/<project>/<session>/` and, for subagents, in `subagents/agent-<id>.jsonl`. Each assistant message carries `usage` with `input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` and `output_tokens`. **Deduplicate by `message.id`**: a streamed response appears on several lines with the same usage.
- OpenTelemetry exports tokens and cost per user and session.

### 4.4. Subscription

On Pro/Max, subagents consume the same 5-hour/weekly window as the main conversation. There is no public dollar conversion. The consequence is direct: **five parallel subagents add five contexts and five loops, and can exhaust the window much sooner**, even if the marginal dollar cost is zero until it runs out.

### 4.5. Agent teams

Teams are different: independent sessions, a shared task list, peer messages. Anthropic reports **~7× tokens** versus a standard session when teammates run in plan mode, and recommends Sonnet for teammates, small teams and shutting them down when done. They are not the cheap option; they are the option for work that needs discussion or divided ownership.

## 5. Recipes

### 5.1. Cheap explorer

```markdown
---
name: repo-scout
description: Read-only repository exploration. Use before implementing in an unfamiliar area.
tools: Read, Grep, Glob
model: haiku
maxTurns: 25
omitClaudeMd: true
---

Search broadly. Return only: relevant paths, symbols, data flow, and risks.
No patches. No file edits. If you cannot answer, say what is missing.
```

Why it saves: 20–40× cheaper model, no `CLAUDE.md`, minimal tools, bounded output.

### 5.2. Independent reviewer

```markdown
---
name: security-reviewer
description: Reviews a diff for security regressions before commit. Use on staged changes.
tools: Read, Grep, Glob, Bash(git diff:*), Bash(git status:*)
model: sonnet
permissionMode: plan
maxTurns: 30
color: red
---

Review the staged diff as an attacker. Report only exploitable issues with file,
line, impact and a concrete fix. Do not edit files.
```

Why it saves: clean context (it does not drag the implementation), read-only permissions, prioritised output. The cost is startup; the benefit is avoiding a failure.

### 5.3. Orchestrator with an allowlist

When the agent runs as the main thread (`claude --agent orchestrator`), the type allowlist works:

```markdown
---
name: orchestrator
description: Coordinates a small roster of specialists.
tools: Agent(repo-scout, security-reviewer), Read, Bash
model: sonnet
---

Delegate exploration to repo-scout and security review to security-reviewer.
Keep implementation in this thread. Wait for both results before editing.
```

In a subagent, listing `Agent` allows it to spawn others, but the parenthesised list is ignored; to restrict, use `permissions.deny` or omit `Agent`.

### 5.4. Skill with a fork

```markdown
---
name: deep-review
description: Runs a three-part review of the staged changes.
context: fork
agent: security-reviewer
disable-model-invocation: true
---

Review the staged changes for security, correctness and test gaps.
Return a short report; do not edit.
```

The fork reuses the parent cache; it is the cheap option when the review needs the conversation context.

## 6. The savings playbook

Ordered by impact:

1. **Route the model.** `haiku` for exploration and summaries; `sonnet` for implementation and review; `opus` for architectural decisions. The difference is 2–40×. To force one model onto every subagent, combine `CLAUDE_CODE_SUBAGENT_MODEL` with `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1`.
2. **Trim startup.** `omitClaudeMd: true` for agents that get everything in the prompt; minimal `tools`; only necessary `skills`; `mcpServers` out of the main thread.
3. **Cap the loop.** `maxTurns` and `autoCompactWindow`; a subagent that explores for 40 turns is a long session in disguise.
4. **Use a fork when the context is the task.** It shares cache and avoids fresh startup.
5. **Choose the TTL.** 5 minutes by default; 1 hour only across long pauses and if the work continues. The 1-hour write costs 2×.
6. **Prune the roster.** Fewer agents = fewer descriptions, less ambiguity, better routing. 205 agents are not 205 capabilities.
7. **Measure per accepted task.** `/usage` gives tokens; the team gives success, rework and human minutes.
8. **Do not switch models mid-task.** It invalidates the cache and can cost a full re-read.

## 7. Traps

- **Thinking a subagent sees the conversation.** It does not (except a fork). If it needs a detail from 20 turns ago, it will search or guess; both cost.
- **Long descriptions.** They always load. The 15k warning is the symptom; routing quality is the problem.
- **Duplicated `CLAUDE.md`.** Every custom subagent loads it on every request. If the subagent does not need it, `omitClaudeMd`.
- **Inherited model by accident.** `Explore` inherits the parent model (capped at Opus on the Anthropic API); a subagent without `model` does too. If you wanted Haiku, declare it.
- **Subagent window.** The window is set by its model, not the parent. A Haiku subagent has a smaller window and may need more compaction.
- **Plugin agents.** They do not support `hooks`, `mcpServers` or `permissionMode`; if you need them, copy the file to `.claude/agents/`.
- **Depth and concurrency.** A subagent that spawns subagents multiplies consumption; the limit of 20 is not a budget.
- **Teams as a synonym for subagents.** Teams are ~7× and experimental; use them when the work needs discussion, not just parallelism.

## 8. Relevance for xscriptor

xscriptor's catalogue (205 agents, 142 with `edit: allow`) is an ideal case for Claude Code and a risk if installed whole. Three concrete actions:

1. **Install a roster of 8–12 agents per project**, not all 205. Descriptions always load; routing improves when the model has fewer options.
2. **Convert with model and limits**, not just name and prompt. The current converter preserves neither `maxTurns` nor the `task` allowlist; adding them reduces cost and risk.
3. **Review editable agents.** Those with `edit: allow` and `bash: ask` end up in Claude with every tool; the source asked to prompt. That is where conversion changes cost and safety.

## 9. Closing

Claude Code offers the most complete toolbox of the three, and the easiest to over-configure. The difference between a roster of eight agents and a library of two hundred is not the number: it is that the first fits in the model's head and the second competes with itself.

A well-placed subagent is a pruning: it removes from the main thread what it does not need to remember. A badly placed subagent is a move: it pays to carry the context to another house and to bring it back summarised.

---

**Next:** [Codex](04-codex.md) — the same problem with TOML, credits and an inherited sandbox.
