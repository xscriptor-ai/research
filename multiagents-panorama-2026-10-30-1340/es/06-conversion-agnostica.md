---
title: "Conversión agnóstica — qué se puede portar y qué se pierde"
description: "AGENTS.md, skills y MCP son la base común; los agentes no. Un esquema IR, tres adaptadores y cuatro niveles de paridad."
date: "2026-10-30"
tags: ["tecnología", "agentes", "conversión", "estándares"]
---

# Conversión agnóstica — qué se puede portar y qué se pierde

> *Un agente no es un archivo: es un archivo más la forma en que su herramienta decide invocarlo. Copiar el archivo es fácil; copiar la decisión es otra cosa.*

## 1. La pregunta

¿Se puede escribir un agente una vez y usarlo en Claude Code, Codex y OpenCode?

**Sí, si aceptas que «usarlo» significa cosas distintas en cada herramienta.** Se puede conseguir que el agente **cargue** y que sus **capacidades principales** estén presentes. No se puede garantizar que **decida igual**, que **cueste lo mismo** o que **respete los mismos permisos** sin una capa de traducción y una evaluación.

La razón es que las tres herramientas comparten la superficie —Markdown, YAML, TOML, instrucciones— pero no la semántica:

| Capa | ¿Común? | Qué cambia |
|---|---|---|
| Instrucciones del proyecto | **Sí** | `AGENTS.md`; Claude Code lo lee si no hay `CLAUDE.md`; OpenCode v2 sólo lee `AGENTS.md` |
| Skills | **Casi** | `SKILL.md` es un estándar, pero el frontmatter tiene extensiones por herramienta |
| Herramientas | **Parcial** | MCP es común; las herramientas nativas tienen nombres y permisos distintos |
| Definición de agente | **No** | Markdown+YAML vs TOML vs Markdown+YAML con otro esquema |
| Invocación | **No** | `Agent` vs spawn del orquestador vs Task/subagent |
| Permisos | **No** | Tools/permissionMode vs sandbox heredado vs action/resource/effect |
| Caché | **No** | TTL explícito vs automático vs dependiente del proveedor |
| Modelos | **No** | Nombres, precios y tokenizadores distintos |

## 2. La base común

### 2.1. AGENTS.md

