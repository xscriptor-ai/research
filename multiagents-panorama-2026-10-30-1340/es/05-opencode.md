---
title: "OpenCode — dos formatos, permisos y sesiones hijas"
description: "Cómo delega OpenCode en v1 y v2, qué cambia en el modelo de permisos y dónde está el ahorro real."
date: "2026-10-30"
tags: ["tecnología", "opencode", "agentes", "tokens"]
---

# OpenCode — dos formatos, permisos y sesiones hijas

> *OpenCode deja la puerta abierta y te pide que escribas la regla. La libertad es el formato; el coste, la regla que olvides.*

## 1. Mapa de la herramienta

OpenCode distingue dos tipos de agente:

- **Primary**: el asistente con el que hablas; `build` y `plan` son los integrados.
- **Subagent**: un agente que el principal lanza para una tarea; `general` y `explore` son los integrados.

Un agente se define en Markdown o JSONC. El nombre del archivo es el nombre del agente (salvo que el formato permita un `name` explícito). Los subagentes corren en **sesiones hijas** con contexto fresco, en primer o segundo plano; el padre recibe el resultado.

La versión instalada en este entorno es **OpenCode v2.1.4**, pero el ecosistema xscriptor y muchos ejemplos públicos siguen en formato v1. El dossier documenta los dos porque la conversión entre ellos ya es un problema económico: un agente que no carga es un agente que no ahorra.

## 2. Los dos formatos

### 2.1. v1 (el formato histórico)

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

Campos habituales: `description`, `mode`, `model`, `temperature`, `top_p`, `steps`, `permission`, `tools` (deprecado), `hidden`, `color`, `disable`.

### 2.2. v2 (el formato actual)

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

Campos: `description`, `mode`, `model` (`provider/model#variant`), `permissions` (lista ordenada de `action`/`resource`/`effect`), `system`, `steps`, `hidden`, `disabled`, `color`, `request` (headers/body por agente).

### 2.3. Tabla de traducción v1 → v2

| v1 | v2 | Nota |
|---|---|---|
| `permission` (mapa) | `permissions` (array) | El orden importa; la última regla que coincide gana |
| `tools` | `permissions` | `tools` está deprecado |
| `agent` (config) | `agents` | Plural |
| `provider` | `providers` | La ortografía v1 todavía carga |
| `mcp` con `enabled` | `mcp.servers` con `disabled` | El flag se invierte |
| `command` / `subtask` | `commands` / `subagent` | Renombrar |
| `skills.paths` + `skills.urls` | `skills` (array) | Unificar |
| `autoupdate` | `update` | `disable`/`notify`/`auto` |
| `plugin` | `plugins` | Requiere portar código |
| `tui.json` | `cli.json` | Los ajustes de proyecto no migran |
| `CLAUDE.md` | `AGENTS.md` | v2 sólo descubre `AGENTS.md` |
| `bash` | `shell` | Acción de permiso |
| `task` | `subagent` | Acción de permiso |
| `write` / `patch` | `edit` | Acción de permiso |

