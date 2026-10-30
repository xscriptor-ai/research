---
title: "OpenCode — two formats, permissions and child sessions"
description: "How OpenCode delegates in v1 and v2, what changes in the permission model, and where the real saving is."
date: "2026-10-30"
tags: ["technology", "opencode", "agents", "tokens"]
---

# OpenCode — two formats, permissions and child sessions

> *OpenCode leaves the door open and asks you to write the rule. Freedom is the format; the cost is the rule you forget.*

## 1. Map of the tool

OpenCode distinguishes two agent types:

- **Primary**: the assistant you talk to; `build` and `plan` are built in.
- **Subagent**: an agent the primary launches for a task; `general` and `explore` are built in.

An agent is defined in Markdown or JSONC. The filename is the agent name (unless the format allows an explicit `name`). Subagents run in **child sessions** with fresh context, in the foreground or background; the parent receives the result.

The version installed in this environment is **OpenCode v2.1.4**, but the xscriptor ecosystem and many public examples still use the v1 format. This dossier documents both because converting between them is already an economic problem: an agent that does not load is an agent that does not save.

## 2. The two formats

### 2.1. v1 (the historical format)

```markdown
---
description: Reviews code for quality and best practices
mode: subagent
model: anthropic/claude-sonnet-5-5
temperature: 0.1
permission:
  edit: deny
  bash:
    "*": ask
    "git diff": allow
    "git log*": allow
  webfetch: deny
---

You are in code review mode. Report findings without making changes.
```

Common fields: `description`, `mode`, `model`, `temperature`, `top_p`, `steps`, `permission`, `tools` (deprecated), `hidden`, `color`, `disable`.

### 2.2. v2 (the current format)

```markdown
---
description: Reviews changes for correctness and regressions
mode: subagent
model: anthropic/claude-sonnet-5-5#high
permissions:
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
---

Review the current changes. List findings in severity order with file and line references.
```

Fields: `description`, `mode`, `model` (`provider/model#variant`), `permissions` (ordered list of `action`/`resource`/`effect`), `system`, `steps`, `hidden`, `disabled`, `color`, `request` (per-agent headers/body).

### 2.3. v1 → v2 translation table

| v1 | v2 | Note |
|---|---|---|
| `permission` (map) | `permissions` (array) | Order matters; the last matching rule wins |
| `tools` | `permissions` | `tools` is deprecated |
| `agent` (config) | `agents` | Plural |
| `provider` | `providers` | The v1 spelling still loads |
| `mcp` with `enabled` | `mcp.servers` with `disabled` | The flag is inverted |
| `command` / `subtask` | `commands` / `subagent` | Rename |
| `skills.paths` + `skills.urls` | `skills` (array) | Merge |
| `autoupdate` | `update` | `disable`/`notify`/`auto` |
| `plugin` | `plugins` | Requires porting code |
| `tui.json` | `cli.json` | Project settings do not migrate |
| `CLAUDE.md` | `AGENTS.md` | v2 only discovers `AGENTS.md` |
| `bash` | `shell` | Permission action |
| `task` | `subagent` | Permission action |
| `write` / `patch` | `edit` | Permission action |

