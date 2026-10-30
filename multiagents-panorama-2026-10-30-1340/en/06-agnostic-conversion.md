---
title: "Agnostic conversion — what can be ported and what is lost"
description: "AGENTS.md, skills and MCP are the common base; agents are not. An IR schema, three adapters and four parity levels."
date: "2026-10-30"
tags: ["technology", "agents", "conversion", "standards"]
---

# Agnostic conversion — what can be ported and what is lost

> *An agent is not a file: it is a file plus the way its tool decides to invoke it. Copying the file is easy; copying the decision is something else.*

## 1. The question

Can you write an agent once and use it in Claude Code, Codex and OpenCode?

**Yes, if you accept that “use it” means different things in each tool.** You can make the agent **load** and its **main capabilities** present. You cannot guarantee that it **decides the same way**, **costs the same** or **respects the same permissions** without a translation layer and an evaluation.

The reason is that the three tools share the surface — Markdown, YAML, TOML, instructions — but not the semantics:

| Layer | Common? | What changes |
|---|---|---|
| Project instructions | **Yes** | `AGENTS.md`; Claude Code reads it if there is no `CLAUDE.md`; OpenCode v2 only reads `AGENTS.md` |
| Skills | **Almost** | `SKILL.md` is a standard, but frontmatter has per-tool extensions |
| Tools | **Partial** | MCP is common; native tools have different names and permissions |
| Agent definition | **No** | Markdown+YAML vs TOML vs Markdown+YAML with another schema |
| Invocation | **No** | `Agent` vs orchestrator spawn vs Task/subagent |
| Permissions | **No** | Tools/permissionMode vs inherited sandbox vs action/resource/effect |
| Cache | **No** | Explicit TTL vs automatic vs provider-dependent |
| Models | **No** | Different names, prices and tokenizers |

## 2. The common base

### 2.1. AGENTS.md

