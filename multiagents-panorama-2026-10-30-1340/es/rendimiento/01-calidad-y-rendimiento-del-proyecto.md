---
title: "Calidad y rendimiento del proyecto — más allá de la factura"
description: "Qué le pasa a la calidad, la fiabilidad y la entrega cuando se acelera con agentes, y cómo medirlo sin autoengaño."
date: "2026-10-30"
tags: ["tecnología", "agentes", "calidad", "rendimiento", "proyectos"]
---

# Calidad y rendimiento del proyecto — más allá de la factura

> *Un proyecto no se mide por lo rápido que escribe el agente, sino por lo rápido que deja de doler lo que escribió.*

## 1. Qué significa «rendimiento del proyecto»

No es una métrica única. Es un conjunto de dimensiones que se mueven en direcciones distintas:

| Dimensión | Qué mide | Señal de que mejora |
|---|---|---|
| **Calidad** | Corrección, seguridad, mantenibilidad | Menos defectos escapados, menos retrabajo |
| **Fiabilidad** | Consistencia entre ejecuciones | Menos fallos intermitentes, menos «funciona a veces» |
| **Entrega** | Lead time, frecuencia, tiempo de revisión | Menos tiempo desde la idea hasta el merge |
| **Change failure rate** | Porcentaje de cambios que rompen algo | Menos rollbacks y hotfixes |
| **MTTR** | Tiempo de recuperación | Menos tiempo para diagnosticar y arreglar |
| **Mantenibilidad** | Coste de cambiar el código después | Menos acoplamiento, tests más claros |
| **Sostenibilidad del equipo** | Carga de revisión y contexto | Menos reuniones, menos retrabajo, menos fatiga |

La aceleración sin calidad es deuda: el proyecto va más rápido hacia el siguiente incendio. La calidad sin entrega es un museo. El objetivo es **flujo sostenible**, no velocidad bruta.

## 2. La evidencia a favor