The table combines the official v1/v2 documentation with a third-party migration guide ([Shattered, 27/10/2026](https://shattered.io/opencode-2-upgrade-from-1x-12-steps-2026/)); the permission and `CLAUDE.md` rows should be verified on the exact version you use. In this environment, v2.1.4, the direction is as described.

## 3. How it is invoked

1. **Automatic**: the primary reads descriptions and launches the right subagent.
2. **Manual**: `@general help me search for this function`, or the agent's name.
3. **By permissions**: `permission.task` (v1) or `permissions` with `action: subagent` (v2) decide which agents each one may launch.
4. **Background**: v2 can launch the subagent asynchronously and continue working; the parent receives the result when it finishes.
5. **Navigation**: child sessions are traversed with shortcuts (`session_child_first`, `session_child_cycle`, `session_parent`).

The subagent uses its configured model or, if none, **inherits the parent's**. That is a cost trap: a subagent without `model` in an Opus session is an Opus subagent.

### 3.1. Nesting

OpenCode v2 documents that the built-in `general` **cannot launch more subagents**. Subagent-to-subagent delegation exists in the permission model (a subagent with `action: subagent` allowed can launch another), but it is not the default behaviour or the recommended path. In v1, nested delegation was an opt-in feature in the past; do not assume it is available without checking.

The economic consequence: OpenCode is excellent for **one level** of delegation; for deep trees, Claude Code and Codex offer more explicit semantics.

## 4. Cost

### 4.1. Model per agent

OpenCode uses `provider/model#variant`. The model is chosen per agent; if absent, it inherits. Examples:

```yaml
# cheap explorer
model: deepseek/deepseek-v4.1-flash
# reviewer
model: anthropic/claude-sonnet-5-5#high
# orchestrator
model: anthropic/claude-opus-5-5
```

The routing saving is the same as in the other tools: 20–40× between a cheap and an expensive model. OpenCode adds an advantage: you can mix providers in the same team (Anthropic, OpenAI, DeepSeek, GLM, Kimi, local), and the price depends on the provider, not the client.

### 4.2. Reference prices

With OpenCode Console (30/10/2026):

| Model | Input | Output | Cache read |
|---|---:|---:|---:|
| Claude Opus 5.5 | $4.00 | $20.00 | $0.20 |
| Claude Sonnet 5.5 | $2.00 | $10.00 | $0.10 |
| Claude Haiku 5.5 (≤100k) | $0.10 | $0.50 | $0.01 |
| GPT-6.1 Sol (≤272k) | $2.00 | $10.00 | $0.10 |
| DeepSeek V4.1 Flash | $0.30 | $1.20 | $0.006 |
| GLM 5.3 Flash | $0.15 | $0.50 | $0.03 |

The DeepSeek V4.1 Flash case: 100k input + 10k output costs **$0.042**. If 100k were cache reads and only 10k were new input, input would cost **$0.0036** and the total with 10k output would be **$0.0156**. It is an order of magnitude below Opus. Quality is not the same; the decision is a product decision, not an accounting one.

### 4.3. Plans

- **OpenCode Go**: $10/month; Go Plus: $40/month. Per-model limits: 20% of the monthly limit per 5-hour window, 50% weekly, 100% monthly.
- **Console/Zen**: pay-as-you-go, with top-ups and workspace budgets.
- **Free models**: some models are free for a limited time, with different data terms (some may use your prompts to improve the model). Do not use them with private code without reading the policy.
- **Your own API key**: you pay the provider; OpenCode adds no margin (except the Console gateway).

### 4.4. Where to see it

OpenCode shows session and per-message cost in the TUI; `opencode stats` aggregates tokens and estimated cost. There is no per-subagent attribution as direct as Claude Code's; for fine analysis, export sessions or instrument the provider.

### 4.5. The hidden cost of migration

v2 is not an incremental update: v1 plugins do not run, the permission format changes, `CLAUDE.md` is no longer discovered and, according to the migration guide, `external_directory` is allowed by default. A badly executed migration can **save tokens and lose security**, or **load the agent without rules** and produce more rework. The cost of migration is not the token; it is the behaviour that is lost.

## 5. Recipes

### 5.1. v2 reviewer

```markdown
---
description: Reviews changes for correctness, security and test gaps
mode: subagent
model: anthropic/claude-sonnet-5-5#high
permissions:
  - action: edit
    resource: "*"
    effect: deny
  - action: shell
    resource: "*"
    effect: deny
  - action: read
    resource: "**"
    effect: allow
steps: 20
---

Review the current diff. Report only actionable findings with file and line references.
Do not edit files. If the diff is empty, say so.
```

Why it saves: without edit or shell, the agent cannot wander; `steps` caps the loop; Sonnet is 2× cheaper than Opus.

### 5.2. v1 explorer

```markdown
---
description: Fast read-only codebase exploration
mode: subagent
model: deepseek/deepseek-v4.1-flash
temperature: 0.1
permission:
  edit: deny
  bash: deny
  webfetch: deny
---

Search for the requested symbol or behavior. Return file paths, line numbers and a
short explanation. Do not propose patches.
```

Why it saves: cheap model, no shell, no edits, bounded output.

### 5.3. v2 orchestrator

```jsonc
{
  "agents": {
    "orchestrator": {
      "description": "Coordinates a small roster of specialists",
      "mode": "primary",
      "model": "anthropic/claude-opus-5-5",
      "permissions": [
        { "action": "subagent", "resource": "*", "effect": "deny" },
        { "action": "subagent", "resource": "repo-scout", "effect": "allow" },
        { "action": "subagent", "resource": "reviewer", "effect": "allow" }
      ]
    }
  }
}
```

Why it saves: the orchestrator cannot launch any agent; the catalogue is reduced to two. The allowlist is also a cost prune.

### 5.4. Background for independent work

```text
Launch the repo-scout subagent in the background to map the auth flow.
While it runs, I will review the failing tests. Notify me when it finishes.
```

Why it saves: the parent does not wait; the subagent does not block. It does not save tokens, it saves wall-clock time.

## 6. The savings playbook

1. **Declare every subagent's model.** Without `model`, it inherits the parent's; an Opus subagent is expensive by accident.
2. **Use `steps`.** A subagent without a limit can explore until the window is exhausted. 15–25 steps is usually enough to explore or review.
3. **Use the cheap provider.** DeepSeek/GLM/Flash for exploration; Sonnet/Opus for decisions. The difference is 10–40×.
4. **Hide internal agents.** `hidden: true` removes them from autocomplete, not from Task; it reduces selection noise.
5. **Restrict `subagent` by permissions.** An orchestrator with an allowlist prevents the model from launching 20 agents out of enthusiasm.
6. **Migrate to v2 with AGENTS.md.** v2 only discovers `AGENTS.md`; if your repo only has `CLAUDE.md`, the agent runs without rules. Copy or link it.
7. **Review `external_directory`.** According to the migration guide, v2 allows it by default; if you do not want it, add an `ask` rule.
8. **Measure the session, not the lone token.** A child session's cost includes its startup; `opencode stats` gives the aggregate, not fine attribution.

## 7. Traps

- **v1 format in v2.** `permission`, `temperature`, `tools` and `prompt` are deprecated; the file may load with ignored fields. Use `permissions`, `request` and `system`.
- **Invisible `CLAUDE.md`.** According to the migration guide, v2 does not discover it. Without `AGENTS.md`, the agent has no project rules.
- **Open `external_directory`.** The agent can read/write outside the project without asking. Add the rule.
- **Broken plugins.** v1 does not run in v2; the port is code, not a name.
- **Missing LSP.** According to the migration guide, v2 keeps the configuration but does not run language servers; if you relied on diagnostics, add lint/test to the loop.
- **Opaque per-subagent cost.** The session shows cost; per-agent attribution requires instrumentation.
- **Free models with data terms.** The price may be your code. Read the terms.
- **Haiku threshold.** >100k prompt multiplies the price by 5; a large subagent is not cheap.

## 8. Relevance for xscriptor

xscriptor's catalogue is written in OpenCode v1 format and the environment already runs v2.1.4: conversion is not optional. Three actions:

1. **Migrate the frontmatter to `permissions`** (array) and `agents` (plural); `permission`, `temperature` and `tools` are deprecated.
2. **Add `AGENTS.md`** next to every `CLAUDE.md`; v2 does not discover the latter.
3. **Assign a model tier**: exploration agents can use DeepSeek/GLM/Flash; decision agents, Sonnet/Opus. The catalogue wins if the tier lives in the IR, not hardcoded across 205 files.

## 9. Closing

OpenCode is the most agnostic of the three and the most demanding to configure. Its permission model is expressive, its model catalogue is broad, and its format changed enough that migration is a project, not an `upgrade`. The saving is in the combination: a cheap model to explore, a narrow permission to prevent wandering, and a `steps` limit to stop the subagent from becoming an infinite session.

The open door is a virtue and a risk. The rule you write is the price.

---

**Next:** [Agnostic conversion](06-agnostic-conversion.md) — what can be ported, what is lost and how to design the converter.
