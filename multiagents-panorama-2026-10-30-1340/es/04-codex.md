---
title: "Codex — subagentes con TOML, créditos y sandbox heredado"
description: "Cómo funciona la delegación en Codex, qué se puede configurar por agente y qué parte del coste queda fuera del archivo."
date: "2026-10-30"
tags: ["tecnología", "codex", "agentes", "tokens"]
---

# Codex — subagentes con TOML, créditos y sandbox heredado

> *Codex delega como un capataz: el trabajador hereda el permiso, no lo negocia. Esa herencia abarata la configuración y encarece los errores.*

## 1. Mapa de la herramienta

Codex tiene subagentes nativos en CLI, IDE y app. La documentación oficial los presenta como **habilitados por defecto** en las versiones actuales; la actividad se ve como hilos de agente y el hilo principal recibe sólo el resumen. La versión GA se sitúa en **marzo de 2026 (v0.115.0)** según documentación secundaria; la fuente primaria no fija la fecha, así que tómala como referencia, no como cita contractual.

Tres conceptos:

| Término | Qué es |
|---|---|
| **Subagent workflow** | Varios agentes en paralelo cuyos resultados se consolidan |
| **Subagent / agent thread** | Un agente delegado con su propio hilo y contexto |
| **Custom agent** | Un archivo TOML que define rol, modelo, esfuerzo e instrucciones |

Codex trae tres roles integrados:

| Agente | Perfil | Uso típico |
|---|---|---|
| `default` | Propósito general | Fallback |
| `worker` | Implementación y arreglos | Cambios de código |
| `explorer` | Exploración de sólo lectura | Mapear el codebase |

Un agente custom con el mismo `name` que un integrado lo reemplaza.

## 2. El archivo de agente

Un archivo TOML por agente:

- `~/.codex/agents/*.toml` — personal.
- `.codex/agents/*.toml` — del proyecto, cargado en proyectos de confianza.

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

### 2.1. Campos

| Campo | Obligatorio | Qué hace |
|---|---|---|
| `name` | Sí | Identificador que usa Codex para lanzarlo |
| `description` | Sí | Guía para decidir cuándo usarlo |
| `developer_instructions` | Sí | Prompt de sistema del agente |
| `model` | No | Modelo; si falta, hereda el del padre |
| `model_reasoning_effort` | No | `low`, `medium`, `high`, `xhigh`, `max`, `ultra` según modelo |
| `model_reasoning_summary` | No | Resumen del razonamiento |
| `model_verbosity` | No | Verbosidad de la salida |
| `features.shell_tool` | No | Desactiva la shell |
| `features.apps`, `plugins`, `memories`, `request_permissions_tool` | No | Desactiva capacidades |
| `skills.config` | No | Habilita/deshabilita skills concretas |
| `skills.bundled.enabled` | No | Desactiva skills integradas |
| `skills.include_instructions` | No | Omite el bloque automático de instrucciones de skills |

Lo que **no** se puede poner en el archivo: `sandbox_mode`, `approval_policy`, `model_provider`, `mcp_servers`. Esas settings se heredan del padre. Un agente custom puede pedir «no edites» en sus instrucciones, pero **no crea un sandbox de sólo lectura**. Para eso, el sandbox se configura en la sesión padre o se usa el rol integrado `explorer`.

### 2.2. Settings globales