`AGENTS.md` is an open instruction format for agents, used by more than **60,000 projects** and stewarded by the Agentic AI Foundation under the Linux Foundation ([agents.md](https://agents.md/)). It is read by Codex, OpenCode, Cursor, Copilot, Gemini CLI, Zed, Warp, Aider, Goose and others.

Claude Code joined: if the repository has `AGENTS.md` and no `CLAUDE.md` or `CLAUDE.local.md` in the working directory or above, it reads `AGENTS.md`. If `CLAUDE.md` exists, `CLAUDE.md` wins; if `CLAUDE.md` imports `AGENTS.md`, both are read ([Claude Code memory](https://code.claude.com/docs/en/memory)). OpenCode v1 reads `AGENTS.md` and uses `CLAUDE.md` as a fallback; OpenCode v2, according to the migration guide, only discovers `AGENTS.md`.

Practical consequence: **`AGENTS.md` is the only layer you can share without conversion**. Everything else needs an adapter.

### 2.2. Skills

`SKILL.md` follows the Agent Skills standard and is consumed by all three tools. But:

- Claude Code adds fields such as `context: fork`, `agent`, `disable-model-invocation`, `allowed-tools`, `paths`, `model`, `effort`, `hooks`.
- Codex can disable skills per agent (`skills.config`) and control the instruction block.
- OpenCode has its own permission model for `skill` and load paths.

A basic `SKILL.md` (`name`, `description`, body, `references/`) travels well. A skill with `context: fork` or `hooks` does not.

### 2.3. MCP

MCP is the most common ground for tools. But configuration is not portable: Claude Code has `.mcp.json`, per-agent `mcpServers` and `mcp__server__*` permissions; Codex configures MCP on the parent and subagents inherit it (it cannot be defined per agent); OpenCode has its own `mcp` block and Code Mode in v2.

## 3. The three agent formats

### 3.1. Claude Code

Markdown + YAML. Relevant fields: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `omitClaudeMd`, `effort`, `isolation`, `color`, `experimental.cacheTtl`.

### 3.2. Codex

TOML, one file per agent. Fields: `name`, `description`, `developer_instructions`, `model`, `model_reasoning_effort`, `model_reasoning_summary`, `model_verbosity`, `features.*`, `skills.config`.

### 3.3. OpenCode

Markdown + YAML (or JSONC). v1: `description`, `mode`, `model`, `temperature`, `top_p`, `steps`, `permission`, `tools`, `hidden`, `color`, `disable`. v2: `description`, `mode`, `model`, `permissions`, `system`, `steps`, `hidden`, `disabled`, `color`, `request`.

## 4. The compatibility matrix

| Capability | Claude Code | Codex | OpenCode v1 | OpenCode v2 |
|---|---|---|---|---|
| `AGENTS.md` | Yes (if no `CLAUDE.md`) | Yes | Yes (fallback `CLAUDE.md`) | Yes (only `AGENTS.md`) |
| `SKILL.md` | Yes | Yes | Yes | Yes |
| Subagent | Yes | Yes | Yes | Yes |
| Nesting | Yes (3 layers) | Yes (orchestrator) | Limited/opt-in | By permissions; `general` cannot |
| Model per agent | Yes | Yes | Yes | Yes |
| Tools per agent | Allow/deny | Disable features only | Permissions | Permissions |
| Per-command permissions | Yes (`Bash(...)`) | No per agent | Yes (glob) | Yes (resource/effect) |
| Sandbox per agent | `permissionMode`/`isolation` | **No**; inherits | No | No |
| MCP per agent | Yes | **No**; inherits | Global config | Global config |
| Hooks per agent | Yes | Global hooks | Plugins | Plugins |
| Configurable cache | `cacheTtl` | Automatic | Provider | Provider |
| Commands | Skills/commands | Skills | Commands | Commands |
| Cost per agent | `/usage`, JSONL | `/status`, logs | Session | Session |

The OpenCode v2 `AGENTS.md` row follows the migration guide; the official v2 rules page was not available on the reading date. Verify it on your version.

The table shows why a converter cannot be a simple key substitution: **some capabilities do not exist in the destination**. The correct conversion is not to invent them; it is to declare the loss.

## 5. An IR schema

A proposed intermediate representation, tool-independent. It is not an official standard; it is a contract for the converter.

```yaml
version: 1
id: repo-scout
name: repo-scout
description: Read-only codebase exploration. Use before implementing in an unfamiliar area.
role: subagent            # primary | subagent | all
model:
  tier: fast              # frontier | balanced | fast
  target_ids:             # optional; the adapter decides whether to use it
    claude: haiku
    codex: gpt-6-luna
    opencode: deepseek/deepseek-v4.1-flash
system: |
  Search broadly. Return paths, symbols and risks. Do not edit files.
tools:
  allow: [read, grep, glob]
  deny: [write, edit, shell, web]
permissions:
  edit: deny
  shell: deny
  network: deny
  subagent: deny
  external_directory: ask
skills: []
mcp_servers: []
limits:
  max_turns: 25
  steps: 25
  token_budget: 120000
isolation: none           # none | worktree
cache:
  ttl: 5m
delegation:
  can_spawn: false
  allowed_agents: []
  max_depth: 0
metadata:
  source: xscriptor/agents
  source_format: opencode-v1
  license: MIT
```

The IR does not try to capture everything. It captures what can be mapped unambiguously and makes the rest explicit.

## 6. The mapping table

| IR | Claude Code | Codex | OpenCode v2 |
|---|---|---|---|
| `name` | `name` | `name` | filename / ID |
| `description` | `description` | `description` | `description` |
| `system` | body | `developer_instructions` | body / `system` |
| `role: subagent` | file in `agents/` | file in `agents/` | `mode: subagent` |
| `role: primary` | `--agent` / settings | main thread | `mode: primary` |
| `model.tier: fast` | `haiku` | `gpt-6-luna` | `.../flash` or `.../haiku` |
| `model.tier: balanced` | `sonnet` | `gpt-6.1-terra` | `.../sonnet` |
| `model.tier: frontier` | `opus` | `gpt-6.1-sol` | `.../opus` |
| `tools.allow` | `tools` | no equivalent; `features.*` | `permissions` allow |
| `tools.deny` | `disallowedTools` | `features.* = false` | `permissions` deny |
| `permissions.edit` | `tools`/`permissionMode` | not per agent | `action: edit` |
| `permissions.shell` | `Bash` + rules | `features.shell_tool` | `action: shell` |
| `permissions.subagent` | `Agent` allow/deny | orchestrator | `action: subagent` |
| `limits.max_turns` | `maxTurns` | not per agent | `steps` |
| `isolation: worktree` | `isolation: worktree` | parent worktree | no direct |
| `cache.ttl` | `experimental.cacheTtl` | automatic | provider |
| `delegation.can_spawn` | `tools: Agent` | orchestrator | `permissions: subagent` |
| `skills` | `skills` (preload) | `skills.config` | project skills |
| `mcp_servers` | `mcpServers` | inherited | global config |

### 6.1. Model mapping

Do not map exact IDs. Map **tiers** and let the adapter resolve:

| Tier | Claude | Codex | OpenCode |
|---|---|---|---|
| `fast` | Haiku 5.5 | GPT-6 Luna | DeepSeek V4.1 Flash / GLM 5.3 Flash / Haiku 5.5 |
| `balanced` | Sonnet 5.5 | GPT-6.1 Terra | Sonnet 5.5 / GPT-6.1 Terra |
| `frontier` | Opus 5.5 / Fable 5.1 | GPT-6.1 Sol / Astra | Opus 5.5 / GPT-6.1 Sol |

A `model.target_ids` allows per-tool overrides without breaking the IR. If the target does not have the tier, the adapter must fail or warn, not silently degrade.

### 6.2. Permission mapping

This is the most dangerous part. A converter can **widen** permissions without meaning to:

- OpenCode `edit: deny` + `bash: ask` → Claude without `tools` → inherits **all** tools, including `Agent` and `Write`.
- Codex does not allow `sandbox_mode` per agent; a TOML that says “do not edit” still has the parent sandbox.
- Claude `permissionMode: plan` is not equivalent to OpenCode `edit: deny`; the first is a session mode, the second a tool rule.

Golden rule: **if the destination cannot express a restriction, the converter must deny by default or mark the agent as non-convertible**, never widen.

## 7. What is lost

| Loss | Why | Mitigation |
|---|---|---|
| `description` routing | Each model interprets the text differently | L3 evaluation with real tasks |
| Fine-grained permissions | Different semantics | Manifest + deny by default |
| Exact model | Different names and versions | Tiers + override |
| Cache | Different TTL and prefixes | Not portable; document it |
| Hooks/plugins | Different APIs | Reimplement per tool |
| Per-agent MCP | Codex does not allow it | Move to parent or accept the loss |
| Cost | Different prices and quotas | Measure L4 per tool |
| Startup context | `CLAUDE.md` vs `AGENTS.md` vs rules | `omitClaudeMd`/AGENTS.md per adapter |

## 8. Prior art

### 8.1. Rulesync

`rulesync` is a CLI that generates configuration for more than 40 tools from a unified `.rulesync/`. It supports rules, commands, MCP, subagents, skills, hooks and permissions for Claude Code, Codex CLI, OpenCode and OpenCode V1, among others. It has `convert --from cursor --to copilot,claudecode` for direct conversions without adopting the `.rulesync/` workflow ([rulesync](https://github.com/dyoshikawa/rulesync)).

It is proof that multi-tool conversion is viable. It is also proof of its limit: the support table marks which features exist in each target; what does not exist is not generated.

### 8.2. Codex `/import`

Codex can import from Claude Code or Cursor:

- instruction files → `AGENTS.md`;
- `settings.json` → `config.toml`;
- skills, plugins, projects, memories, recent chats, MCP, hooks and commands;
- **subagents** → Codex subagents.

The documentation warns you to review tool restrictions, permissions, MCP authentication, hooks and prompt templates. It is a migration, not semantic synchronisation; the app can enable “automatic updates”, but the CLI imports once.

### 8.3. OpenCode

OpenCode has a migration ecosystem from Claude Code and a v2 format that breaks with v1. Its migration guide is, in practice, a manual converter: rename keys, port plugins, move `CLAUDE.md` to `AGENTS.md`, translate permissions.

### 8.4. xscriptor

The `agents` repo already has a converter: `claude/tools/xscriptor-convert.py`. It is a real case and useful for studying the limits.

**What it does well:**

- Parses OpenCode v1 frontmatter.
- Emits `name`, `description`, `color` and, for read-only agents, a tool allowlist.
- Rewrites skill references (`senior/python` → `senior-python`).
- Copies skills and `references/`.
- Converts commands and, if they have `agent`, wraps them to delegate via Task.
- Is idempotent and has a `--dry` mode.

**What it does not do:**

- **Does not emit Codex TOML.**
- **Does not emit OpenCode v2** (`permissions` array, `agents` plural).
- **Does not round-trip** or validate loading.
- **Does not preserve the model** (only one agent in the repo declares `model`; the rest inherit).
- **Does not add `maxTurns`/`steps`** (they do not exist in the source, but the destination would benefit from an explicit limit).
- **Does not preserve the `task` allowlist**; an agent with `task: allow` gets `Task`/`Agent` with no type restriction (Claude Code renamed `Task` to `Agent` in 2.1.63; the alias still works).
- **Does not preserve `temperature`, `top_p`** (Claude does not support them; that is an acceptable loss).
- **Widens permissions on editable agents**: if `edit: allow` and `bash: ask`, it emits no `tools`, so the Claude subagent inherits all tools, including `Write`, `Edit`, `Agent` and MCP. The source asked for `bash: ask`; the destination does not ask.

**The repo numbers:** 205 agents, 142 with `edit: allow`, 105 with `task: allow`, 202 with `webfetch: allow`, ~17,768 characters of descriptions (~4,400 tokens). The converter is correct for its goal (L1: they load), but it does not guarantee L2: capabilities and restrictions are not equivalent.

## 9. Designing the converter

### 9.1. Pipeline

```text
source (v1/v2/Claude/Codex)
   │
   ├─ parse ──► IR ──► validate(IR)
   │                     │
   │                     ├─► adapter Claude ──► .claude/agents/*.md
   │                     ├─► adapter Codex  ──► .codex/agents/*.toml
   │                     └─► adapter OpenCode ─► .opencode/agents/*.md
   │
   └─ manifest.json (mapped, lost, warnings)
          │
          └─► load test (each tool loads the agent)
                 │
                 └─► parity eval (L3/L4)
```

### 9.2. Manifest

Every conversion should produce a `manifest.json`:

```json
{
  "source": "opencode-v1",
  "target": "claude",
  "agent": "repo-scout",
  "status": "partial",
  "mapped": ["name", "description", "system", "tools.allow", "model.tier"],
  "lost": ["temperature", "permission.bash.patterns", "steps"],
  "warnings": [
    "bash: ask has no exact equivalent; Bash is emitted without patterns",
    "task: allow is emitted as Agent without an allowlist"
  ],
  "requires_review": true
}
```

Without a manifest, the user does not know what changed. With a manifest, the loss is a decision, not an accident.

### 9.3. Proposed CLI

```bash
# convert the whole catalogue to all three targets
xscriptor-convert convert \
  --from opencode-v1 \
  --to claude,codex,opencode-v2 \
  --src ./agents \
  --out ./dist \
  --manifest

# validate that each target loads
xscriptor-convert validate --dist ./dist --target all

# compare behaviour and cost
xscriptor-convert eval \
  --suite ./evals/tasks.yaml \
  --baseline claude \
  --candidate codex \
  --report ./reports/parity.json
```

### 9.4. Conversion rules

1. **Deny by default when the destination cannot express the restriction.** Never widen.
2. **Map model tiers, not IDs.** Overrides are explicit.
3. **Preserve the `description` text**; it is the routing signal. If translated, evaluate again.
4. **Mark every lost field** in the manifest.
5. **Validate loading** in the target tool, not just parsing.
6. **Evaluate L3/L4** before declaring parity.
7. **Version the IR and the adapters**; a format change breaks the promise.

## 10. Parity levels and acceptance criteria

| Level | Criterion | Test |
|---|---|---|
| **L0** | Complete inventory | `convert --dry` lists every agent |
| **L1** | Loads | `claude plugin validate`, `opencode debug agents` (v2; `opencode agent list` on older versions), TOML parse for Codex |
| **L2** | Capabilities | Tools/permissions/model/skills matrix with no widening |
| **L3** | Behaviour | 20 tasks; same delegation decision ≥80%; quality not worse |
| **L4** | Economics | Cost per accepted task within ±20% |

L1 is cheap. L2 requires a manifest and a review. L3 requires an evaluation set and repeated runs. L4 requires instrumentation and current prices.

## 11. Verdict

**Agnostic conversion is possible as infrastructure, not as a promise.** You can write once, translate to three formats and validate that they load. You can preserve the description, the prompt, the model tier and the main capabilities. You cannot guarantee that the agent decides the same way, costs the same or respects the same limits without evaluation.

For xscriptor, the recommendation is concrete:

1. **Keep the catalogue in OpenCode v1/v2 as the source**, but define an IR.
2. **Write adapters for Claude, Codex and OpenCode v2**, not a monolithic script.
3. **Emit a manifest and validate loading** on every release.
4. **Measure L3/L4 with 20 tasks** before declaring the roster portable.
5. **Score agents by risk**: those that widen permissions are not converted without review.

The current converter is a good start and a warning: translating keys is easy; translating decisions is not.

---

**Next:** [Measurement and playbook](07-measurement-and-playbook.md) — how to measure L3/L4 and what to do on Monday.
