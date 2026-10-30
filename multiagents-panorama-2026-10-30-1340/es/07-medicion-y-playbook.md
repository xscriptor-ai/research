---
title: "Medición y playbook — cómo saber si el ahorro es real"
description: "Métricas, instrumentación, diseño de experimentos y reglas de decisión para delegar sin autoengaño."
date: "2026-10-30"
tags: ["tecnología", "agentes", "medición", "economía"]
---

# Medición y playbook — cómo saber si el ahorro es real

> *El token se cuenta solo; el ahorro hay que demostrarlo. La diferencia entre ambos es el método.*

## 1. Por qué las anécdotas se contradicen

En la misma semana se publican dos relatos opuestos:

- «Delegar el trabajo sucio a subagentes redujo mi factura un 40%».
- «Cinco subagentes en paralelo agotaron mi límite de Pro en 15 minutos».

Los dos pueden ser ciertos. El primero suele medir **una sesión larga** en la que el subagente evitó arrastrar un contexto enorme; el segundo, **un fan-out** en el que cinco arranques compitieron por la misma ventana. La diferencia no es la herramienta: es la topología, la longitud de la sesión y la métrica.

Sin método, el debate se resuelve con la anécdota que confirma la intuición previa. Con método, se resuelve con una tabla.

## 2. Las métricas

| Métrica | Definición | Por qué importa |
|---|---|---|
| \(C_{\text{run}}\) | Coste de una ejecución | La factura inmediata |
| \(P_{\text{éxito}}\) | Tareas aceptadas / tareas intentadas | La fiabilidad |
| \(C_{\text{aceptada}}\) | \(C_{\text{run}} / P_{\text{éxito}}\) | El coste real por resultado |
| \(Q\) | Puntuación de rúbrica (0–100) | La calidad |
| \(T_{\text{pared}}\) | Tiempo hasta el resultado | El coste humano |
| \(M\) | Tokens delegados / tokens directos | El multiplicador de tokens |
| \(H\) | Tasa de acierto de caché | La eficiencia del contexto |
| \(R\) | Retrabajo (minutos humanos) | El coste que no aparece en la factura |
| \(E\) | \(Q / C_{\text{aceptada}}\) | Eficiencia económica |

La métrica de decisión es \(C_{\text{aceptada}}\), no \(C_{\text{run}}\). Un agente que cuesta $1,50 y acierta el 90% ($1,67) puede ser mejor que uno que cuesta $0,80 y acierta el 60% ($1,33) si el segundo genera 20 minutos de revisión. A $60/hora, 20 minutos son $20: el «barato» es 12× más caro.

## 3. Instrumentación

### 3.1. Claude Code

- **`/usage`**: coste de sesión, uso por modelo, atribución a skills/subagentes/plugins/MCP en planes de suscripción, línea de caché.
- **Transcripciones**: `~/.claude/projects/<proyecto>/<sesión>/` y `subagents/agent-<id>.jsonl`. Cada mensaje del asistente lleva `usage` con `input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`.
- **Trampa**: el streaming escribe varias líneas con el mismo `message.id` y el mismo `usage`. **Deduplica por `message.id`** o triplicarás la cuenta.
- **OpenTelemetry**: exporta tokens y coste por usuario/sesión; es la única opción que llega a un stack propio en tiempo real.
- **`/context`**: qué ocupa la ventana; útil para ver el peso de `CLAUDE.md`, tools y skills.

Comando mínimo para leer un subagente:

```bash
for f in ~/.claude/projects/*/SESSION_ID/subagents/agent-*.jsonl; do
  jq -s -c --arg f "$(basename "$f")" '
    map(select(.message.usage)) | unique_by(.message.id) | map(.message.usage)
    | {agent: $f, requests: length,
       input: (map(.input_tokens)|add),
       cache_write: (map(.cache_creation_input_tokens)|add),
       cache_read: (map(.cache_read_input_tokens)|add),
       output: (map(.output_tokens)|add)}' "$f"
done
```

### 3.2. Codex

- **`/status`** en la CLI: modelo y uso restante.
- **Dashboard**: `chatgpt.com/codex/settings/usage`.
- **API**: logs de la organización y la respuesta; el coste se calcula con la tabla de precios.
- **Subagentes**: la app muestra hilos y resúmenes; no hay una atribución pública por subagente equivalente a la de Claude Code. Para un análisis fino, instrumenta el proveedor o el gateway.

