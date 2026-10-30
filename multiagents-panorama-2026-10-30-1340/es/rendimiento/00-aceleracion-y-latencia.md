---
title: "Aceleración y latencia — el otro lado de la factura"
description: "Qué acelera de verdad un subagente, qué sólo paraleliza, y por qué 1→2→4 puede ser más lento que un solo hilo."
date: "2026-10-30"
tags: ["tecnología", "agentes", "rendimiento", "latencia"]
---

# Aceleración y latencia — el otro lado de la factura

> *Paralelizar no es acelerar: es repartir la espera. Si la espera está en la integración, repartirla no la elimina.*

## 1. Tres métricas que no son la misma

| Métrica | Definición | Qué mide |
|---|---|---|
| **Latencia** | Tiempo desde la petición hasta el resultado aceptado | La espera del usuario |
| **Throughput** | Tareas aceptadas por unidad de tiempo | La capacidad del sistema |
| **Coste de la velocidad** | Tokens (o cuota) por unidad de tiempo ahorrada | Lo que pagas por ir más rápido |

Un subagente puede mejorar la latencia de una tarea y empeorar el throughput del proyecto; puede acelerar la exploración y ralentizar la integración. La pregunta correcta no es «¿va más rápido?», sino **«¿qué parte del camino crítico se acortó?»**.

## 2. La ley de Amdahl, sin escapatoria

Si una fracción \(p\) del trabajo es paralelizable y el resto es secuencial, el speedup con \(N\) workers es:

$$
S(N) = \frac{1}{(1-p) + p/N}
$$

| Fracción paralela \(p\) | N = 2 | N = 4 | N = 8 | Límite teórico |
|---:|---:|---:|---:|---:|
| 0,50 | 1,33× | 1,60× | 1,78× | 2,00× |
| 0,70 | 1,54× | 2,11× | 2,58× | 3,33× |
| 0,90 | 1,82× | 3,08× | 4,71× | 10,00× |
| 0,95 | 1,90× | 3,48× | 5,93× | 20,00× |

La consecuencia para 1→2→4: **los 8 leaves no aceleran lo que no es paralelizable**. Si la integración, los contratos, los tests finales o el build consumen el 30% del tiempo, el máximo es 3,3× aunque lances infinitos agentes. Si consumen el 50%, el máximo es 2×. El fan-out no rompe Amdahl; sólo lo disimula hasta que llega la integración.

## 3. La coordinación también tarda

Un modelo más realista:

$$
T(N) = T_s + \frac{T_p}{N} + cN
$$

donde \(T_s\) es el trabajo secuencial, \(T_p\) el paralelizable, \(c\) el coste de coordinar cada agente (mensajes, esperas, conflictos, revisión). El óptimo no es «todos los agentes», sino:

$$
N^* = \sqrt{\frac{T_p}{c}}
$$

Ejemplo: \(T_s=1\) h, \(T_p=8\) h, \(c=0,25\) h por agente. \(N^* \approx 5,7\); con 6 agentes, \(T \approx 3,8\) h frente a 9 h de un solo hilo: **2,4×**. Si la coordinación sube a \(c=1\) h, \(N^* \approx 2,8\) y el speedup cae a **1,35×**. Más agentes, menos aceleración.

Ese es el argumento matemático contra 1→2→4 por defecto: **11 agentes no son 11 veces más rápidos; son 11 veces más puntos de coordinación**.

## 4. La evidencia

| Fuente | Qué mide | Resultado | Lectura |
|---|---|---|---|
| Anthropic, Research (2025) | Investigación multiagente | Hasta **90% menos tiempo** en consultas complejas | El paralelismo real acelera la exploración |
| Anthropic, Research (2025) | Coste del mismo sistema | ~15× tokens vs chat | La velocidad se paga |
| Anthropic, Applied AI (2026) | Bucle único vs multiagente | **~2× más rápido** con un solo agente | Menos agentes puede ser más rápido |
| Claude Code, costes (2026) | Equipos de agentes | ~7× tokens vs sesión estándar | La coordinación tiene factura |
| Anécdota pública (2026) | 5 subagentes en paralelo | ~15 min vs ~30 min secuencial | 2× de velocidad, más cuota |
| Este dossier | 1→2→4 con 8 hojas | Limitado por la hoja más lenta + integración | El árbol no elimina el camino crítico |

Las dos primeras filas explican por qué el multiagente es atractivo en investigación: el trabajo es *breadth-first*, hay muchas ramas independientes y el resultado es un resumen. Las dos siguientes explican por qué no se traduce automáticamente a coding: el trabajo es secuencial, comparte archivos y la integración es el cuello de botella.

## 5. El impuesto de la velocidad

