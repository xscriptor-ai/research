---
title: "Multiagents Panorama — El precio y la velocidad de delegar"
description: "Dossier sobre la economía, la aceleración y el rendimiento de proyectos con agentes y subagentes en Claude Code, Codex y OpenCode."
date: "2026-10-30"
tags: ["tecnología", "agentes", "economía", "rendimiento", "investigación"]
---

# Multiagents Panorama — El precio y la velocidad de delegar

> *Ningún subagente es gratis: paga un contexto de arranque antes de leer la primera línea. La pregunta no es si delega, sino qué parte del trabajo merece existir dos veces — y qué parte merece la pena esperar.*

[English](../en/README.md) · **Español**

Dossier sobre **agentes y subagentes en Claude Code, OpenAI Codex y OpenCode**, organizado en dos ejes:

- **Economía** — tokens, caché, enrutamiento de modelos, punto de equilibrio y coste por tarea aceptada.
- **Rendimiento** — aceleración, latencia, coordinación, calidad y entrega del proyecto.

Los precios, los nombres de modelo y el comportamiento de producto se verificaron contra documentación primaria el **30 de octubre de 2026**. Los precios cambian; las fórmulas no.

## La respuesta corta

1. **Un subagente no es un cupón de descuento.** Es un mecanismo de aislamiento de contexto y de enrutamiento de modelos. Reduce el *coste por tarea aceptada* sólo cuando el trabajo aislado es lo bastante grande, desechable e independiente como para pagar su arranque, su propio bucle y el resumen que devuelve al padre.
2. **Los tokens totales normalmente suben.** Anthropic midió ~4× los tokens de un chat para un agente y ~15× para un sistema multiagente de investigación; los equipos de agentes de Claude Code, ~7× una sesión estándar. Un subagente fresco de Claude Code escribió **12.427–40.763 tokens** en su caché antes de hacer nada.
3. **La gran palanca financiera es el modelo, no la delegación.** El mismo trabajo de 100k de entrada y 10k de salida cuesta **$0,015 en Haiku 5.5**, **$0,30 en Sonnet 5.5**, **$0,60 en Opus 5.5** y **$1,50 en Fable 5.1**. Un subagente barato puede ahorrar 20–40× por token; el aislamiento solo ahorra la tarifa de caché, a menudo un 5–10% de eso.
4. **La aceleración es local.** El fan-out acelera sólo las ramas independientes. Si la integración es el 30% del trabajo, Amdahl limita el speedup a 3,3× aunque lances infinitos agentes; si es el 50%, a 2×. Más agentes reparten la espera; no la eliminan.
5. **La calidad no escala con el número de agentes.** El propio equipo de Applied AI de Anthropic encontró que un agente único con skills superaba a menudo en calidad y coste a la arquitectura de subagentes; los subagentes quedan para aislamiento, paralelismo y permisos.
6. **La conversión agnóstica es posible en la capa estructural.** `AGENTS.md`, `SKILL.md` y MCP son terreno común. Los archivos de agente no: Claude Code usa Markdown + YAML, Codex TOML, OpenCode Markdown + YAML con otro modelo de permisos. Un conversor alcanza **paridad de capacidades**; la **paridad de comportamiento y economía** exige IR, manifiesto y evaluación.

## Los números clave

| Afirmación | Número | Fuente |
|---|---|---|
| Agentes vs chat | ~4× tokens | Anthropic, sistema multiagente de investigación (2025) |
| Multiagente vs chat | ~15× tokens | Ibíd. |
| Equipos de agentes de Claude Code vs sesión estándar | ~7× tokens (modo plan) | Docs de costes de Claude Code (2026) |
| Arranque de un subagente fresco de Claude Code | 12.427–40.763 tokens de escritura de caché | Medición independiente de transcripciones (2026) |
| Enrutamiento de modelo, mismo trabajo | 40× entre Haiku 5.5 y Opus 5.5 | Precios de lista de Anthropic (2026) |
| Tiempo de investigación con agentes en paralelo | Hasta **90% menos** en consultas complejas | Anthropic, multi-agent research (2025) |
| Bucle único vs multiagente | **~2× más rápido** en un caso empresarial | Anthropic Applied AI (2026) |
| Biblioteca de agentes de xscriptor | 205 agentes, ~17.768 caracteres de descripciones | Repo local `agents` (2026) |
| Gasto medio de Claude Code | ~$13/desarrollador/día activo; $150–250/mes | Docs de costes de Anthropic (2026) |

