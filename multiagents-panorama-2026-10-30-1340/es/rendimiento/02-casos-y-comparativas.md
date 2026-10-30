---
title: "Casos y variables — programación, escritura, apps y papers"
description: "Matriz de escenarios para decidir cuándo un 1→2→4 acelera y cuándo sólo añade coordinación: lenguajes, tipos de aplicación, documentación y artículos científicos."
date: "2026-10-30"
tags: ["tecnología", "agentes", "rendimiento", "programación", "escritura"]
---

# Casos y variables — programación, escritura, apps y papers

> *No hay una arquitectura para todo. Hay tareas que se dejan cortar y tareas que sólo se dejan contar.*

## 1. La pregunta correcta

«¿Sirve 1→2→4 para una webapp?» es una pregunta incompleta. La pregunta útil es:

**¿Qué variables de esta tarea permiten repartirla sin perder decisiones, y cuánto cuesta verificar que el reparto no rompió nada?**

Un mismo lenguaje puede ser buen o mal caso según el proyecto. Un paper y una distro comparten más de lo que parece: ambos tienen una parte paralelizable (literatura, paquetes) y una parte secuencial (argumento, build). La diferencia está en dónde vive el camino crítico.

## 2. Las cinco variables

| Variable | Qué mide | Alta cuando… | Baja cuando… |
|---|---|---|---|
| **D — Descomponibilidad** | ¿Se puede partir en subtareas con entregables claros? | Módulos, capítulos, paquetes, endpoints | Monolito, argumento único, estado compartido |
| **C — Acoplamiento** | ¿Cuánto depende una parte de las decisiones de otra? | Contratos implícitos, mismos archivos, mismo grafo | Interfaces estables, archivos disjuntos |
| **V — Verificabilidad** | ¿Hay un gate objetivo? | Tests, tipos, compilador, rúbrica, citas | «Queda bien», revisión subjetiva |
| **L — Localidad del contexto** | ¿La subtarea necesita poco contexto externo? | Función, capítulo, componente | Refactor transversal, coherencia global |
| **I — Coste de iteración** | ¿Cuánto cuesta equivocarse y repetir? | Build lento, hardware, migraciones | Script, borrador, test rápido |

Una heurística de *fitness* para 1→2→4, en escala 0–5:

$$
F = 0{,}30D + 0{,}25V + 0{,}20L + 0{,}15(5-C) + 0{,}10(5-I)
$$

Los pesos son una convención, no una ley. Cambialos según tu proyecto; lo importante es que la decisión no la tome sólo «quiero paralelizar».

## 3. Matriz de casos

Escala 1–5. «Fit» es la afinidad con 1→2→4, no con cualquier subagente.

| Caso | D | C | V | L | I | Fit | Topología recomendada |
|---|---:|---:|---:|---:|---:|---:|---|
| Webapp greenfield | 5 | 2 | 5 | 3 | 2 | **4,1** | 1→2 (frontend/backend) → 4 módulos disjuntos |
| Webapp existente | 3 | 4 | 4 | 2 | 3 | **2,7** | 1→2 máximo; revisión en paralelo |
| CLI / herramienta | 4 | 2 | 5 | 4 | 1 | **4,1** | 1→3 (comandos, tests, docs) |
| Librería / SDK | 4 | 2 | 5 | 4 | 2 | **4,0** | 1→2 (API, tests) + revisión |
| Backend / servicio | 4 | 3 | 5 | 3 | 2 | **3,7** | 1→2 (endpoints, infra) |
| App móvil | 3 | 3 | 4 | 3 | 3 | **3,0** | 1→2 (UI, lógica) + dispositivo |
| App de escritorio | 3 | 3 | 4 | 3 | 2 | **3,1** | 1→2 (UI, core) |
| Embedded / firmware | 1 | 5 | 2 | 2 | 5 | **1,2** | Hilo único + revisión; hardware-in-the-loop |
| Data / ML pipeline | 4 | 3 | 3 | 3 | 3 | **3,1** | 1→2 (datos, modelo) + validación |
| Distro Linux: build | 1 | 5 | 2 | 1 | 5 | **1,0** | Pipeline serial con gates |
| Distro Linux: investigación | 4 | 1 | 4 | 4 | 1 | **4,0** | 1→N por paquete/familia |
| Documentación técnica | 5 | 1 | 4 | 4 | 1 | **4,3** | 1→N capítulos + un editor |
| Ensayo / libro | 3 | 4 | 2 | 2 | 2 | **2,3** | Investigación en paralelo; escritura única |
| Paper científico | 3 | 4 | 4 | 2 | 2 | **2,8** | Literatura en paralelo; argumento y resultados en serie |
| Traducción | 5 | 2 | 3 | 4 | 1 | **3,9** | 1→N segmentos + glosario + un revisor |

