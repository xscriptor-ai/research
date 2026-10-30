---
title: "Contabilidad de tokens — la fórmula que nadie enseña"
description: "Entrada, caché, salida, TTL y modelo: cómo se convierte una sesión de agente en una factura, con ejemplos numéricos."
date: "2026-10-30"
tags: ["tecnología", "tokens", "economía", "caché"]
---

# Contabilidad de tokens — la fórmula que nadie enseña

> *Un token no cuesta lo mismo la primera vez que se lee, la segunda que se recuerda y la tercera que se reescribe. La factura es la historia de esas tres lecturas.*

## 1. Qué se factura

Los proveedores no facturan «una conversación». Facturan cuatro categorías de token, cada una con su precio:

| Categoría | Qué es | Cuándo se paga |
|---|---|---|
| **Entrada no cacheada** | Tokens nuevos que el modelo procesa | En cada petición, por la parte que no está en caché |
| **Escritura de caché** | Tokens que se almacenan para reutilizar | La primera vez, con recargo (1,25× o 2×) |
| **Lectura de caché** | Tokens recuperados de una caché válida | En peticiones posteriores, con descuento (0,05×–0,1×) |
| **Salida** | Tokens que genera el modelo, incluido el razonamiento | Siempre, al precio de salida |

La fórmula general, para una petición:

$$
C = p_{\text{in}} \cdot U + p_{\text{cw}} \cdot W + p_{\text{cr}} \cdot R + p_{\text{out}} \cdot O
$$

donde \(U\) es la entrada no cacheada, \(W\) la escritura de caché, \(R\) la lectura de caché y \(O\) la salida. Para una sesión, se suma sobre todas las peticiones:

$$
C_{\text{sesión}} = \sum_{t=1}^{T} \left( p_{\text{in}} U_t + p_{\text{cw}} W_t + p_{\text{cr}} R_t + p_{\text{out}} O_t \right)
$$

Esta suma es la que convierte una decisión de arquitectura en dinero. Un subagente añade peticiones; un cambio de modelo cambia los cuatro precios; una caché mal alineada cambia \(W\) y \(R\).

## 2. Precios de referencia (30 de octubre de 2026)

### 2.1. Anthropic

Precios en USD por millón de tokens (MTok). La caché de 5 minutos se escribe a 1,25× la entrada; la de 1 hora, a 2×.

| Modelo | Entrada | Salida | Lectura de caché | Escritura 5m | Escritura 1h |
|---|---:|---:|---:|---:|---:|
| Claude Haiku 5.5 (≤100k) | $0,10 | $0,50 | $0,01 | $0,125 | $0,20 |
| Claude Haiku 5.5 (>100k) | $0,50 | $2,50 | $0,05 | $0,625 | $1,00 |
| Claude Sonnet 5.5 | $2,00 | $10,00 | $0,10 | $2,50 | $4,00 |
| Claude Opus 5.5 | $4,00 | $20,00 | $0,20 | $5,00 | $8,00 |
| Claude Fable 5.1 | $10,00 | $50,00 | $0,25 | $12,50 | $20,00 |

Notas que cambian la comparación:

- **Haiku 5.5 tiene dos tarifas**: si el prompt supera los 100.000 tokens, la entrada se multiplica por 5 y la salida por 5. Un subagente de 130k tokens no paga tarifa barata.
- **La lectura de caché no es 0,1× en todos los modelos**: en Opus 5.5 y Sonnet 5.5 es **0,05×**; en Fable 5.1 y Mythos 5.1, **0,025×**. La tabla ya lo incorpora.
- **Claude 4.7 y posteriores usan un tokenizador nuevo** que produce ~30% más tokens para el mismo texto. Comparar precios por token entre generaciones sin corregir esto exagera el ahorro de los modelos nuevos.
- **Batch API**: 50% de descuento en entrada y salida, pero no es interactivo; no sustituye a un subagente en una sesión.
- **Fast mode**: duplica el precio de Opus 5.5 ($8/$40) y de Opus 5/4.8 ($10/$50); se suma a la caché y a la residencia de datos.
- **Residencia de datos** (`inference_geo: "us"`): 1,1× en todas las categorías para Claude 4.6+.