En `config.toml`:

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 4
default_subagent_model = "gpt-6-luna"
default_subagent_reasoning_effort = "high"
interrupt_message = true
```

| Campo | Efecto |
|---|---|
| `enabled` | Activa/desactiva las herramientas multiagente |
| `max_concurrent_threads_per_session` | Tope de hilos concurrentes (sin contar el principal) |
| `default_subagent_model` | Modelo por defecto de los subagentes |
| `default_subagent_reasoning_effort` | Esfuerzo por defecto |
| `interrupt_message` | Registra un mensaje visible cuando se interrumpe un agente |

La resolución es: valor explícito del spawn → default de `[agents]` → valor del padre. Un archivo custom que fija `model` conserva el esfuerzo resuelto previamente; si el modelo no soporta ese esfuerzo, hay que fijar `model_reasoning_effort` también.

## 3. Cómo se invoca

Codex **no delega proactivamente** en clientes locales. Lo hace cuando:

1. Se lo pides explícitamente: «spawn two agents», «delegate this in parallel», «use one agent per review lens».
2. Un `AGENTS.md` o una skill lo piden.
3. En ChatGPT Work con Ultra, puede delegar por iniciativa propia (no es el caso de la CLI).

El prompt de delegación debe decir **cómo dividir**, **si esperar** y **qué devolver**. Ejemplo:

```text
Review this branch with parallel subagents. Spawn one subagent for security risks,
one for test gaps, and one for maintainability. Wait for all three, then summarize
by category with file references.
```

Los subagentes heredan el sandbox, el approval mode, el proveedor y los MCP del padre. En CLI, las aprobaciones de hilos inactivos aparecen en una capa aparte; en ejecución no interactiva, una acción que requiere aprobación falla y el error vuelve al padre. En la app, los hilos se pueden inspeccionar, parar o cerrar.

El orquestador puede lanzar nuevos subagentes; no es una capacidad que el archivo custom controle. La profundidad y la concurrencia las decide el runtime.

## 4. El coste

### 4.1. Créditos, no dólares

Con suscripción, Codex factura en **créditos**; con API key, en dólares. Las dos facturas no son convertibles. Para GPT-6.1 Sol:

| Categoría | Créditos / MTok | USD / MTok (API) |
|---|---:|---:|
| Entrada | 50 | $2,00 |
| Entrada cacheada | 2,5 | $0,10 |
| Salida | 250 | $10,00 |
| Escritura de caché | **no se cobra** | $2,50 (30m) / $5,00 (12h) |

Para GPT-6 Luna: 2,5 / 0,25 / 12,5 créditos por MTok (API $0,10 / $0,01 / $0,50). Para GPT-6.1 Terra: 25 / 1,25 / 125 (API $1,00 / $0,05 / $5,00).

Un trabajo de 100k de entrada + 10k de salida cuesta, en créditos:

| Modelo | Entrada | Salida | Total | vs Sol |
|---|---:|---:|---:|---:|
| GPT-6.1 Sol | 5,0 | 2,5 | **7,5** | 1× |
| GPT-6.1 Terra | 2,5 | 1,25 | **3,75** | 2× más barato |
| GPT-6 Luna | 0,25 | 0,125 | **0,375** | 20× más barato |

La página de precios dice que una tarea típica de GPT-5.6 Sol puede usar **2–15 créditos**. Es una unidad más intuitiva que el millón de tokens, pero no responde a «¿cuánto cuesta mi tarea?» sin medir.

### 4.2. Planes y ventanas

| Plan | Precio | Qué incluye |
|---|---|---|
| Free | $0 | GPT-6 Luna, tareas rápidas |
| Go | $8/mes | Codex ligero |
| Plus | $20/mes | GPT-6.1 Sol y Luna; uso semanal |
| Pro | desde $100/mes | $100/$200/$500; sin límite de 5 horas |
| Business / Enterprise | según contrato | Créditos, flexible pricing |
| API key | pago por token | Sin funciones cloud |

Plus y Standard Business estiman, por ventana de 5 horas: **15–160 mensajes locales** con GPT-6.1 Sol y **350–3.000** con GPT-6 Luna. Son estimaciones, no límites fijos. Los subagentes consumen la misma cuota; el fan-out la gasta más rápido.

### 4.3. Modos rápidos

| Modo | Uso incluido | Créditos comprados |
|---|---:|---:|
| Standard | 1× | 1× |
| Fast | 2,5× | 2× |
| Ultrafast (Astra/Sol) | 8× | 6× |

Fast y Ultrafast compran latencia con cuota. Para un subagente de exploración, Standard suele ser suficiente; para una revisión con reloj, Fast puede tener sentido.

### 4.4. Dónde se ve

- `/status` en la CLI: modelo activo y uso restante.
- Dashboard de uso: `chatgpt.com/codex/settings/usage`.
- La app muestra la actividad de subagentes y el resumen que devuelven.
- En API, la respuesta y los logs de la organización.

No hay una línea de «coste por subagente» equiparable a la de Claude Code; el análisis se hace por hilo y por modelo.

## 5. Recetas

### 5.1. Explorador barato

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

Por qué ahorra: Luna es 20× más barato que Sol por token; sin shell, el agente no puede divagar ejecutando comandos; la salida es acotada. Límite: no es un sandbox real; el padre debe correr con el sandbox adecuado.

### 5.2. Revisor

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

Por qué ahorra: el modelo caro se usa sólo para el juicio; el contexto de implementación se queda en el padre.

### 5.3. Worker de implementación

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

Por qué ahorra: Terra es 2× más barato que Sol; el alcance acotado evita exploraciones largas.

### 5.4. Defaults del proyecto

```toml
[agents]
max_concurrent_threads_per_session = 4
default_subagent_model = "gpt-6-luna"
default_subagent_reasoning_effort = "high"
```

Un default barato evita que cada spawn herede Sol por inercia. Los agentes que necesiten Sol lo declaran.

## 6. El playbook de ahorro

1. **Luna para explorar, Terra para implementar, Sol para revisar.** Es la asignación más rentable con los precios actuales.
2. **Limita la concurrencia.** `max_concurrent_threads_per_session` es un presupuesto, no un adorno. Cuatro hilos suelen bastar.
3. **Sé explícito con la división.** Un prompt vago produce duplicación; la duplicación es el mayor multiplicador de coste oculto.
4. **Evita el fan-out en escritura.** Dos agentes editando el mismo archivo crean conflictos y retrabajo; usa paralelismo para lectura, tests y revisión.
5. **Aprovecha la herencia.** El sandbox, el approval mode y los MCP se configuran una vez en el padre; no intentes replicarlos en cada TOML.
6. **Usa AGENTS.md anidado.** Codex lee el `AGENTS.md` más cercano; un monorepo con instrucciones por paquete evita cargar el mundo entero.
7. **Mide en créditos y en tarea aceptada.** Un subagente de 0,375 créditos que falla es más caro que uno de 7,5 que acierta.
8. **No confundas Fast con barato.** Fast multiplica la cuota; úsalo cuando el reloj valga más.

## 7. Trampas

- **Creer que el TOML aísla permisos.** No lo hace: hereda sandbox, approval, proveedor y MCP. El archivo puede desactivar features, no crear un sandbox.
- **Pedir «sólo lectura» en las instrucciones.** Es una intención, no un control. Usa el rol `explorer` o el sandbox del padre.
- **Asumir que Codex delega solo.** En local no lo hace proactivamente; hay que pedirlo o ponerlo en `AGENTS.md`/skill.
- **Ignorar el coste de la concurrencia.** Cada hilo es un bucle; cuatro hilos no cuestan lo mismo que uno.
- **Usar el precio de API para estimar el plan.** La propia documentación lo desaconseja.
- **Olvidar el contexto largo.** En API, >272k duplica la entrada y multiplica la salida; en suscripción, GPT-6.1 Sol no tiene recargo de contexto largo, pero el uso sigue contando.
- **Confundir créditos con calidad.** Luna es barato y capaz para explorar; no es Sol para decidir.

## 8. Relevancia para xscriptor

El conversor de xscriptor no emite TOML, así que Codex es hoy el destino con más trabajo manual. Si se añade un adaptador, tres decisiones importan:

1. **Mapear niveles, no IDs**: `fast` → `gpt-6-luna`, `balanced` → `gpt-6.1-terra`, `frontier` → `gpt-6.1-sol`. Así el catálogo sobrevive a los cambios de modelo.
2. **No prometer sandbox por agente**: Codex lo hereda. Un agente de auditoría debe usar el rol `explorer` o el sandbox del padre, no un TOML que diga «no edites».
3. **Limitar la concurrencia por proyecto**: 205 agentes no implican 205 hilos; `max_concurrent_threads_per_session` es el freno económico.

## 9. Cierre

Codex es la herramienta más opaca en dinero y la más clara en herencia: el subagente no negocia permisos, los recibe. Esa simplicidad es una virtud de configuración y un riesgo de seguridad. El ahorro real viene de la asignación de modelos —Luna, Terra, Sol— y de la disciplina de concurrencia, no de un campo mágico en el TOML.

Un subagente de Codex es un hilo con un contrato corto. Cuanto más corto el contrato, más barato el hilo; cuanto más vago, más se parece a contratar un segundo equipo para repetir la reunión.

---

**Siguiente:** [OpenCode](05-opencode.md) — la versión abierta: dos formatos, permisos y sesiones hijas.