La tabla dice lo que la intuición suele olvidar: **la documentación técnica es el caso más paralelizable de la lista**, y **un paper es más secuencial que una app móvil** porque el argumento y la verificación de citas son un cuello de botella.

## 4. Programación por lenguaje

El lenguaje importa menos que el contrato y el gate, pero no es neutro. Cambian el coste de iteración, la calidad de la verificación y el riesgo de que el agente «compiles y rompa».

| Lenguaje | Iteración | Verificación | Riesgo típico | Fit 1→2→4 |
|---|---|---|---|---|
| **Python** | Muy rápida | Tests + tipos opcionales | Errores en runtime; imports dinámicos | Alto para scripts, data, automatización |
| **TypeScript / JS** | Rápida | `tsc` + tests; contrato de tipos | Deriva de tipos entre módulos; `any` | Muy alto si hay contrato y monorepo |
| **Go** | Rápida | Compilador + tests; tooling simple | Interfaces implícitas; concurrencia | Alto para backend/CLI; paquetes disjuntos |
| **Rust** | Lenta | Compilador + borrow checker | El agente itera mucho; ciclos de compilación | Medio: bueno para review/refactor acotado, malo para 8 hojas |
| **Java / Kotlin** | Media | Compilador + tests; Gradle/Maven | Contexto pesado; configuración | Medio-alto por módulos, si el build no se toca |
| **C / C++** | Lenta | Compilador + tests + sanitizers | UB, memoria, build systems | Bajo para escritura paralela; alto para review |
| **SQL / datos** | Media | Constraints + tests de datos | Migraciones y esquema compartido | Bajo para cambios de esquema; alto para consultas |
| **Shell** | Muy rápida | `shellcheck` + tests | Efectos laterales, portabilidad | Medio; mejor un agente por script |

### 4.1. Lo que cambia de verdad

- **TypeScript** tiene una ventaja enorme para 1→2→4: el compilador es un gate barato. Si el contrato de tipos está fijado, dos agentes pueden trabajar en frontend y backend sin hablar. Si no, el `any` esconde la incompatibilidad hasta la integración.
- **Rust** tiene el gate más fuerte, pero también el ciclo de iteración más caro. Un agente que pelea con el borrow checker puede consumir más turnos que el trabajo original. Para 1→2→4, mejor dividir por *crates* con interfaces estables que por archivos.
- **Go** es el caso más favorable para backend/CLI: compilación rápida, tooling uniforme y paquetes con límites claros. El riesgo está en la concurrencia y en las interfaces implícitas.
- **Python** es ideal para scripts y datos, pero el tipado opcional hace que el gate dependa de tests. Sin tests, el subagente «funciona» hasta que no.
- **C/C++ y embedded** son el peor caso para fan-out de escritura: el estado compartido, el build y el hardware hacen que la integración sea el trabajo. Ahí el subagente sirve para revisar, no para editar en paralelo.
- **Java/Kotlin** escala bien por módulos, pero el contexto de Spring/Gradle es pesado; el agente necesita más contexto y el build es un cuello de botella.

## 5. Creación de aplicaciones

### 5.1. Por tipo de aplicación

| Tipo | Parte paralelizable | Parte serial | Fit |
|---|---|---|---|
| Webapp greenfield | Frontend, backend, tests, docs | Contrato de API, integración, auth | **Alto** |
| API / backend | Endpoints, repositorios, tests | Esquema, migraciones, autenticación | **Alto-medio** |
| CLI | Comandos, parsers, tests | UX de comandos, integración | **Alto** |
| Librería | Módulos, docs, ejemplos | API pública, semver | **Alto** |
| App móvil | Pantallas, lógica, tests | Navegación, estado, dispositivo | **Medio** |
| Desktop | UI, core, packaging | Integración con SO, updater | **Medio** |
| Embedded | Drivers aislados, tests de host | Hardware, timing, memoria | **Bajo** |
| Data/ML | Ingesta, features, evaluación | Esquema, leakage, validación | **Medio** |