La tabla anterior combina la documentación oficial de v1/v2 con la guía de migración de terceros ([Shattered, 27/10/2026](https://shattered.io/opencode-2-upgrade-from-1x-12-steps-2026/)); las filas de permisos y descubrimiento de `CLAUDE.md` conviene verificarlas en la versión exacta que uses. En este entorno, v2.1.4, la dirección es la que se describe.

## 3. Cómo se invoca

1. **Automático**: el principal lee las descripciones y lanza el subagente adecuado.
2. **Manual**: `@general help me search for this function`, o el nombre del agente.
3. **Por permisos**: `permission.task` (v1) o `permissions` con `action: subagent` (v2) deciden qué agentes puede lanzar cada uno.
4. **En segundo plano**: v2 permite lanzar el subagente de forma asíncrona y seguir trabajando; el padre recibe el resultado al terminar.
5. **Navegación**: las sesiones hijas se recorren con atajos (`session_child_first`, `session_child_cycle`, `session_parent`).

El subagente usa su modelo configurado o, si no hay ninguno, **hereda el del padre**. Eso es una trampa de coste: un subagente sin `model` en una sesión Opus es un subagente Opus.

### 3.1. Anidamiento

OpenCode v2 documenta que el `general` integrado **no puede lanzar más subagentes**. La delegación de subagente a subagente existe en el modelo de permisos (un subagente con `action: subagent` permitido puede lanzar otro), pero no es el comportamiento por defecto ni el camino recomendado. En v1, la delegación anidada fue una función opt-in en el pasado; no la asumas disponible sin comprobarla.

La consecuencia económica: OpenCode es excelente para **un nivel** de delegación; para árboles profundos, Claude Code y Codex ofrecen una semántica más explícita.

## 4. El coste

### 4.1. Modelo por agente

OpenCode usa `provider/model#variant`. El modelo se elige por agente; si falta, hereda. Ejemplos:

```yaml
# explorador barato
model: deepseek/deepseek-v4.1-flash
# revisor
model: anthropic/claude-sonnet-5-5#high
# orquestador
model: anthropic/claude-opus-5-5
```

El ahorro por enrutamiento es el mismo que en las otras herramientas: 20–40× entre un modelo barato y uno caro. OpenCode añade una ventaja: puedes mezclar proveedores en el mismo equipo (Anthropic, OpenAI, DeepSeek, GLM, Kimi, local), y el precio depende del proveedor, no del cliente.

### 4.2. Precios de referencia

Con OpenCode Console (30/10/2026):

| Modelo | Entrada | Salida | Lectura caché |
|---|---:|---:|---:|
| Claude Opus 5.5 | $4,00 | $20,00 | $0,20 |
| Claude Sonnet 5.5 | $2,00 | $10,00 | $0,10 |
| Claude Haiku 5.5 (≤100k) | $0,10 | $0,50 | $0,01 |
| GPT-6.1 Sol (≤272k) | $2,00 | $10,00 | $0,10 |
| DeepSeek V4.1 Flash | $0,30 | $1,20 | $0,006 |
| GLM 5.3 Flash | $0,15 | $0,50 | $0,03 |

El caso DeepSeek V4.1 Flash: 100k de entrada + 10k de salida cuesta **$0,042**. Si 100k fueran lectura de caché y sólo 10k entrada nueva, la entrada costaría **$0,0036** y el total con 10k de salida, **$0,0156**. Es un orden de magnitud por debajo de Opus. La calidad no es la misma; la decisión es de producto, no de contabilidad.

### 4.3. Planes

- **OpenCode Go**: $10/mes; Go Plus: $40/mes. Límites por modelo: 20% del mensual por ventana de 5 horas, 50% semanal, 100% mensual.
- **Console/Zen**: pago por uso, con recarga y presupuestos por workspace.
- **Modelos free**: algunos modelos son gratuitos por tiempo limitado, con términos de datos distintos (algunos pueden usar tus prompts para mejorar el modelo). No los uses con código privado sin leer la política.
- **Tu propia API key**: pagas al proveedor; OpenCode no añade margen (salvo la pasarela Console).

### 4.4. Dónde se ve

OpenCode muestra el coste por sesión y por mensaje en la TUI; `opencode stats` agrega tokens y coste estimado. No hay una atribución por subagente tan directa como la de Claude Code; para un análisis fino, exporta las sesiones o instrumenta el proveedor.

### 4.5. El coste oculto de la migración

v2 no es una actualización incremental: los plugins de v1 no corren, el formato de permisos cambia, `CLAUDE.md` deja de descubrirse y, según la guía de migración, `external_directory` pasa a estar permitido por defecto. Una migración mal hecha puede **ahorrar tokens y perder seguridad**, o **cargar el agente sin reglas** y producir más retrabajo. El coste de la migración no es el token; es el comportamiento que se pierde.

## 5. Recetas

### 5.1. Revisor v2

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

Por qué ahorra: sin edición ni shell, el agente no puede divagar; `steps` acota el bucle; Sonnet es 2× más barato que Opus.

### 5.2. Explorador v1

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

Por qué ahorra: modelo barato, sin shell, sin edición, salida acotada.

### 5.3. Orquestador v2

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

Por qué ahorra: el orquestador no puede lanzar cualquier agente; el catálogo se reduce a dos. La allowlist es también una poda de coste.

### 5.4. Fondo para trabajo independiente

```text
Launch the repo-scout subagent in the background to map the auth flow.
While it runs, I will review the failing tests. Notify me when it finishes.
```

Por qué ahorra: el padre no espera; el subagente no bloquea. No ahorra tokens, ahorra tiempo de pared.

## 6. El playbook de ahorro

1. **Declara el modelo de cada subagente.** Sin `model`, hereda el del padre; un subagente Opus es caro por accidente.
2. **Usa `steps`.** Un subagente sin límite puede explorar hasta agotar la ventana. 15–25 pasos suele bastar para explorar o revisar.
3. **Aprovecha el proveedor barato.** DeepSeek/GLM/Flash para explorar; Sonnet/Opus para decidir. La diferencia es de 10–40×.
4. **Oculta los agentes internos.** `hidden: true` los quita del autocompletado, no del Task; reduce el ruido de selección.
5. **Restringe `subagent` por permisos.** Un orquestador con allowlist evita que el modelo lance 20 agentes por entusiasmo.
6. **Migra a v2 con AGENTS.md.** v2 sólo descubre `AGENTS.md`; si tu repo sólo tiene `CLAUDE.md`, el agente corre sin reglas. Copia o enlaza.
7. **Revisa `external_directory`.** Según la guía de migración, v2 lo permite por defecto; si no lo quieres, añade una regla `ask`.
8. **Mide la sesión, no el token suelto.** El coste de una sesión hija incluye su arranque; `opencode stats` da el agregado, no la atribución fina.

## 7. Trampas

- **Formato v1 en v2.** `permission`, `temperature`, `tools` y `prompt` están deprecados; el archivo puede cargar con campos ignorados. Usa `permissions`, `request` y `system`.
- **`CLAUDE.md` invisible.** Según la guía de migración, v2 no lo descubre. Sin `AGENTS.md`, el agente no tiene reglas del proyecto.
- **`external_directory` abierto.** El agente puede leer/escribir fuera del proyecto sin preguntar. Añade la regla.
- **Plugins rotos.** v1 no corre en v2; el puerto es de código, no de nombre.
- **LSP ausente.** Según la guía de migración, v2 conserva la configuración pero no ejecuta language servers; si dependías de diagnósticos, añade lint/test al bucle.
- **Coste por subagente opaco.** La sesión muestra coste; la atribución por agente requiere instrumentar.
- **Modelos free con datos.** El precio puede ser tu código. Lee los términos.
- **Umbral de Haiku.** >100k de prompt multiplica el precio por 5; un subagente grande no es barato.

## 8. Relevancia para xscriptor

El catálogo de xscriptor está escrito en formato OpenCode v1 y el entorno ya corre v2.1.4: la conversión no es opcional. Tres acciones:

1. **Migrar el frontmatter a `permissions`** (array) y `agents` (plural); `permission`, `temperature` y `tools` están deprecados.
2. **Añadir `AGENTS.md`** junto a cada `CLAUDE.md`; v2 no descubre el segundo.
3. **Asignar modelo por nivel**: los agentes de exploración pueden usar DeepSeek/GLM/Flash; los de decisión, Sonnet/Opus. El catálogo gana si el nivel está en el IR, no hardcodeado en 205 archivos.

## 9. Cierre

OpenCode es el más agnóstico de los tres y el más exigente con la configuración. Su modelo de permisos es expresivo, su catálogo de modelos es amplio y su formato cambió lo bastante como para que la migración sea un proyecto, no un `upgrade`. El ahorro está en la combinación: un modelo barato para explorar, un permiso estrecho para no divagar y un `steps` para no convertir el subagente en una sesión infinita.

La puerta abierta es una virtud y un riesgo. La regla que escribas es el precio.

---

**Siguiente:** [Conversión agnóstica](06-conversion-agnostica.md) — qué se puede portar, qué se pierde y cómo diseñar el conversor.
