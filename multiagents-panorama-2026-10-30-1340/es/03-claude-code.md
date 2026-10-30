---
title: "Claude Code — subagentes, forks y el precio de un contexto limpio"
description: "Cómo se define, se invoca y se paga un subagente en Claude Code; qué campos importan y qué recetas ahorran de verdad."
date: "2026-10-30"
tags: ["tecnología", "claude-code", "agentes", "tokens"]
---

# Claude Code — subagentes, forks y el precio de un contexto limpio

> *Claude Code no delega para trabajar menos: delega para que el hilo principal no recuerde lo que no necesita. El recuerdo, aquí, se factura por turno.*

## 1. Mapa de la herramienta

Claude Code tiene cuatro mecanismos que se confunden entre sí:

| Mecanismo | Qué es | Contexto | Coste relativo |
|---|---|---|---|
| **Subagente** | Agente especializado con ventana propia, invocado con la herramienta `Agent` | Nuevo, salvo fork | Arranque + trabajo |
| **Fork** | Subagente que hereda la conversación del padre | Copia del padre | Comparte caché; más barato |
| **Equipo de agentes** | Varias sesiones de Claude Code coordinadas por un líder | Cada una independiente | ~7× una sesión estándar en modo plan |
| **Skill** | Instrucciones que se cargan bajo demanda en el hilo actual | El mismo hilo | Coste de carga, sin arranque de agente |

La diferencia clave: un subagente **no ve la conversación** (salvo fork), y un equipo **no comparte contexto**: comparte mensajes y una lista de tareas. Si el problema es «este hilo tiene demasiado ruido», subagente. Si es «necesito que varias personas discutan», equipo. Si es «necesito una instrucción reutilizable sin crear un agente», skill.

## 2. El archivo de agente

Los subagentes viven en Markdown con frontmatter YAML:

- `~/.claude/agents/` — personales, todos los proyectos.
- `.claude/agents/` — del proyecto, versionables.
- Plugins, managed settings y `--agents` JSON — otros ámbitos.

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

### 2.1. Campos que cambian el coste