### 5.2. Polyglot: el contrato manda

Un proyecto con TypeScript en el frontend, Go en el backend y Python para datos no se reparte «por lenguaje». Se reparte **por contrato**:

1. Definí el esquema de API y los tipos compartidos.
2. Asigná un dueño por servicio o paquete.
3. Paralelizá sólo lo que no comparte archivos ni migraciones.
4. Integrá en un solo hilo y corré el gate completo.

Si el contrato no existe, tres agentes en tres lenguajes producen tres interpretaciones del mismo campo. La diversidad de lenguajes no es el problema; la ausencia de contrato sí.

## 6. Escritura y papers

### 6.1. Documentación técnica

Es el caso más paralelizable de la lista. Capítulos, referencias, ejemplos y guías son independientes; el riesgo es la **voz** y la **terminología**. Solución: glosario, style guide y un editor único que integra. 1→N funciona mejor que 1→2→4; no necesitás una jerarquía, necesitás segmentos disjuntos.

### 6.2. Ensayo / libro

La investigación puede ir en paralelo; la escritura no. La voz, el argumento y el ritmo son un estado global. Si cuatro agentes escriben cuatro capítulos, el libro se lee como cuatro libros. Topología: un escritor principal, subagentes de investigación y verificación, un editor.

### 6.3. Paper científico

Un paper tiene cuatro fases con distinto fit:

| Fase | Paralelizable | Topología | Riesgo |
|---|---|---|---|
| Revisión de literatura | **Sí** | 1→N búsquedas/lecturas | Citas inventadas, fuentes duplicadas |
| Argumento / outline | **No** | Hilo único | Coherencia, originalidad |
| Métodos / resultados | **Parcial** | 1→2 (métodos, experimentos) | Reproducibilidad, leakage |
| Redacción | **Parcial** | Un escritor + revisores | Voz, precisión |
| Citas / formato | **Sí** | 1→N verificación | Referencias mal atribuidas |

El paper es un caso **medio**: se acelera la literatura y la verificación, no la tesis. La regla es la misma que en código: **el argumento es el contrato; la evidencia es el gate**.

### 6.4. Traducción

Muy paralelizable por segmentos, con dos condiciones: glosario fijo y un revisor que unifique. Sin glosario, 8 agentes producen 8 traducciones del mismo término. La verificación es lingüística, no automática; el gate es la revisión.

## 7. Scores y decisión

Con la fórmula heurística:

| Caso | D | C | V | L | I | F |
|---|---:|---:|---:|---:|---:|---:|
| Documentación técnica | 5 | 1 | 4 | 4 | 1 | **4,3** |
| Webapp greenfield | 5 | 2 | 5 | 3 | 2 | **4,1** |
| CLI | 4 | 2 | 5 | 4 | 1 | **4,1** |
| Distro: investigación | 4 | 1 | 4 | 4 | 1 | **4,0** |
| Traducción | 5 | 2 | 3 | 4 | 1 | **3,9** |
| App móvil | 3 | 3 | 4 | 3 | 3 | **3,0** |
| Paper | 3 | 4 | 4 | 2 | 2 | **2,8** |
| Ensayo | 3 | 4 | 2 | 2 | 2 | **2,3** |
| Embedded | 1 | 5 | 2 | 2 | 5 | **1,2** |
| Distro: build | 1 | 5 | 2 | 1 | 5 | **1,0** |

La conclusión no es «programación sí, escritura no». Es:

- **Alto**: documentación, investigación de distro, webapp greenfield, CLI, librerías, traducción.
- **Medio**: paper, app móvil, backend, data/ML, Rust/Java por módulos.
- **Bajo**: embedded, build de distro, ensayo de voz única, refactor acoplado, C/C++ paralelo.

## 8. Cierre

El lenguaje, el tipo de app y el género de escritura cambian el coste y el gate, pero no cambian la ley: **se paraleliza lo que tiene límites claros y se escribe en serie lo que tiene voz o estado global**. Un paper y una distro se parecen más de lo que parece; una webapp y un libro, menos de lo que parece.

La pregunta final sigue siendo la misma: **¿cuál es la parte que no se puede cortar?** Esa parte decide la topología. Todo lo demás es fan-out.

---

**Volver:** [Calidad y rendimiento del proyecto](01-calidad-y-rendimiento-del-proyecto.md) · [Índice](../README.md)
