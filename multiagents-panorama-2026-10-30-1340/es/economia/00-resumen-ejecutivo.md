---
title: "Resumen ejecutivo — ¿ahorran dinero los agentes y subagentes?"
description: "La respuesta corta, la aritmética que la sostiene y las siete decisiones que se siguen de ella."
date: "2026-10-30"
tags: ["tecnología", "agentes", "economía", "tokens"]
---

# Resumen ejecutivo — ¿ahorran dinero los agentes y subagentes?

> *Delegar no es repartir el trabajo: es decidir qué contexto merece sobrevivir. El resto es contabilidad.*

## 1. La respuesta corta

**Sí, pueden ahorrar; no, no ahorran por el hecho de ser agentes.**

Un agente o un subagente ahorra cuando se cumple al menos una de estas tres condiciones:

1. **Aísla contexto desechable** que, si se quedara en el hilo principal, se cobraría en cada turno posterior y empujaría la sesión hacia la compactación.
2. **Enruta el trabajo a un modelo más barato** que es suficientemente bueno para esa subtarea.
3. **Reduce el número de intentos fallidos** porque la tarea se ejecuta con un contexto limpio, permisos más estrechos o una verificación independiente.

No ahorra cuando la tarea es pequeña, secuencial, depende del contexto acumulado o exige que dos agentes tomen decisiones acopladas. En esos casos el subagente paga un contexto de arranque, ejecuta su propio bucle y devuelve un resumen: tres costes nuevos para ahorrar un contexto que, además, estaba cacheado.

La frase que resume el dossier:

$$
\text{ahorro} = \underbrace{\text{contexto que sale del hilo principal} \times \text{turnos restantes} \times \text{tarifa de caché}}_{\text{lo que se evita}}
\;-\;
\underbrace{(\text{arranque} + \text{trabajo} + \text{resumen} + \text{coordinación})}_{\text{lo que se paga}}
$$

Con caché caliente, la tarifa de lectura es del **5–10%** del precio de entrada. Ese es el detalle que desmonta la intuición: sacar 100.000 tokens del hilo principal no ahorra 100.000 tokens por turno; ahorra **5.000–10.000 tokens equivalentes** por turno. La delegación sólo paga si quedan muchos turnos, si el trabajo es muy voluminoso o si el subagente usa un modelo más barato.

## 2. Los tres mercados del coste

La pregunta «¿reduce el coste por tarea?» tiene tres respuestas distintas según la moneda:

| Moneda | Qué se paga | Cómo se mide | Quién lo nota |
|---|---|---|---|
| **Dólares de API** | Tokens de entrada, caché y salida | Factura del proveedor | Empresa con API key o cloud |
| **Cuota de suscripción** | Ventana de uso (5 h, semanal, mensual) | Barras de `/usage`, `/status`, dashboard | Claude Pro/Max; ChatGPT Plus/Pro; OpenCode Go |
| **Tiempo humano** | Minutos del desarrollador, revisión, retrabajo | Reloj y coste laboral | Cualquier equipo |

Un subagente puede ser **más caro en dólares** y **más barato en tiempo humano**. También puede ser **más barato en dólares** y **más caro en cuota**, porque cinco subagentes en paralelo consumen la misma ventana cinco veces más rápido. Y puede ser **más barato en cuota** si el trabajo se mueve a un modelo incluido en el plan o a un modelo local.