| Campo | Qué hace | Efecto económico |
|---|---|---|
| `description` | El padre lo usa para decidir si delega | Se carga en el contexto principal; mantenlo corto |
| `tools` | Allowlist de herramientas | Menos esquemas = menos tokens por petición; menos divagación |
| `disallowedTools` | Denylist | Alternativa cuando quieres heredar casi todo |
| `model` | `haiku`, `sonnet`, `opus`, `fable`, ID o `inherit` | La palanca más grande: 2–40× por token |
| `permissionMode` | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan`, `manual` | Controla autonomía y riesgo, no tokens directamente |
| `maxTurns` | Corta el bucle | Evita exploraciones infinitas |
| `autoCompactWindow` | Baja el umbral de compactación | Menos contexto, más compactaciones |
| `skills` | Precarga skills completas | Ahorra exploración, pero carga tokens al inicio |
| `mcpServers` | MCPs sólo para este agente | Evita que sus esquemas ocupen el contexto principal |
| `hooks` | Hooks con alcance de agente | Puede filtrar salidas antes de que lleguen al modelo |
| `memory` | Memoria persistente (`user`, `project`, `local`) | Evita reexplorar; añade contexto |
| `background` | Mantiene el agente en segundo plano | Útil para trabajo independiente |
| `omitClaudeMd` | No carga `CLAUDE.md` de usuario/proyecto/local | Ahorra miles de tokens por petición |
| `effort` | `low`…`max` | Más esfuerzo = más tokens de razonamiento (salida) |
| `isolation: worktree` | Copia aislada del repo | Seguridad y paralelismo, no ahorro directo |
| `experimental.cacheTtl` | `5m` o `1h` | La caché de 1 h cuesta 2× la escritura |

Sólo `name` y `description` son obligatorios. Las descripciones de todos los subagentes se cargan en el contexto principal; cuando superan **15.000 tokens**, Claude Code avisa al arrancar. Es una señal de sprawl, no sólo de coste.

### 2.2. Lo que carga un subagente

Un subagente no-fork empieza con:

- su prompt de sistema (el cuerpo del archivo);
- el mensaje de delegación que escribe Claude;
- la jerarquía de `CLAUDE.md` (salvo `Explore`, `Plan` y `omitClaudeMd: true`);
- el snapshot de `git status`;
- las skills precargadas;
- las definiciones de herramientas permitidas.

Ese payload se escribe en caché desde cero. En una medición independiente de 19 subagentes, la primera petición escribió entre **12.427 y 40.763 tokens** y leyó **cero** de caché ([Tokenade, 25/10/2026](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0)). El dato no es una ley, pero da el orden de magnitud: **el arranque de un subagente son decenas de miles de tokens**.

### 2.3. Fork

Un fork hereda el prompt, las herramientas y la conversación del padre. Su primera petición **reutiliza la caché del padre**, así que es más barato que un subagente fresco para tareas que necesitan el mismo contexto. Se lanza con la herramienta `Agent` y `isolation`/fork según la versión, o desde una skill con `context: fork`. Límite: arrastra todo el contexto, incluido el ruido, y no puede lanzar más forks.

## 3. Cómo se invoca

1. **Automático**: Claude lee las descripciones y decide. Una descripción con trigger claro («use before commits», «when exploring a large codebase») enruta mejor que una genérica («security expert»).
2. **Explícito**: «Use the repo-scout agent to…», o `@agent` en la interfaz.
3. **Por comando/skill**: una skill con `context: fork` y `agent` ejecuta el trabajo en un subagente.
4. **Por `--agents` JSON**: definiciones efímeras para automatización.
5. **Anidado**: un subagente puede lanzar otro hasta **3 capas** por defecto (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`). En el límite, pierde la herramienta `Agent`.
6. **Concurrente**: por defecto, **20** subagentes a la vez (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`). El límite no es un presupuesto.

## 4. El coste, con nombres y números

### 4.1. Arranque

30.000 tokens de arranque, escritura de caché de 5 minutos:

| Modelo | Coste |
|---|---:|
| Haiku 5.5 (≤100k) | $0,00375 |
| Sonnet 5.5 | $0,075 |
| Opus 5.5 | $0,15 |

### 4.2. Punto de equilibrio

Con \(D=100k\), \(S=30k\), \(W=100k\), \(O=10k\), \(R=2k\):

| Principal → subagente | 10 turnos | 25 turnos | 50 turnos |
|---|---:|---:|---:|
| Opus → Sonnet | −$0,18 | +$0,12 | +$0,62 |
| Sonnet → Sonnet | −$0,28 | −$0,13 | +$0,12 |
| Opus → Haiku (>100k) | +$0,10 | +$0,40 | +$0,90 |

La conclusión práctica: **si el subagente va a hacer poco trabajo, no lo lances por dinero**. Si el hilo principal es Opus y el subagente puede ser Sonnet/Haiku, el enrutamiento paga antes que el aislamiento.

### 4.3. Dónde se ve

- `/usage` muestra el coste de la sesión, el uso por modelo y, en planes de suscripción, la atribución a skills, subagentes, plugins y MCP.
- La línea `Prompt cache (main)` muestra aciertos, fallos y si la caché está caliente.
- Las transcripciones están en `~/.claude/projects/<proyecto>/<sesión>/` y, para subagentes, en `subagents/agent-<id>.jsonl`. Cada mensaje del asistente lleva `usage` con `input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens` y `output_tokens`. **Deduplica por `message.id`**: una respuesta en streaming aparece en varias líneas con el mismo usage.
- OpenTelemetry exporta tokens y coste por usuario y sesión.

### 4.4. Suscripción

En Pro/Max, los subagentes consumen la misma ventana de 5 horas/semanal que la conversación principal. No hay conversión pública a dólares. La consecuencia es directa: **cinco subagentes en paralelo suman cinco contextos y cinco bucles, y pueden agotar la ventana mucho antes**, aunque el coste marginal en dólares sea cero hasta agotarla.

### 4.5. Equipos de agentes

Los equipos son otra cosa: sesiones independientes, lista de tareas compartida, mensajes entre pares. Anthropic reporta **~7× tokens** frente a una sesión estándar cuando los teammates corren en modo plan, y recomienda Sonnet para teammates, equipos pequeños y apagarlos al terminar. No son la opción barata; son la opción para trabajo que necesita discusión o ownership dividido.

## 5. Recetas

### 5.1. Explorador barato

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

Por qué ahorra: modelo 20–40× más barato, sin `CLAUDE.md`, tools mínimas, salida acotada.

### 5.2. Revisor independiente

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

Por qué ahorra: contexto limpio (no arrastra la implementación), permisos de sólo lectura, salida priorizada. El coste es el arranque; el beneficio es evitar un fallo.

### 5.3. Orquestador con allowlist

Cuando el agente corre como hilo principal (`claude --agent orchestrator`), la allowlist de tipos funciona:

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

En un subagente, listar `Agent` permite lanzar otros, pero la lista entre paréntesis se ignora; para restringir, usa `permissions.deny` o no incluyas `Agent`.

### 5.4. Skill con fork

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

El fork reutiliza la caché del padre; es la opción barata cuando la revisión necesita el contexto de la conversación.

## 6. El playbook de ahorro

Ordenado por impacto:

1. **Enruta el modelo.** `haiku` para explorar y resumir; `sonnet` para implementar y revisar; `opus` para decisiones arquitectónicas. La diferencia es 2–40×. Para forzar un modelo en todos los subagentes, combina `CLAUDE_CODE_SUBAGENT_MODEL` con `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1`.
2. **Recorta el arranque.** `omitClaudeMd: true` en agentes que reciben todo en el prompt; `tools` mínimas; `skills` sólo las necesarias; `mcpServers` fuera del hilo principal.
3. **Acota el bucle.** `maxTurns` y `autoCompactWindow`; un subagente que explora 40 turnos es una sesión larga disfrazada.
4. **Usa fork cuando el contexto es la tarea.** Comparte caché y evita el arranque fresco.
5. **Elige el TTL.** 5 minutos por defecto; 1 hora sólo si hay pausas largas y el trabajo continúa. La escritura de 1 h cuesta 2×.
6. **Poda el roster.** Menos agentes = menos descripciones, menos ambigüedad, mejor enrutamiento. 205 agentes no son 205 capacidades.
7. **Mide por tarea aceptada.** `/usage` da tokens; el equipo da éxito, retrabajo y minutos humanos.
8. **No cambies de modelo a mitad de tarea.** Invalida la caché y puede costar una relectura completa.

## 7. Trampas

- **Creer que un subagente ve la conversación.** No la ve (salvo fork). Si necesita un detalle de hace 20 turnos, lo buscará o lo adivinará; ambas cosas cuestan.
- **Descripciones largas.** Se cargan siempre. El aviso de 15k es el síntoma; el problema es la calidad del enrutamiento.
- **`CLAUDE.md` duplicado.** Cada subagente custom lo carga en cada petición. Si el subagente no lo necesita, `omitClaudeMd`.
- **Modelo heredado sin querer.** `Explore` hereda el modelo del padre (con tope en Opus en la API de Anthropic); un subagente sin `model` también. Si querías Haiku, decláralo.
- **Ventana del subagente.** La ventana la fija su modelo, no el padre. Un subagente en Haiku tiene menos ventana y puede necesitar más compactaciones.
- **Agentes de plugin.** No soportan `hooks`, `mcpServers` ni `permissionMode`; si los necesitas, copia el archivo a `.claude/agents/`.
- **Profundidad y concurrencia.** Un subagente que lanza subagentes multiplica el consumo; el límite de 20 no es un presupuesto.
- **Equipos como sinónimo de subagentes.** Los equipos son ~7× y experimental; úsalos cuando el trabajo necesite discusión, no cuando sólo necesite paralelismo.

## 8. Relevancia para xscriptor

El catálogo de xscriptor (205 agentes, 142 con `edit: allow`) es un caso ideal para Claude Code y un riesgo si se instala entero. Tres acciones concretas:

1. **Instalar un roster de 8–12 agentes por proyecto**, no los 205. Las descripciones se cargan siempre; el enrutamiento mejora cuando el modelo tiene pocas opciones.
2. **Convertir con modelo y límites**, no sólo con nombre y prompt. El conversor actual no conserva `maxTurns` ni la allowlist de `task`; añadirlos reduce el coste y el riesgo.
3. **Revisar los agentes editables.** Los que tienen `edit: allow` y `bash: ask` acaban en Claude con todas las herramientas; el fuente pedía preguntar. Ese es el punto donde la conversión cambia el coste y la seguridad.

## 9. Cierre

Claude Code ofrece la caja de herramientas más completa de las tres, y también la más fácil de sobreconfigurar. La diferencia entre un roster de ocho agentes y una biblioteca de doscientos no es el número: es que el primero cabe en la cabeza del modelo y el segundo compite consigo mismo.

Un subagente bien puesto es una poda: quita del hilo principal lo que no necesita recordar. Un subagente mal puesto es una mudanza: paga por llevar el contexto a otra casa y por traerlo de vuelta resumido.

---

**Siguiente:** [Codex](04-codex.md) — el mismo problema con TOML, créditos y sandbox heredado.