`AGENTS.md` es un formato abierto de instrucciones para agentes, con más de **60.000 proyectos** y custodiado por la Agentic AI Foundation bajo la Linux Foundation ([agents.md](https://agents.md/)). Lo leen Codex, OpenCode, Cursor, Copilot, Gemini CLI, Zed, Warp, Aider, Goose y otros.

Claude Code se sumó: si el repositorio tiene `AGENTS.md` y no tiene `CLAUDE.md` ni `CLAUDE.local.md` en el directorio de trabajo o por encima, lee `AGENTS.md`. Si hay `CLAUDE.md`, gana `CLAUDE.md`; si `CLAUDE.md` importa `AGENTS.md`, se leen ambos ([Claude Code memory](https://code.claude.com/docs/en/memory)). OpenCode v1 lee `AGENTS.md` y usa `CLAUDE.md` como fallback; OpenCode v2, según la guía de migración, sólo descubre `AGENTS.md`.

Consecuencia práctica: **`AGENTS.md` es la única capa que puedes compartir sin conversión**. Todo lo demás necesita adaptador.

### 2.2. Skills

`SKILL.md` sigue el estándar Agent Skills y lo consumen las tres herramientas. Pero:

- Claude Code añade campos como `context: fork`, `agent`, `disable-model-invocation`, `allowed-tools`, `paths`, `model`, `effort`, `hooks`.
- Codex permite deshabilitar skills por agente (`skills.config`) y controlar el bloque de instrucciones.
- OpenCode tiene su propio modelo de permisos para `skill` y rutas de carga.

Un `SKILL.md` básico (`name`, `description`, cuerpo, `references/`) viaja bien. Un skill con `context: fork` o `hooks` no.

### 2.3. MCP

MCP es el terreno más común para herramientas. Pero la configuración no es portable: Claude Code tiene `.mcp.json`, `mcpServers` por agente y permisos `mcp__server__*`; Codex configura MCP en el padre y los subagentes los heredan (no se pueden definir por agente); OpenCode tiene su propio bloque `mcp` y Code Mode en v2.

## 3. Los tres formatos de agente

### 3.1. Claude Code

Markdown + YAML. Campos relevantes: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `omitClaudeMd`, `effort`, `isolation`, `color`, `experimental.cacheTtl`.

### 3.2. Codex

TOML, un archivo por agente. Campos: `name`, `description`, `developer_instructions`, `model`, `model_reasoning_effort`, `model_reasoning_summary`, `model_verbosity`, `features.*`, `skills.config`.

### 3.3. OpenCode

Markdown + YAML (o JSONC). v1: `description`, `mode`, `model`, `temperature`, `top_p`, `steps`, `permission`, `tools`, `hidden`, `color`, `disable`. v2: `description`, `mode`, `model`, `permissions`, `system`, `steps`, `hidden`, `disabled`, `color`, `request`.

## 4. La matriz de compatibilidad

| Capacidad | Claude Code | Codex | OpenCode v1 | OpenCode v2 |
|---|---|---|---|---|
| `AGENTS.md` | Sí (si no hay `CLAUDE.md`) | Sí | Sí (fallback `CLAUDE.md`) | Sí (sólo `AGENTS.md`) |
| `SKILL.md` | Sí | Sí | Sí | Sí |
| Subagente | Sí | Sí | Sí | Sí |
| Anidamiento | Sí (3 capas) | Sí (orquestador) | Limitado/opt-in | Por permisos; `general` no |
| Modelo por agente | Sí | Sí | Sí | Sí |
| Herramientas por agente | Allow/deny | Sólo desactivar features | Permisos | Permisos |
| Permisos por comando | Sí (`Bash(...)`) | No por agente | Sí (glob) | Sí (resource/effect) |
| Sandbox por agente | `permissionMode`/`isolation` | **No**; hereda | No | No |
| MCP por agente | Sí | **No**; hereda | Config global | Config global |
| Hooks por agente | Sí | Hooks globales | Plugins | Plugins |
| Caché configurable | `cacheTtl` | Automática | Proveedor | Proveedor |
| Comandos | Skills/commands | Skills | Commands | Commands |
| Coste por agente | `/usage`, JSONL | `/status`, logs | Sesión | Sesión |

La fila de OpenCode v2 sobre `AGENTS.md` sigue la guía de migración; la página oficial de reglas v2 no estaba disponible en la fecha de lectura. Verifícala en tu versión.

La tabla muestra por qué un conversor no puede ser una simple sustitución de claves: **hay capacidades que no existen en el destino**. La conversión correcta no es inventarlas; es declarar la pérdida.

## 5. Un esquema IR

Propuesta de representación intermedia, independiente de la herramienta. No es un estándar oficial; es un contrato para el conversor.

```yaml
version: 1
id: repo-scout
name: repo-scout
description: Read-only codebase exploration. Use before implementing in an unfamiliar area.
role: subagent            # primary | subagent | all
model:
  tier: fast              # frontier | balanced | fast
  target_ids:             # opcional; el adaptador decide si lo usa
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

El IR no intenta capturar todo. Captura lo que se puede mapear sin ambigüedad y deja explícito lo que no.

## 6. La tabla de mapeo

| IR | Claude Code | Codex | OpenCode v2 |
|---|---|---|---|
| `name` | `name` | `name` | nombre del archivo / ID |
| `description` | `description` | `description` | `description` |
| `system` | cuerpo | `developer_instructions` | cuerpo / `system` |
| `role: subagent` | archivo en `agents/` | archivo en `agents/` | `mode: subagent` |
| `role: primary` | `--agent` / settings | hilo principal | `mode: primary` |
| `model.tier: fast` | `haiku` | `gpt-6-luna` | `.../flash` o `.../haiku` |
| `model.tier: balanced` | `sonnet` | `gpt-6.1-terra` | `.../sonnet` |
| `model.tier: frontier` | `opus` | `gpt-6.1-sol` | `.../opus` |
| `tools.allow` | `tools` | no equivalente; `features.*` | `permissions` allow |
| `tools.deny` | `disallowedTools` | `features.* = false` | `permissions` deny |
| `permissions.edit` | `tools`/`permissionMode` | no por agente | `action: edit` |
| `permissions.shell` | `Bash` + reglas | `features.shell_tool` | `action: shell` |
| `permissions.subagent` | `Agent` allow/deny | orquestador | `action: subagent` |
| `limits.max_turns` | `maxTurns` | no por agente | `steps` |
| `isolation: worktree` | `isolation: worktree` | worktree del padre | no directo |
| `cache.ttl` | `experimental.cacheTtl` | automática | proveedor |
| `delegation.can_spawn` | `tools: Agent` | orquestador | `permissions: subagent` |
| `skills` | `skills` (preload) | `skills.config` | skills del proyecto |
| `mcp_servers` | `mcpServers` | heredado | config global |

### 6.1. Mapeo de modelos

No mapees IDs exactos. Mapea **niveles** y deja que el adaptador resuelva:

| Nivel | Claude | Codex | OpenCode |
|---|---|---|---|
| `fast` | Haiku 5.5 | GPT-6 Luna | DeepSeek V4.1 Flash / GLM 5.3 Flash / Haiku 5.5 |
| `balanced` | Sonnet 5.5 | GPT-6.1 Terra | Sonnet 5.5 / GPT-6.1 Terra |
| `frontier` | Opus 5.5 / Fable 5.1 | GPT-6.1 Sol / Astra | Opus 5.5 / GPT-6.1 Sol |

Un `model.target_ids` permite overrides por herramienta sin romper el IR. Si el destino no tiene el nivel, el adaptador debe fallar o avisar, no degradar en silencio.

### 6.2. Mapeo de permisos

Es la parte más peligrosa. Un conversor puede **ampliar** permisos sin querer:

- OpenCode `edit: deny` + `bash: ask` → Claude sin `tools` → hereda **todas** las herramientas, incluida `Agent` y `Write`.
- Codex no permite `sandbox_mode` por agente; un TOML que pide «no edites» sigue teniendo el sandbox del padre.
- Claude `permissionMode: plan` no equivale a OpenCode `edit: deny`; el primero es un modo de sesión, el segundo una regla de herramienta.

Regla de oro: **si el destino no puede expresar una restricción, el conversor debe denegar por defecto o marcar el agente como no convertible**, nunca ampliar.

## 7. Lo que se pierde

| Pérdida | Por qué | Cómo se mitiga |
|---|---|---|
| Enrutamiento por `description` | Cada modelo interpreta el texto distinto | Eval L3 con tareas reales |
| Permisos finos | Semántica distinta | Manifiesto + denegar por defecto |
| Modelo exacto | Nombres y versiones distintos | Niveles + override |
| Caché | TTL y prefijos distintos | no portable; documentar |
| Hooks/plugins | APIs distintas | Reimplementar por herramienta |
| MCP por agente | Codex no lo permite | Mover al padre o aceptar pérdida |
| Coste | Precios y cuotas distintos | Medir L4 por herramienta |
| Contexto de arranque | `CLAUDE.md` vs `AGENTS.md` vs reglas | `omitClaudeMd`/AGENTS.md por adaptador |

## 8. Prior art

### 8.1. Rulesync

`rulesync` es un CLI que genera configuración para más de 40 herramientas desde un `.rulesync/` unificado. Soporta reglas, comandos, MCP, subagentes, skills, hooks y permisos para Claude Code, Codex CLI, OpenCode y OpenCode V1, entre otros. Tiene `convert --from cursor --to copilot,claudecode` para conversiones directas sin adoptar el flujo `.rulesync/` ([rulesync](https://github.com/dyoshikawa/rulesync)).

Es la prueba de que la conversión multiherramienta es viable. También es la prueba de su límite: la tabla de soporte marca qué features existen en cada target; lo que no existe, no se genera.

### 8.2. Codex `/import`

Codex puede importar desde Claude Code o Cursor:

- archivos de instrucciones → `AGENTS.md`;
- `settings.json` → `config.toml`;
- skills, plugins, proyectos, memorias, chats recientes, MCP, hooks y comandos;
- **subagentes** → subagentes de Codex.

La documentación advierte revisar restricciones de herramientas, permisos, autenticación de MCP, hooks y plantillas de prompt. Es una migración, no una sincronización semántica; en la app se puede activar «automatic updates», pero la CLI importa una vez.

### 8.3. OpenCode

OpenCode tiene un ecosistema de migración desde Claude Code y un formato v2 que rompe con v1. Su guía de migración es, en la práctica, un conversor manual: renombrar claves, portar plugins, mover `CLAUDE.md` a `AGENTS.md`, traducir permisos.

### 8.4. xscriptor

El repo `agents` ya tiene un conversor: `claude/tools/xscriptor-convert.py`. Es un caso real y útil para estudiar los límites.

**Qué hace bien:**

- Parsea el frontmatter de OpenCode v1.
- Emite `name`, `description`, `color` y, para agentes de sólo lectura, una allowlist de herramientas.
- Reescribe referencias a skills (`senior/python` → `senior-python`).
- Copia skills y `references/`.
- Convierte comandos y, si tienen `agent`, los envuelve para delegar vía Task.
- Es idempotente y tiene modo `--dry`.

**Qué no hace:**

- **No emite Codex TOML.**
- **No emite OpenCode v2** (`permissions` array, `agents` plural).
- **No hace round-trip** ni validación de carga.
- **No conserva el modelo** (sólo un agente del repo declara `model`; el resto hereda).
- **No añade `maxTurns`/`steps`** (no existen en el fuente, pero el destino se beneficiaría de un límite explícito).
- **No conserva la allowlist de `task`**; un agente con `task: allow` recibe `Task`/`Agent` sin restricción de tipos (Claude Code renombró `Task` a `Agent` en 2.1.63; el alias sigue funcionando).
- **No conserva `temperature`, `top_p`** (Claude no los soporta; es una pérdida aceptable).
- **Amplía permisos en agentes editables**: si `edit: allow` y `bash: ask`, no emite `tools`, así que el subagente Claude hereda todas las herramientas, incluidas `Write`, `Edit`, `Agent` y MCP. El fuente pedía `bash: ask`; el destino no pregunta.

**Los números del repo:** 205 agentes, 142 con `edit: allow`, 105 con `task: allow`, 202 con `webfetch: allow`, ~17.768 caracteres de descripciones (~4.400 tokens). El conversor es correcto para su objetivo (L1: que carguen), pero no garantiza L2: las capacidades y restricciones no son equivalentes.

## 9. Diseño del conversor

### 9.1. Pipeline

```text
fuente (v1/v2/Claude/Codex)
   │
   ├─ parse ──► IR ──► validate(IR)
   │                     │
   │                     ├─► adapter Claude ──► .claude/agents/*.md
   │                     ├─► adapter Codex  ──► .codex/agents/*.toml
   │                     └─► adapter OpenCode ─► .opencode/agents/*.md
   │
   └─ manifest.json (mapeado, perdido, warnings)
          │
          └─► load test (cada herramienta carga el agente)
                 │
                 └─► parity eval (L3/L4)
```

### 9.2. Manifiesto

Cada conversión debe producir un `manifest.json`:

```json
{
  "source": "opencode-v1",
  "target": "claude",
  "agent": "repo-scout",
  "status": "partial",
  "mapped": ["name", "description", "system", "tools.allow", "model.tier"],
  "lost": ["temperature", "permission.bash.patterns", "steps"],
  "warnings": [
    "bash: ask no tiene equivalente exacto; se emite Bash sin patrones",
    "task: allow se emite como Agent sin allowlist"
  ],
  "requires_review": true
}
```

Sin manifiesto, el usuario no sabe qué cambió. Con manifiesto, la pérdida es una decisión, no un accidente.

### 9.3. CLI propuesta

```bash
# convertir todo el catálogo a los tres destinos
xscriptor-convert convert \
  --from opencode-v1 \
  --to claude,codex,opencode-v2 \
  --src ./agents \
  --out ./dist \
  --manifest

# validar que cada destino carga
xscriptor-convert validate --dist ./dist --target all

# comparar comportamiento y coste
xscriptor-convert eval \
  --suite ./evals/tasks.yaml \
  --baseline claude \
  --candidate codex \
  --report ./reports/parity.json
```

### 9.4. Reglas de conversión

1. **Denegar por defecto cuando el destino no puede expresar la restricción.** Nunca ampliar.
2. **Mapear niveles de modelo, no IDs.** El override es explícito.
3. **Conservar el texto de `description`**; es la señal de enrutamiento. Si se traduce, evaluar de nuevo.
4. **Marcar cada campo perdido** en el manifiesto.
5. **Validar carga** en la herramienta destino, no sólo el parseo.
6. **Evaluar L3/L4** antes de declarar paridad.
7. **Versionar el IR** y los adaptadores; un cambio de formato rompe la promesa.

## 10. Niveles de paridad y criterios de aceptación

| Nivel | Criterio | Test |
|---|---|---|
| **L0** | Inventario completo | `convert --dry` lista todos los agentes |
| **L1** | Carga | `claude plugin validate`, `opencode debug agents` (v2; `opencode agent list` en versiones anteriores), parseo TOML de Codex |
| **L2** | Capacidades | Matriz de tools/permisos/modelo/skills sin ampliaciones |
| **L3** | Comportamiento | 20 tareas; misma decisión de delegación ≥80%; calidad no inferior |
| **L4** | Economía | Coste por tarea aceptada dentro de ±20% |

L1 es barato. L2 requiere un manifiesto y una revisión. L3 requiere un set de evaluación y ejecuciones repetidas. L4 requiere instrumentación y precios del día.

## 11. Veredicto

**La conversión agnóstica es posible como infraestructura, no como promesa.** Se puede escribir una vez, traducir a tres formatos y validar que cargan. Se puede conservar la descripción, el prompt, el nivel de modelo y las capacidades principales. No se puede garantizar que el agente decida igual, cueste lo mismo o respete los mismos límites sin evaluación.

Para xscriptor, la recomendación es concreta:

1. **Mantener el catálogo en OpenCode v1/v2 como fuente**, pero definir un IR.
2. **Escribir adaptadores para Claude, Codex y OpenCode v2**, no un script monolítico.
3. **Emitir manifiesto y validar carga** en cada release.
4. **Medir L3/L4 con 20 tareas** antes de declarar que el roster es portable.
5. **Puntuar los agentes por riesgo**: los que amplían permisos no se convierten sin revisión.

El conversor actual es un buen comienzo y una advertencia: traducir claves es fácil; traducir decisiones, no.

---

**Siguiente:** [Medición y playbook](07-medicion-y-playbook.md) — cómo medir L3/L4 y qué hacer el lunes.