Si el fan-out multiplica los tokens por \(M\) y acelera por \(S\), el coste por unidad de velocidad es:

$$
\text{impuesto} = \frac{M}{S}
$$

| Escenario | M | S | Impuesto | Lectura |
|---|---:|---:|---:|---|
| 5 subagentes, 30k arranque, 100k trabajo | 2,5× | 2,0× | 1,25× | Pagas 25% más tokens por unidad de velocidad |
| Equipo de agentes en plan mode | 7,0× | 2,0× | 3,5× | La velocidad sale cara |
| 1→2→4 con 8 hojas | ~3–4× | 2,0× | 1,5–2,0× | Depende de la integración |
| Modelo Fast (Codex) | 2,0× cuota | <2,0× latencia | >1,0× | La cuota sube más que la velocidad |

El impuesto no dice «no lo hagas»; dice **cuánto estás pagando por el reloj**. Si el reloj vale más que la cuota, es racional. Si la cuota es el recurso escaso, el fan-out es un lujo.

## 6. Dónde sí acelera cada herramienta

| Herramienta | Mecanismo | Acelera bien | Acelera mal |
|---|---|---|---|
| **Claude Code** | Subagentes, fork, background, equipos | Exploración, review, tests en paralelo | Edición del mismo archivo, tareas acopladas |
| **Codex** | Hilos de agente, `max_concurrent_threads_per_session` | Exploración, revisión por lentes, tests | Fan-out de escritura, merges |
| **OpenCode v2** | Sesiones hijas, background, `steps` | Trabajo independiente, exploración | Anidamiento profundo, tareas con estado compartido |

Claude Code tiene fork: comparte la caché del padre y acelera sin pagar el arranque completo. Codex hereda sandbox y permisos, lo que reduce la configuración pero no la coordinación. OpenCode v2 permite background, pero su `general` no anida: el árbol 1→2→4 hay que construirlo con agentes custom y permisos, y el coste de coordinación sube.

## 7. 1→2→4: cuándo acelera y cuándo no

| Proyecto | ¿Acelera? | Por qué |
|---|---|---|
| **Webapp greenfield** | **Sí, con contrato primero** | Frontend/backend y módulos disjuntos; 8 hojas pueden trabajar en archivos distintos |
| **Webapp existente** | **Parcial** | Contratos y archivos compartidos; la integración domina |
| **Diseño UI/UX** | **Parcial** | 4 hojas inventando estilos producen inconsistencia; mejor un dueño del design system |
| **Distro Linux: investigación** | **Sí** | Paquetes, licencias, compatibilidad, arte y docs son ramas independientes |
| **Distro Linux: build** | **No** | PKGBUILD, kernel, initramfs, instalador, firmas e ISO comparten estado; la integración es serial |
| **Debugging con hipótesis** | **Sí** | Ramas independientes que compiten; el equipo contrasta |
| **Refactor acoplado** | **No** | Cada cambio depende del anterior; el resumen pierde detalle |

La regla práctica: **1→2→4 acelera cuando los 8 leaves escriben en archivos distintos y la integración es barata**. Si comparten archivos, contratos o estado de build, el árbol es una máquina de conflictos.

## 8. Cómo medirlo

- **Latencia**: tiempo hasta el resultado aceptado, no hasta el primer token.
- **Throughput**: tareas aceptadas por hora, no tareas lanzadas.
- **Tiempo de integración**: merges, conflictos, tests finales, revisión.
- **Tiempo de coordinación**: mensajes, esperas, reintentos, agentes idle.
- **Impuesto**: \(M/S\), tokens por unidad de velocidad.
- **Calidad**: si la velocidad sube y la tasa de aceptación baja, no hay aceleración real.

Un experimento mínimo: misma tarea, misma rama, 3 repeticiones; mide \(T_1\), \(T_N\), tokens y aceptación. Si \(S(N) < 1,2\) y el impuesto es >1,5, el fan-out no está comprando velocidad; está comprando actividad.

## 9. Cierre

La aceleración es real, pero es local: vive en las ramas independientes, no en el árbol entero. Un 1→2→4 puede reducir a la mitad la exploración y duplicar el tiempo de integración; el resultado neto depende de qué pesa más en el camino crítico.

La pregunta no es «¿cuántos agentes puedo lanzar en paralelo?» sino **«¿cuál es la hoja más lenta y cuánto tarda la integración?»**. Si la respuesta es la integración, más agentes sólo reparten la espera.

---

**Siguiente:** [Calidad y rendimiento del proyecto](01-calidad-y-rendimiento-del-proyecto.md) — qué pasa con la calidad, la fiabilidad y la entrega cuando se acelera.