- **Cobertura de revisión**: Anthropic reporta que el multiagente de investigación superó al agente único en **90,2%** en su evaluación interna de investigación ([multi-agent research system](https://www.anthropic.com/research/multi-agent-research-system)). La razón no es que cada agente sea más listo, sino que varias ramas independientes cubren más superficie.
- **Contexto limpio**: un revisor que no arrastra la implementación detecta sesgos que el autor ya no ve. Es el mismo argumento de «fresh eyes» que usa la revisión humana.
- **Paralelismo de verificación**: tests, análisis estático, revisión de seguridad y revisión de rendimiento pueden correr en ramas separadas y converger en un informe. La cobertura sube si las ramas no se pisan.
- **Menos contexto ruidoso**: Chroma muestra que el rendimiento se degrada con la longitud y el ruido del contexto; un subagente con un contexto enfocado puede rendir mejor en su subtarea ([Context Rot](https://research.trychroma.com/context-rot)).

## 3. La evidencia en contra

- **Handoff lossy**: el subagente devuelve un resumen, no su traza. Las decisiones implícitas —por qué eligió una API, qué descartó, qué supuesto asumió— se pierden. Cognition lo llama «teléfono descompuesto» y advierte que dos agentes pueden tomar decisiones incompatibles sin saberlo ([Don't Build Multi-Agents](https://www.cognition.ai/blog/dont-build-multi-agents)).
- **Conflictos de escritura**: dos agentes editando el mismo archivo o el mismo paquete producen merges, regresiones y retrabajo. La velocidad de generación no compensa la velocidad de reconciliación.
- **Inconsistencia de estilo**: cuatro agentes diseñando UI sin un design system producen cuatro estilos. La calidad visual baja aunque cada pantalla sea correcta por separado.
- **Cuello de botella de revisión**: si los agentes producen diez veces más código, el revisor humano sigue siendo uno. El throughput del proyecto no mejora; se mueve al revisor.
- **Complejidad de coordinación**: más agentes = más estados, más mensajes, más dependencias. La calidad del sistema puede caer aunque la calidad de cada pieza suba.

La propia Anthropic encontró que, en varias implantaciones empresariales, **un solo agente con skills superó en calidad y a menudo en coste y latencia** a la arquitectura de subagentes; el subagente queda para aislamiento, paralelismo o permisos ([When to fold subagents](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)). Es la corrección más importante del dossier: más agentes no es más calidad por defecto.

## 4. El efecto sobre el proyecto

### 4.1. Webapp

| Fase | Efecto de 1→2→4 | Condición |
|---|---|---|
| Descubrimiento | **Mejora** | Ramas de investigación independientes |
| Diseño de API | **Mejora si hay contrato** | Un dueño del esquema; los agentes no lo inventan |
| Frontend | **Mejora parcial** | Componentes disjuntos y design tokens fijos |
| Backend | **Mejora parcial** | Endpoints disjuntos y contrato estable |
| Integración | **Empeora** | Más ramas = más merges y pruebas cruzadas |
| Revisión | **Mejora** | Revisores independientes con contexto limpio |
| Mantenimiento | **Riesgo** | Código generado sin dueño ni tests |

La webapp greenfield es el mejor caso: los límites son claros y el contrato se puede fijar antes del fan-out. La webapp existente es peor: los contratos ya están implícitos en el código y el agente los descubre tarde.

### 4.2. Distro Linux

| Fase | Efecto de 1→2→4 | Condición |
|---|---|---|
| Investigación de paquetes | **Mejora** | Ramas independientes por paquete/familia |
| Licencias y compliance | **Mejora** | Revisión paralela con criterios fijos |
| Kernel/initramfs | **Riesgo** | Configuración compartida y dependencias |
| Instalador | **Riesgo** | Estado y pruebas de integración |
| Build reproducible | **Empeora** | Un solo grafo de dependencias; los agentes compiten |
| Branding/docs | **Mejora** | Ramas independientes |
| QA en VMs | **Mejora parcial** | Matriz de pruebas paralela, pero cuello de botella en fallos |

La distro es un **pipeline con gates**, no un abanico. La calidad depende de la reproducibilidad y de la integración, no de cuántos agentes escriben PKGBUILDs en paralelo.

## 5. Métricas para no engañarse

| Métrica | Cómo se mide | Qué indica |
|---|---|---|
| **Tasa de aceptación** | Cambios aceptados / cambios propuestos | Calidad real, no volumen |
| **Retrabajo** | Horas de corrección por tarea | Coste oculto |
| **Change failure rate** | Cambios que requieren rollback/hotfix | Fragilidad |
| **Lead time** | Idea → merge | Flujo del proyecto |
| **Tiempo de revisión** | Espera + revisión humana | Cuello de botella |
| **Cobertura de tests** | % de comportamiento cubierto | Red de seguridad |
| **Defectos escapados** | Bugs encontrados post-merge | Calidad de la verificación |
| **MTTR** | Tiempo de recuperación | Operabilidad |
| **Sostenibilidad** | Horas de revisión por semana | Coste humano |

No todas se mueven a la vez. Un fan-out puede mejorar la tasa de aceptación y empeorar el lead time; puede reducir el tiempo de exploración y aumentar el de integración. Mide el **flujo completo**, no la fase que el agente aceleró.

## 6. El diseño que sí funciona

1. **Contrato primero.** Esquema de API, tipos, tokens de diseño y lista de archivos por agente. Sin contrato, el paralelismo produce interpretaciones incompatibles.
2. **Un dueño por archivo.** Si dos agentes pueden tocar el mismo archivo, no es paralelizable.
3. **Revisión independiente.** El revisor no debe ver la implementación; debe ver el diff y el contrato.
4. **Un solo integrador.** La integración es serial por diseño; no la repartas entre agentes.
5. **Tests como gate.** Un agente no «termina» hasta que el test pasa; el resumen sin verificación no es un resultado.
6. **Roster pequeño.** 5–10 agentes con triggers distintos; el resto, skills.
7. **Medición por tarea aceptada.** La velocidad de generación no es la métrica; el cambio aceptado sí.

## 7. Cierre

La calidad no escala con el número de agentes; escala con la calidad de los límites. Un 1→2→4 bien diseñado puede cubrir más superficie, revisar con ojos frescos y reducir el tiempo de descubrimiento. El mismo árbol mal diseñado puede producir cuatro estilos, ocho conflictos y un revisor agotado.

La diferencia no está en el fan-out. Está en el contrato, el archivo y el gate.

---

**Siguiente:** [Casos y variables](02-casos-y-comparativas.md) — lenguajes, apps, documentación y papers.

**Volver:** [Aceleración y latencia](00-aceleracion-y-latencia.md) · [Índice](../README.md)