## Secciones

| Sección | Español | English |
|---|---|---|
| **Economía** | [Índice](economia/README.md) | [Index](../en/economics/README.md) |
| **Rendimiento** | [Índice](rendimiento/README.md) | [Index](../en/performance/README.md) |
| **Herramientas** | [Claude Code](03-claude-code.md) · [Codex](04-codex.md) · [OpenCode](05-opencode.md) | [Claude Code](../en/03-claude-code.md) · [Codex](../en/04-codex.md) · [OpenCode](../en/05-opencode.md) |
| **Conversión** | [Conversión agnóstica](06-conversion-agnostica.md) | [Agnostic conversion](../en/06-agnostic-conversion.md) |
| **Playbook** | [Medición](07-medicion-y-playbook.md) | [Measurement](../en/07-measurement-and-playbook.md) |
| **Fuentes** | [Fuentes](08-fuentes.md) | [Sources](../en/08-sources.md) |

## Documentos

| # | Español | English |
|---|---|---|
| 00 | [Resumen ejecutivo](economia/00-resumen-ejecutivo.md) | [Executive summary](../en/economics/00-executive-summary.md) |
| 01 | [Contabilidad de tokens](economia/01-contabilidad-de-tokens.md) | [Token accounting](../en/economics/01-token-accounting.md) |
| 02 | [Subagentes y contexto](economia/02-subagentes-y-contexto.md) | [Subagents and context](../en/economics/02-subagents-and-context.md) |
| 03 | [Aceleración y latencia](rendimiento/00-aceleracion-y-latencia.md) | [Acceleration and latency](../en/performance/00-acceleration-and-latency.md) |
| 04 | [Calidad y rendimiento del proyecto](rendimiento/01-calidad-y-rendimiento-del-proyecto.md) | [Quality and project performance](../en/performance/01-quality-and-project-performance.md) |
| 05 | [Casos y variables](rendimiento/02-casos-y-comparativas.md) | [Cases and variables](../en/performance/02-cases-and-comparisons.md) |
| 06 | [Claude Code](03-claude-code.md) | [Claude Code](../en/03-claude-code.md) |
| 07 | [Codex](04-codex.md) | [Codex](../en/04-codex.md) |
| 08 | [OpenCode](05-opencode.md) | [OpenCode](../en/05-opencode.md) |
| 09 | [Conversión agnóstica](06-conversion-agnostica.md) | [Agnostic conversion](../en/06-agnostic-conversion.md) |
| 10 | [Medición y playbook](07-medicion-y-playbook.md) | [Measurement and playbook](../en/07-measurement-and-playbook.md) |
| 11 | [Fuentes](08-fuentes.md) | [Sources](../en/08-sources.md) |

**Datos:** [precios por modelo](../data/prices-2026-10-30.csv) · [escenarios de delegación](../data/delegation-scenarios.csv) · [notas](../data/README.md).

## Cómo leerlo

- Si sólo tienes diez minutos: **00** y la tabla de decisión de **02**.
- Si te importa el coste: **01** y **02**.
- Si te importa la velocidad o la calidad: **03**, **04** y **05**.
- Si vas a configurar una herramienta: **06**, **07** o **08**.
- Si vas a escribir el conversor: **09**.
- Si vas a medir o defender el cambio: **10**.
- Si quieres auditar una cifra: **11**.

## Método

- **Fuentes primarias primero**: documentación y páginas de precios de los proveedores, leídas el 30 de octubre de 2026.
- **Fuentes secundarias etiquetadas**: las mediciones independientes y la prensa se marcan como tales; ninguna anécdota se presenta como benchmark.
- **Fórmulas antes que folclore**: cada afirmación de coste o velocidad se deriva de categorías de token, la ley de Amdahl o tiempos medidos, con supuestos explícitos.
- **Reproducible**: el protocolo de medición y los niveles de paridad de la conversión están especificados para repetir el experimento en tu repositorio.

Esto es un dossier de investigación, no una recomendación de compra. La arquitectura correcta depende del valor de la tarea, del coste de una respuesta equivocada, de la longitud de la sesión y del camino crítico del proyecto: las variables que la factura de tokens no muestra.

## Licencia

MIT. El texto y las tablas pueden reutilizarse con atribución.