### 3.3. OpenCode

- **Coste de sesión** en la TUI; `opencode stats` agrega tokens y coste estimado.
- **Console**: presupuestos por workspace y miembro; recarga automática.
- **Sesiones hijas**: el coste se agrega a la sesión; la atribución por subagente requiere exportar o instrumentar.
- **Modelos free**: el coste no es cero si el precio son tus datos; la métrica económica debe incluir la política de datos.

### 3.4. Denominador común

Para comparar entre herramientas, normaliza:

- **Mismo modelo o mismo nivel** (no Opus vs Luna sin decirlo).
- **Mismo esfuerzo/razonamiento**.
- **Mismo prompt de sistema y mismas herramientas**.
- **Mismo conjunto de tareas y mismo criterio de aceptación**.
- **Misma ventana temporal** (la cuota cambia con el tiempo).

## 4. Diseño del experimento

### 4.1. Set de tareas

20–30 tareas representativas, estratificadas por tipo:

| Tipo | Ejemplos | Por qué |
|---|---|---|
| Exploración | «¿Dónde se valida el token?» | Contexto desechable |
| Logs/tests | «¿Qué tests fallan y por qué?» | Salida voluminosa |
| Revisión | «Revisa este diff» | Contexto limpio |
| Implementación pequeña | «Corrige este typo» | Debe quedarse en el padre |
| Implementación acoplada | «Cambia esta interfaz en tres módulos» | El contexto importa |
| Investigación | «Compara estas dos librerías» | Paralelismo |

Cada tarea necesita:

- un **resultado esperado** verificable (test, diff, lista de archivos, rúbrica);
- un **criterio de aceptación** binario o una rúbrica de 0–100;
- un **coste de referencia** (ejecución directa);
- un **timeout** y un **presupuesto máximo**.

### 4.2. Diseño pareado

Para cada tarea, ejecuta:

- **A**: hilo principal, sin delegación.
- **B**: con subagente(s), misma tarea y mismo modelo principal.
- **C** (opcional): con subagente barato (Haiku/Luna) y principal caro.

Repite 3 veces cada celda, con el mismo modelo y esfuerzo, y aleatoriza el orden para evitar efectos de calentamiento de caché. Si puedes, usa dos repositorios o dos ramas para evitar contaminación.

### 4.3. Análisis

- **Mediana y rango intercuartílico**, no media: los costes de agente tienen colas gruesas.
- **Bootstrap pareado** para el intervalo de confianza de la diferencia de coste y de éxito.
- **Informa el tamaño del efecto**: «B cuesta 1,8× tokens y acierta 8 puntos más» es más útil que un p-valor.
- **Separa por tipo de tarea**: el promedio global esconde que la delegación gana en exploración y pierde en edición.
- **Incluye el coste humano**: minutos de revisión × tarifa horaria.

### 4.4. Tamaño de muestra

Con 20 tareas puedes detectar efectos grandes (≥30% de diferencia) si el diseño es pareado. Para efectos del 10–15%, necesitas 50–100 tareas o más repeticiones. No concluyas con 3 tareas y una anécdota.

## 5. Reglas de decisión

### 5.1. Delega si…

- el trabajo produce **mucho output desechable** (búsquedas, logs, tests);
- el trabajo es **independiente** y se puede paralelizar;
- necesitas **contexto limpio** para revisar o verificar;
- hay un **modelo más barato** que puede hacerlo;
- la sesión principal tiene **muchos turnos por delante**;
- el coste de un fallo es bajo o el subagente tiene permisos estrechos.

### 5.2. No delegues si…

- la tarea es **pequeña** (menos de ~30k tokens de trabajo);
- el trabajo es **secuencial** y cada paso depende del anterior;
- dos agentes tocarían **el mismo archivo**;
- el subagente necesita **todo el historial** para decidir;
- el **arranque** cuesta más que la tarea;
- no puedes **verificar** el resultado.

### 5.3. Umbrales prácticos

| Señal | Umbral | Acción |
|---|---|---|
| Output esperado | >50k tokens | Delegar a explorador |
| Trabajo del subagente | <30k tokens | Quedarse en el padre |
| Turnos restantes | <20 | No delegar por ahorro |
| Diferencia de modelo | ≥2× | Enrutar |
| Diferencia de modelo | ≥20× | Enrutar aunque el arranque duela |
| Tareas independientes | ≥3 | Paralelizar, con límite |
| Archivos compartidos | ≥1 | No paralelizar escritura |

