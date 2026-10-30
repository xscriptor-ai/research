---
title: "Rendimiento — aceleración, latencia y calidad"
description: "Índice de la sección de rendimiento: qué acelera el fan-out, qué frena la integración y qué pasa con la calidad."
date: "2026-10-30"
tags: ["rendimiento", "latencia", "calidad", "agentes"]
---

# Rendimiento — aceleración, latencia y calidad

Esta sección responde cuánto acelera y qué calidad deja. La sección de [economía](../economia/README.md) responde cuánto cuesta.

| Documento | Contenido |
|---|---|
| [00 — Aceleración y latencia](00-aceleracion-y-latencia.md) | Amdahl, coordinación, impuesto de velocidad y viabilidad de 1→2→4 |
| [01 — Calidad y rendimiento del proyecto](01-calidad-y-rendimiento-del-proyecto.md) | Calidad, fiabilidad, entrega, mantenibilidad y webapp vs distro |
| [02 — Casos y variables](02-casos-y-comparativas.md) | Lenguajes, tipos de app, documentación, papers y matriz de decisión |

## La idea en una línea

El fan-out acelera sólo las ramas independientes. Si la integración es el 30% del trabajo, el máximo es 3,3× aunque lances infinitos agentes; si es el 50%, el máximo es 2×. Más agentes no rompen Amdahl: reparten la espera.

**Siguiente:** [Economía](../economia/README.md) · [Índice general](../README.md)