Fuente: [Anthropic pricing](https://platform.claude.com/docs/en/about-claude/pricing), leída el 30/10/2026.

### 2.2. OpenAI

Precios en USD por MTok, contexto corto (≤272k):

| Modelo | Entrada | Salida | Entrada cacheada | Escritura 30m | Escritura 12h |
|---|---:|---:|---:|---:|---:|
| GPT-6.1 Sol | $2,00 | $10,00 | $0,10 | $2,50 | $5,00 |
| GPT-6.1 Terra | $1,00 | $5,00 | $0,05 | $1,25 | $2,50 |
| GPT-6 Luna | $0,10 | $0,50 | $0,01 | $0,125 | $0,25 |
| GPT-6 Astra | $10,00 | $50,00 | $1,00 | $12,50 | $25,00 |

Notas:

- **Contexto largo** (>272k): GPT-6.1 Sol pasa a $4,00 de entrada y $15,00 de salida; la entrada cacheada a $0,20.
- **Codex con suscripción** no factura en dólares por token: factura en **créditos**. Para GPT-6.1 Sol, 50 créditos por MTok de entrada, 2,5 por MTok cacheado y 250 por MTok de salida. **No hay cargo separado por escritura de caché** en la facturación de Codex.
- **Los precios de API no sirven para estimar tareas incluidas en el plan.** La propia página de pricing de Codex lo dice: «API token prices are separate from subscription usage; don't use them to estimate included tasks».
- **Fast** multiplica por 2,5 el uso incluido y por 2 los créditos comprados; **Ultrafast** por 8 y 6, respectivamente.

Fuente: [OpenAI API pricing](https://platform.openai.com/docs/pricing) y [Codex pricing](https://learn.chatgpt.com/docs/pricing), leídas el 30/10/2026.

### 2.3. OpenCode

OpenCode no es un proveedor de modelos: es un cliente y, opcionalmente, un gateway (Console/Zen). El coste depende del proveedor que elijas. OpenCode Console publica una tabla por modelo; algunos ejemplos del 30/10/2026:

| Modelo | Entrada | Salida | Lectura caché |
|---|---:|---:|---:|
| Claude Opus 5.5 | $4,00 | $20,00 | $0,20 |
| Claude Sonnet 5.5 | $2,00 | $10,00 | $0,10 |
| Claude Haiku 5.5 (≤100k) | $0,10 | $0,50 | $0,01 |
| GPT-6.1 Sol (≤272k) | $2,00 | $10,00 | $0,10 |
| DeepSeek V4.1 Flash | $0,30 | $1,20 | $0,006 |
| GLM 5.3 Flash | $0,15 | $0,50 | $0,03 |

El detalle de DeepSeek V4.1 Flash es instructivo: su lectura de caché es **$0,006/M**, un 98% más barata que su entrada. Eso hace que un subagente de exploración con contexto repetido sea casi gratis en esa combinación. Pero el modelo, la latencia y la calidad no son los de Opus; el ahorro es real y la comparación no es equivalente.

Fuente: [OpenCode Console pricing](https://opencode.ai/docs/zen/), leída el 30/10/2026.

## 3. La caché: dónde se gana y dónde se pierde

### 3.1. Cómo funciona

La caché de prompt guarda un **prefijo** de la petición. Si la siguiente petición empieza exactamente igual, el proveedor lee ese prefijo de caché en lugar de reprocesarlo. El prefijo incluye, en orden:

1. El prompt de sistema y las definiciones de herramientas.
2. El contexto del proyecto (`CLAUDE.md`, `AGENTS.md`, reglas, skills cargadas).
3. La conversación: mensajes, resultados de herramientas, respuestas.

Un cambio en cualquier punto invalida todo lo que viene después. Por eso cambiar de modelo a mitad de sesión, conectar un MCP o editar el prompt de sistema puede costar una relectura completa.

### 3.2. Los multiplicadores que importan

| Operación | Anthropic | OpenAI | Efecto práctico |
|---|---|---|---|
| Lectura de caché | 0,05×–0,1× | 0,05× | El contexto repetido es casi gratis |
| Escritura 5m/30m | 1,25× | 1,25× | La primera vez cuesta un 25% más |
| Escritura 1h/12h | 2× | 2,5× | La caché larga es más cara; sólo paga si hay pausas |
| Cambio de modelo | Invalida | Invalida | Un cambio a mitad de tarea puede costar la sesión entera |
| Compactación | Reescribe la conversación | Equivalente | La siguiente petición no comparte prefijo |

La regla de oro: **la caché premia la estabilidad**. Un agente que cambia de modelo, de herramientas o de prompt a mitad de tarea paga por un contexto que ya había pagado.

### 3.3. El caso del subagente

Un subagente **no comparte la caché del padre** salvo que sea un *fork*. Su prompt de sistema, sus herramientas y su contexto son distintos, así que su primera petición escribe su propio prefijo y no lee el del padre. En Claude Code:

- Un subagente normal empieza con **cero lecturas de caché** y paga la escritura de su arranque.
- Un **fork** hereda el prompt, las herramientas y la conversación del padre; su primera petición reutiliza la caché del padre. Es más barato para tareas que necesitan el mismo contexto.
- Los subagentes tienen su propio TTL (5 minutos por defecto, incluso en suscripción); se puede subir con `subagentPromptCacheTtl` o `cacheTtl` en el frontmatter.

Fuente: [Claude Code prompt caching](https://code.claude.com/docs/en/prompt-caching), [subagents](https://code.claude.com/docs/en/sub-agents).

## 4. Ejemplos numéricos

### 4.1. El mismo trabajo en cuatro modelos

100k de entrada + 10k de salida, sin caché:

| Modelo | Entrada | Salida | Total | vs Haiku |
|---|---:|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0,010 | $0,005 | **$0,015** | 1× |
| GPT-6 Luna | $0,010 | $0,005 | **$0,015** | 1× |
| Sonnet 5.5 | $0,200 | $0,100 | **$0,30** | 20× |
| GPT-6.1 Sol | $0,200 | $0,100 | **$0,30** | 20× |
| Opus 5.5 | $0,400 | $0,200 | **$0,60** | 40× |
| Fable 5.1 | $1,000 | $0,500 | **$1,50** | 100× |

Si esos 100k fueran lectura de caché y sólo 10k fueran entrada nueva:

| Modelo | Entrada nueva | Caché | Salida | Total |
|---|---:|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0,001 | $0,001 | $0,005 | **$0,007** |
| Sonnet 5.5 | $0,020 | $0,010 | $0,100 | **$0,13** |
| Opus 5.5 | $0,040 | $0,020 | $0,200 | **$0,26** |
| GPT-6.1 Sol | $0,020 | $0,010 | $0,100 | **$0,13** |

La caché reduce el coste de entrada en un orden de magnitud. Eso explica por qué «tener el contexto en el hilo principal» no es tan caro como parece… y por qué sacarlo tampoco ahorra tanto como parece.

### 4.2. El arranque de un subagente

30.000 tokens de arranque, escritura de caché de 5 minutos:

| Modelo | Coste de arranque |
|---|---:|
| Haiku 5.5 (≤100k) | $0,00375 |
| Sonnet 5.5 | $0,075 |
| Opus 5.5 | $0,15 |
| Fable 5.1 | $0,375 |

Si el subagente trabaja 10.000 tokens y devuelve 1.000, el coste total (sin salida) es:

| Modelo | Arranque | Trabajo | Total |
|---|---:|---:|---:|
| Haiku 5.5 (≤100k) | $0,00375 | $0,001 | **$0,00475** |
| Sonnet 5.5 | $0,075 | $0,020 | **$0,095** |
| Opus 5.5 | $0,15 | $0,040 | **$0,19** |

Con Sonnet, un subagente que hace 10k de trabajo cuesta **$0,095** antes de la salida. Si el trabajo se pudiera hacer en el hilo principal por $0,02, el subagente es un lujo. Si el trabajo produce 100k de logs que se habrían quedado en el contexto, el cálculo cambia.

### 4.3. El punto de equilibrio, con números

Sea \(D=100.000\) tokens que se mantienen fuera del hilo principal, \(T\) turnos restantes, \(r=0,05\) la fracción de lectura de caché, \(p_{\text{in}}\) el precio de entrada del modelo principal. El ahorro es:

$$
\text{ahorro}(T) = D \cdot T \cdot r \cdot p_{\text{in}}
$$

El coste del subagente, con arranque \(S=30.000\), trabajo \(W=100.000\), salida \(O=10.000\) y resumen \(R=2.000\), es:

$$
\text{coste}_{\text{sub}} = (1{,}25S + W) \cdot p_{\text{in,sub}} + O \cdot p_{\text{out,sub}} + R \cdot p_{\text{in,main}}
$$

Con Sonnet 5.5 en ambos lados:

- Ahorro por turno: \(100.000 \times 0,05 \times \$2/M = \$0,01\).
- Coste del subagente: \((37.500+100.000)\times\$2/M + 10.000\times\$10/M + 2.000\times\$2/M = \$0,275 + \$0,10 + \$0,004 = \$0,379\).
- Punto de equilibrio: \(T = 0,379/0,01 \approx 38\) turnos.

Con Opus 5.5 en el principal y Sonnet 5.5 en el subagente:

- Ahorro por turno: \(100.000 \times 0,05 \times \$4/M = \$0,02\).
- Coste del subagente: **el mismo**, $0,379.
- Punto de equilibrio: \(T \approx 19\) turnos.

Con Opus 5.5 en ambos:

- Ahorro por turno: $0,02.
- Coste: \((37.500+100.000)\times\$4/M + 10.000\times\$20/M + 2.000\times\$4/M = \$0,55 + \$0,20 + \$0,008 = \$0,758\).
- Punto de equilibrio: \(T \approx 38\) turnos.

La lección: **el enrutamiento a un modelo más barato reduce el punto de equilibrio a la mitad; el aislamiento por sí solo no**. Y si el subagente es más caro que el principal, el punto de equilibrio se aleja.

## 5. Suscripción no es dólares

En Claude Code Pro/Max, Codex Plus/Pro y OpenCode Go, el usuario paga una cuota fija y consume una **ventana de uso**. No hay una conversión pública de tokens a dólares para esas ventanas. Lo que sí se sabe:

- **Claude Code**: todos los subagentes consumen la misma cuota que la conversación principal. `/usage` muestra la atribución por skills, subagentes, plugins y MCP; la ventana es de 5 horas y semanal. Cinco subagentes en paralelo pueden agotar una ventana mucho antes que una sesión secuencial.
- **Codex**: los subagentes consumen los mismos créditos del plan. `/status` muestra el modelo y el uso; el dashboard muestra los límites. La tabla de créditos permite comparar modelos, pero no convierte la cuota en dólares.
- **OpenCode Go**: los límites son por modelo, con un 20% del límite mensual por ventana de 5 horas, 50% semanal y 100% mensual. Un fan-out grande en un solo modelo puede bloquear ese modelo el resto del día.

La consecuencia práctica: **si pagas suscripción, el coste marginal de un token es cero hasta que no lo es**. El recurso escaso no es el dinero, es la ventana. Eso cambia la decisión: un subagente que ahorra 20 minutos puede ser racional aunque consuma 3× los tokens, si la ventana aguanta.

## 6. El coste por tarea aceptada

La métrica que importa no es \(C_{\text{sesión}}\), sino:

$$
C_{\text{aceptada}} = \frac{C_{\text{ejecución}} + C_{\text{retrabajo}} \cdot P_{\text{fallo}}}{P_{\text{éxito}}}
$$

Si el reintento cuesta lo mismo que la ejecución, \(C_{\text{aceptada}} = C_{\text{ejecución}} / P_{\text{éxito}}\). Ejemplos:

| Coste por ejecución | Tasa de éxito | Coste por tarea aceptada |
|---:|---:|---:|
| $0,50 | 70% | **$0,71** |
| $0,80 | 60% | **$1,33** |
| $1,50 | 90% | **$1,67** |
| $3,00 | 95% | **$3,16** |

Un agente más caro pero más fiable puede ser más barato por tarea aceptada. Y si el fallo implica 30 minutos de revisión humana, el cálculo se dispara: a $60/hora, 30 minutos son $30, veinte veces el coste de tokens del ejemplo.

## 7. Lo que la tabla de precios no dice

- **Calidad**: el modelo barato puede fallar más; el coste por tarea aceptada lo captura, el precio por token no.
- **Latencia**: un subagente en paralelo puede reducir el tiempo de pared sin reducir tokens; un modelo barato puede ser más lento.
- **Compactación**: un contexto grande obliga a resumir; la compactación es una petición grande y puede perder información.
- **Cuota**: la suscripción no se convierte en dólares; la ventana sí se agota.
- **Tokens de razonamiento**: se facturan como salida; un esfuerzo alto puede multiplicar la salida sin cambiar la entrada.
- **Tokenizador**: el mismo texto puede tener un 30% más de tokens en modelos nuevos.
- **Impuestos, descuentos y acuerdos**: los precios de lista no son la factura final.

## 8. Cierre

La contabilidad de tokens es incómoda porque obliga a distinguir tres cosas que la intuición mezcla: lo que cuesta **leer**, lo que cuesta **recordar** y lo que cuesta **volver a leer**. Un subagente es una apuesta sobre la tercera. A veces la gana; a veces sólo paga dos veces por el mismo contexto.

La fórmula no decide por ti. Pero evita que la decisión la tome una anécdota.

---

**Siguiente:** [Subagentes y contexto](02-subagentes-y-contexto.md) — por qué el aislamiento a veces ahorra y a veces sólo traslada el coste.