Los umbrales son heurísticas, no leyes. Úsalos para decidir qué medir, no para sustituir la medición.

## 6. Playbook por escenario

| Escenario | Arquitectura | Modelo | Por qué |
|---|---|---|---|
| «¿Dónde está la lógica de auth?» | Subagente explorador | Fast | Output grande, respuesta pequeña |
| «¿Por qué falla este test?» | Subagente de triaje | Fast/Balanced | Logs voluminosos, resumen corto |
| «Revisa este PR» | Subagente revisor | Balanced/Frontier | Contexto limpio; el juicio importa |
| «Implementa esta feature pequeña» | Hilo principal | Balanced | Arranque > tarea |
| «Refactoriza tres módulos acoplados» | Hilo principal o fork | Frontier | Decisiones compartidas |
| «Compara tres librerías» | 3 subagentes en paralelo | Fast/Balanced | Paralelismo real |
| «Investiga un tema amplio» | Equipo o multiagente | Frontier + Balanced | Breadth-first; 7–15× tokens |
| «Arregla todos los errores de tipos» | 2–3 subagentes por paquete | Balanced | Independientes por archivo |
| «Revisa seguridad antes de commit» | Subagente con permisos de sólo lectura | Frontier | Riesgo alto; contexto limpio |
| «Documenta este módulo» | Hilo principal + skill | Balanced | Contexto compartido |

## 7. Anti-patrones

1. **Agent sprawl.** 205 agentes instalados, 205 descripciones cargadas, 205 formas de elegir mal. Poda.
2. **Fan-out de escritura.** Cinco agentes editando el mismo repo: conflictos, merges y retrabajo.
3. **«Devuélveme un informe completo».** El informe entra en el padre y se paga en cada turno. Pide listas, rutas, hallazgos.
4. **Subagente para un typo.** El arranque cuesta más que el arreglo.
5. **Haiku para un prompt de 130k.** El umbral de 100k multiplica el precio por 5.
6. **Cambiar de modelo a mitad de tarea.** Invalida la caché; pagas la relectura.
7. **No poner `maxTurns`/`steps`.** Un subagente puede explorar hasta agotar la ventana.
8. **Medir tokens totales y llamarlo ahorro.** El total puede subir mientras el coste por tarea aceptada baja.
9. **Ignorar la cuota.** En suscripción, el recurso escaso es la ventana, no el dólar.
10. **Convertir sin manifiesto.** Un agente que carga pero amplía permisos no es una conversión; es un riesgo.

## 8. Plan de 30 días

### Semana 1 — línea base

- Instala sólo 8–12 agentes relevantes.
- Mide 10 tareas sin delegación: coste, tiempo, éxito.
- Registra la cuota consumida por sesión.
- Mide el tamaño de `CLAUDE.md`/`AGENTS.md` y de las descripciones.

### Semana 2 — una delegación

- Elige el trabajo más voluminoso y desechable (búsqueda o logs).
- Créale un subagente con modelo barato, `omitClaudeMd`/tools mínimas y salida acotada.
- Compara A/B en 10 tareas.
- Ajusta el prompt de delegación, no el modelo, primero.

### Semana 3 — enrutamiento

- Identifica las tareas que pueden usar un modelo 2–20× más barato.
- Crea dos o tres agentes con niveles `fast`/`balanced`.
- Mide coste por tarea aceptada, no sólo tokens.
- Revisa la cuota: ¿el fan-out la agota?

### Semana 4 — poda y conversión

- Elimina los agentes que no usaste.
- Mide L1/L2 del conversor (carga y capacidades).
- Si conviertes, emite manifiesto y revisa los agentes que amplían permisos.
- Documenta las tres tareas que seguirán en el hilo principal.

## 9. Cierre

Medir es la forma de distinguir el ahorro del movimiento. Un subagente puede reducir la factura, la latencia, la ventana o el riesgo; rara vez las cuatro a la vez. El método sirve para elegir cuál de las cuatro estás comprando.

La pregunta final no es «¿delego o no?». Es «¿qué estoy dispuesto a pagar por no recordar esto?». Si la respuesta es clara, la arquitectura se sigue sola.

---

**Siguiente:** [Fuentes](08-fuentes.md) — las referencias, la calidad de la evidencia y lo que no pudimos verificar.