El dato que pone escala: Claude Code cuesta, en despliegues empresariales, **~$13 por desarrollador y día activo**, **$150–250 por mes**, y el 90% de los usuarios queda por debajo de **$30 por día activo** ([Anthropic, costes de Claude Code](https://code.claude.com/docs/en/costs)). Comparado con el coste de una hora de ingeniería, el token es barato; lo caro es equivocarse, revisar y repetir. Por eso el coste por tarea aceptada —no el coste por millón de tokens— es la métrica correcta.

## 3. Qué hacen las tres herramientas

| | **Claude Code** | **Codex** | **OpenCode** |
|---|---|---|---|
| **Subagentes** | Sí; `.claude/agents/*.md` | Sí; TOML en `~/.codex/agents/` o `.codex/agents/` | Sí; Markdown en `~/.config/opencode/agents/` o `.opencode/agents/` |
| **Delegación automática** | Por `description` | Cuando se pide o lo pide `AGENTS.md`/skill | Por `description`, o `@mención` |
| **Anidamiento** | Sí, 3 capas por defecto; configurable | Sí; el orquestador puede lanzar nuevos subagentes | v2: controlado por permisos; el `general` integrado no anida |
| **Modelo por agente** | Sí (`haiku`, `sonnet`, `opus`, `fable`, ID, `inherit`) | Sí (`model`, `model_reasoning_effort`) | Sí (`provider/model#variant`) |
| **Contexto aislado** | Sí; fork comparte caché, el resto no | Sí; hilo propio | Sí; sesión hija |
| **En segundo plano** | Sí | Sí | Sí (v2) |
| **Coste por agente** | Visible en `/usage` y transcripciones | `/status`, dashboard, logs | Coste de sesión; no siempre por subagente |
| **Techo práctico** | 20 subagentes concurrentes; 15k tokens de descripciones | `max_concurrent_threads_per_session` | Permisos y `steps` |

Las tres responden que **sí** a la pregunta «¿puede un agente delegar en un subagente?». La diferencia está en la semántica: Claude Code y Codex pueden anidar; OpenCode v2 permite configurar qué subagentes puede lanzar cada agente, pero su agente general integrado está explícitamente limitado. En las tres, el coste de la delegación es real: cada subagente es un bucle de modelo con su propio contexto.

## 4. La aritmética del ahorro

### 4.1. El mismo trabajo, cuatro modelos

100.000 tokens de entrada y 10.000 de salida, sin caché, a precios de lista del 30 de octubre de 2026:

| Modelo | Entrada | Salida | Total |
|---|---:|---:|---:|
| Claude Haiku 5.5 (≤100k) | $0,010 | $0,005 | **$0,015** |
| GPT-6 Luna | $0,010 | $0,005 | **$0,015** |
| Claude Sonnet 5.5 | $0,200 | $0,100 | **$0,30** |
| GPT-6.1 Sol | $0,200 | $0,100 | **$0,30** |
| Claude Opus 5.5 | $0,400 | $0,200 | **$0,60** |
| Claude Fable 5.1 | $1,000 | $0,500 | **$1,50** |

La diferencia entre el modelo barato y el caro es de **40×** para el mismo número de tokens. La diferencia entre un subagente y el hilo principal, para el mismo modelo, es de un **5–10%** de la entrada cacheada. Enrutar bien importa mucho más que delegar.

### 4.2. El arranque de un subagente

Un subagente fresco de Claude Code no empieza vacío: carga su prompt de sistema, la jerarquía de `CLAUDE.md`, el estado de git, las definiciones de herramientas y las skills precargadas. En una sesión medida por un tercero, **19 subagentes** escribieron entre **12.427 y 40.763 tokens** en su primera petición, con **cero lecturas de caché** ([Tokenade, 25 oct 2026](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0)). Ese arranque se paga a tarifa de escritura de caché: **1,25×** la entrada en el TTL de 5 minutos, **2×** en el de 1 hora.

A precios de Sonnet 5.5, 30.000 tokens de arranque cuestan **$0,075**; a precios de Opus 5.5, **$0,15**; a precios de Haiku 5.5 (≤100k), **$0,004**. Diecinueve subagentes son, sólo en arranque, **$1,43–$2,85** con Sonnet/Opus. No es una fortuna, pero es dinero que se paga antes de que el subagente lea un archivo.

### 4.3. El punto de equilibrio

Modelo: el hilo principal deja de cargar \(D=100.000\) tokens, al subagente le cuesta \(S=30.000\) de arranque y \(W=100.000\) de trabajo, devuelve \(R=2.000\) de resumen y produce \(O=10.000\) de salida. La caché principal está caliente y se lee al 5% de la entrada.

| Modelo principal → subagente | 10 turnos restantes | 25 turnos | 50 turnos | 100 turnos |
|---|---:|---:|---:|---:|
| Opus 5.5 → Opus 5.5 | −$0,56 | −$0,26 | +$0,24 | +$1,24 |
| Opus 5.5 → Sonnet 5.5 | −$0,18 | **+$0,12** | +$0,62 | +$1,62 |
| Opus 5.5 → Haiku 5.5 (>100k) | **+$0,10** | +$0,40 | +$0,90 | +$1,90 |
| Sonnet 5.5 → Sonnet 5.5 | −$0,28 | −$0,13 | +$0,12 | +$0,62 |
| Sonnet 5.5 → Haiku 5.5 (>100k) | +$0,002 | +$0,15 | +$0,40 | +$0,90 |

Lectura: **con el mismo modelo, delegar 100k tokens sólo empieza a pagar alrededor de 50 turnos restantes**. Con un modelo principal caro y un subagente barato, paga casi de inmediato. La tabla no incluye el valor de la calidad ni el coste de la compactación evitada; si el subagente evita un solo retrabajo, la balanza se mueve.

### 4.4. El multiplicador del fan-out

Si lanzas \(N\) subagentes en paralelo, con un trabajo total \(W\) repartido entre ellos y un arranque \(S\) por agente, el coste de tokens se multiplica aproximadamente por:

$$
M_{\text{fan-out}} \approx 1 + \frac{N \cdot S}{W}
$$

| N | S = 15k, W = 100k | S = 30k, W = 100k | S = 30k, W = 500k |
|---:|---:|---:|---:|
| 2 | 1,30× | 1,60× | 1,12× |
| 5 | 1,75× | 2,50× | 1,30× |
| 10 | 2,50× | 4,00× | 1,60× |

La tabla usa \(W\) como trabajo total; con \(N=5\) y \(W=100k\), cada agente hace 20k de trabajo y paga 30k de arranque. El paralelismo compra **tiempo de pared**, no tokens. Cinco subagentes que terminan en un quinto del tiempo pueden costar 2,5× los tokens. Si la cuota es el recurso escaso, el paralelismo es un lujo; si el reloj es el recurso escaso, es una compra racional.

## 5. La conversión agnóstica

**Se puede hacer, pero hay que elegir qué significa «funcionar».**

- **Capa común**: `AGENTS.md` para instrucciones (Claude Code lo lee si no hay `CLAUDE.md`; Codex y OpenCode lo usan de forma nativa), `SKILL.md` para skills (estándar Agent Skills) y MCP para herramientas.
- **Capa no común**: el archivo de agente. Claude Code: Markdown + YAML (`description`, `tools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `omitClaudeMd`, `isolation`). Codex: TOML (`name`, `description`, `developer_instructions`, `model`, `model_reasoning_effort`). OpenCode: Markdown + YAML (`mode`, `model`, `permissions` en v2; `permission` y `temperature` en v1).
- **Capa imposible de copiar**: la semántica de permisos, el enrutamiento por `description`, el modelo exacto, la caché y los hooks. Un conversor puede emitir un archivo válido y perder behavior.

Niveles de paridad que propone este dossier:

| Nivel | Qué garantiza | Cómo se comprueba |
|---|---|---|
| **L0** | Inventario | Parsear y listar agentes |
| **L1** | Archivo válido | La herramienta lo carga |
| **L2** | Capacidades | Herramientas, permisos, modelo y skills mapeados |
| **L3** | Comportamiento | Misma decisión de delegación en un set de tareas |
| **L4** | Economía | Mismo coste por tarea aceptada dentro de una tolerancia |

La mayoría de conversores llegan a **L1–L2**. `rulesync` ya genera subagentes para Claude Code, Codex y OpenCode; Codex tiene `/import` desde Claude Code; el conversor de xscriptor (`agents/claude/tools/xscriptor-convert.py`) es un caso real **unidireccional** OpenCode→Claude. Ninguno garantiza L3–L4 sin una evaluación.

## 6. La paradoja de xscriptor

El repo `agents` contiene **205 agentes** (181 especializados + 24 senior) y **~17.768 caracteres de descripciones** (unos **4.400 tokens**). Instalar los 205 en Claude Code, Codex u OpenCode tiene dos efectos:

1. **Coste fijo**: las descripciones se cargan en el contexto principal para que el modelo sepa a quién delegar. 4.400 tokens por petición es poco en dólares (unas centésimas por sesión), pero ocupa ventana y compite por atención.
2. **Coste de enrutamiento**: 205 opciones hacen que la elección sea menos fiable. Anthropic lo dice explícitamente: «flooding Claude with options makes automatic delegation less reliable; most teams settle on a handful of well-scoped agents» ([How and when to use subagents](https://claude.com/blog/subagents-in-claude-code)). La biblioteca es un catálogo excelente; el roster por defecto debería ser otro.

La recomendación práctica: **catálogo completo, instalación por capas**. Un núcleo de 8–12 agentes por proyecto, packs de dominio bajo demanda y el resto como skills o definiciones disponibles pero no cargadas. La diferencia entre un catálogo y un roster es la misma que entre una biblioteca y una mochila.

## 7. Siete decisiones

1. **Mide por tarea aceptada, no por token.** \(C_{\text{aceptada}} = C_{\text{ejecución}} / P_{\text{éxito}}\). Un agente de $1,50 con 90% de éxito ($1,67) es más barato que uno de $0,80 con 60% ($1,33)… sólo si el segundo no genera retrabajo humano. Añade el coste de revisión.
2. **Enruta antes de delegar.** Si la tarea puede hacerla Haiku/Luna, el ahorro es de 20–40×; el aislamiento de contexto ahorra un 5–10%. Empieza por el modelo.
3. **Delega sólo trabajo desechable y voluminoso.** Búsquedas amplias, triaje de logs, tests, revisión independiente, exploración de decenas de archivos. No delegues ediciones pequeñas, cadenas secuenciales ni ficheros que ya están abiertos.
4. **Cuenta el arranque.** Si el subagente va a hacer menos de ~30k tokens de trabajo, el arranque se come el ahorro. Usa fork cuando necesites el mismo contexto; usa el hilo principal cuando el contexto sea la tarea.
5. **Paraleliza por tiempo, no por dinero.** El fan-out multiplica tokens. Úsalo cuando el reloj valga más que la cuota, y limita la concurrencia.
6. **Convierte con manifiesto, no con fe.** Un conversor debe emitir qué mapeó, qué perdió y qué requiere verificación. Sin L3/L4, «funciona» significa «carga».
7. **Poda el roster.** Cada agente que no se usa es una descripción que se paga y una opción que distrae. 205 agentes no son 205 capacidades; son 205 decisiones.

## 8. Conclusión

La economía de los agentes no es la economía del token. Es la economía de la **atención**: qué contexto se conserva, qué se descarta y qué se paga por volver a leerlo. Un subagente es una forma de olvidar a propósito. Bien usado, olvidar es barato; mal usado, es pagar dos veces por el mismo contexto y llamarlo arquitectura.

La pregunta correcta no es *«¿cuántos agentes puedo lanzar?»* sino *«¿qué parte de este trabajo no necesita existir en el hilo principal?»*. Cuando la respuesta es clara, el subagente ahorra. Cuando no lo es, el subagente sólo añade una capa de indirección entre el problema y la factura.

---

**Siguiente:** [Contabilidad de tokens](01-contabilidad-de-tokens.md) — las fórmulas, los precios y los ejemplos que sostienen estas cifras.
