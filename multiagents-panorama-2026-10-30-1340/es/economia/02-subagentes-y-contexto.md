---
title: "Subagentes y contexto — el ahorro que no está en el token"
description: "Qué aísla un subagente, qué cuesta, cuándo el aislamiento paga y por qué el agente único con skills vuelve a ganar en muchos casos."
date: "2026-10-30"
tags: ["tecnología", "agentes", "contexto", "economía"]
---

# Subagentes y contexto — el ahorro que no está en el token

> *Un subagente no reduce el trabajo: reduce lo que el trabajo deja en la memoria. La factura llega por los dos lados.*

## 1. Qué es un subagente, en términos de coste

Un subagente es tres cosas a la vez:

1. **Un contexto separado**: su propia ventana, su propio prompt de sistema, sus propias herramientas. No ve la conversación del padre; recibe un mensaje de delegación.
2. **Un bucle separado**: varias peticiones al modelo, con sus propias lecturas de archivos, llamadas a herramientas y resultados. Cada petición reenvía su contexto creciente.
3. **Un resumen de retorno**: al terminar, devuelve un texto al padre. Ese texto entra en el contexto principal y se paga en todos los turnos siguientes.

Esa separación tiene un beneficio y tres costes:

| Beneficio | Coste |
|---|---|
| El contexto ruidoso no contamina el hilo principal | **Arranque**: prompt, `CLAUDE.md`/`AGENTS.md`, git status, herramientas, skills |
| Se puede usar otro modelo, más barato o más especializado | **Trabajo**: el subagente ejecuta su propio bucle y paga sus tokens |
| Se puede paralelizar | **Retorno**: el resumen entra en el padre y se reenvía |
| Se puede aislar permisos y herramientas | **Coordinación**: elegir, lanzar, esperar, sintetizar y verificar |

El error habitual es contar sólo el beneficio («el contexto no está en el padre») y olvidar los tres costes. El segundo error es contar sólo el coste («los subagentes usan más tokens») y olvidar que el padre, sin ellos, también habría pagado por leer, compactar y arrastrar ese contexto.

## 2. La condición de ahorro

Sea:

- \(D\): tokens que el subagente mantiene fuera del hilo principal.
- \(T\): turnos que quedan en la sesión principal después de la delegación.
- \(r\): fracción del precio de entrada que cuesta una lectura de caché (0,05–0,1).
- \(S\): tokens de arranque del subagente.
- \(W\): tokens de trabajo del subagente (entrada).
- \(O\): tokens de salida del subagente.
- \(R\): tokens del resumen que vuelven al padre.
- \(c_w\): multiplicador de escritura de caché (1,25× o 2×).

El ahorro neto es:

$$
\Delta = D \cdot T \cdot r \cdot p_{\text{in,main}}
\;-\;
\left[ (c_w S + W) \cdot p_{\text{in,sub}} + O \cdot p_{\text{out,sub}} + R \cdot p_{\text{in,main}} \right]
$$

Si \(\Delta > 0\), el subagente ahorra **en dólares de API**. Si \(\Delta < 0\), puede seguir siendo racional por calidad, latencia o cuota, pero no por dinero. El cálculo trata \(W\) como entrada procesada una vez; un subagente que relee su contexto paga más, y uno que lo cachea paga menos.

### 2.1. La tabla que importa

Con \(D=100k\), \(S=30k\), \(W=100k\), \(O=10k\), \(R=2k\), caché de 5 minutos:

| Principal → subagente | 10 turnos | 25 turnos | 50 turnos | 100 turnos |
|---|---:|---:|---:|---:|
| Opus 5.5 → Opus 5.5 | −$0,56 | −$0,26 | +$0,24 | +$1,24 |
| Opus 5.5 → Sonnet 5.5 | −$0,18 | **+$0,12** | +$0,62 | +$1,62 |
| Opus 5.5 → Haiku 5.5 (>100k) | **+$0,10** | +$0,40 | +$0,90 | +$1,90 |
| Sonnet 5.5 → Sonnet 5.5 | −$0,28 | −$0,13 | +$0,12 | +$0,62 |
| Sonnet 5.5 → Haiku 5.5 (>100k) | +$0,002 | +$0,15 | +$0,40 | +$0,90 |

Tres lecturas:

1. **Con el mismo modelo, el aislamiento tarda en pagar.** A 5% de lectura de caché, sacar 100k tokens del padre ahorra $0,01 por turno en Sonnet; el subagente cuesta $0,38. Hacen falta ~38 turnos.
2. **El enrutamiento cambia la respuesta.** Opus principal + Sonnet subagente paga a partir de ~19 turnos; Opus + Haiku paga casi desde el principio, incluso con el umbral de 100k.
3. **El caso malo es el subagente caro en un principal barato.** Sonnet → Opus pierde dinero en casi todos los horizontes: se paga el modelo caro para ahorrar caché barata.

### 2.2. Lo que la fórmula no captura

- **Compactación evitada**: si el contexto del padre supera el umbral, la sesión resume y pierde detalle. El subagente puede evitar esa pérdida. El coste de una compactación es una petición grande más el riesgo de olvidar.
- **Calidad**: un contexto limpio puede producir una respuesta mejor; un contexto compartido puede producir una respuesta más coherente. La fórmula no decide cuál.
- **Latencia**: el paralelismo reduce el tiempo de pared aunque aumente tokens.
- **Cuota**: en suscripción, el coste marginal en dólares es cero hasta agotar la ventana; el recurso escaso es la ventana.
- **Riesgo**: un subagente con permisos amplios puede hacer más daño; uno con permisos estrechos puede no poder terminar.

## 3. El multiplicador del fan-out

Si lanzas \(N\) subagentes en paralelo, con un trabajo total \(W\) repartido entre ellos y un arranque \(S\) por agente, y el padre sintetiza, el coste de tokens crece aproximadamente:

$$
M_{\text{fan-out}} \approx 1 + \frac{N \cdot S}{W}
$$

donde \(W\) es el trabajo **total**, no el de cada agente. Con \(N=5\), \(W=100k\) y \(S=30k\), cada agente hace 20k de trabajo y paga 30k de arranque: \(1 + 5 \cdot 30/100 = 2{,}5\times\).

| N | S = 15k, W = 100k | S = 30k, W = 100k | S = 30k, W = 500k |
|---:|---:|---:|---:|
| 2 | 1,30× | 1,60× | 1,12× |
| 5 | 1,75× | 2,50× | 1,30× |
| 10 | 2,50× | 4,00× | 1,60× |

La fórmula es una aproximación: ignora el resumen, la salida y la coordinación, y supone que el trabajo se reparte sin duplicación. Aun así, muestra el orden de magnitud: **el fan-out es barato cuando el trabajo por agente es grande y el arranque pequeño; es caro cuando cada agente hace poco**.

El caso real que circula —cinco subagentes en paralelo agotando el límite de Pro en unos 15 minutos, frente a ~30 minutos en secuencial— es coherente con esta aritmética: cinco arranques, cinco bucles y cinco contextos compiten por la misma ventana ([DEV/Tokenade](https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0), anécdota secundaria). No es un benchmark, pero es la dirección correcta.

## 4. Los números de la industria

### 4.1. 4×, 15× y 7×

| Fuente | Qué mide | Multiplicador | Comentario |
|---|---|---|---|
| Anthropic, Research (2025) | Agentes vs chat | **~4×** | Un agente que usa herramientas |
| Anthropic, Research (2025) | Multiagente vs chat | **~15×** | Arquitectura de investigación |
| Anthropic, Claude Code (2026) | Equipos de agentes vs sesión estándar | **~7×** | Teammates en modo plan |
| Este dossier | Fan-out de 5 con 30k de arranque y 100k de trabajo | **2,5×** | Fórmula \(1+NS/W\) |

No son contradictorios: miden arquitecturas distintas. Un subagente puntual no cuesta 15×; un sistema de investigación con un líder y muchos subagentes, sí. Un equipo de agentes con contextos independientes y plan mode, 7×. La pregunta correcta es **qué multiplicador corresponde a tu topología**, no cuál es el número universal.

### 4.2. El dato de calidad

En la evaluación interna de Anthropic, un sistema multiagente con Opus 4 como líder y Sonnet 4 como subagentes superó a un solo Opus 4 en **90,2%** en tareas de investigación. El análisis encontró que el uso de tokens explicaba el **80%** de la varianza en BrowseComp, con el número de llamadas a herramientas y la elección de modelo como otros factores ([Anthropic, 2025](https://www.anthropic.com/research/multi-agent-research-system)).

Ese resultado es el argumento más fuerte a favor del multiagente: **más tokens compran más capacidad en tareas de exploración amplia**. Pero el propio artículo advierte que la mayoría de las tareas de coding tienen menos paralelismo real y que los agentes aún no son buenos coordinándose en tiempo real. La conclusión no es «multiagente para todo», sino «multiagente para lo que se parece a investigar, no para lo que se parece a editar el mismo archivo».

### 4.3. El contraargumento: el agente único con skills

En octubre de 2026, el equipo de Applied AI de Anthropic publicó una corrección importante: en varias implantaciones empresariales, **un solo agente con skills superó en calidad y a menudo en coste y latencia** a la arquitectura de subagentes. Un cliente vio un runtime **~2× más rápido** con el bucle único. La razón: el contexto compartido permite conectar decisiones, y la divulgación progresiva de skills y herramientas evita cargar todo de golpe. El artículo recomienda plegar subagentes en el bucle principal y quedarse con ellos sólo cuando:

- el bucle único se degrada por saturación de contexto;
- hay tareas realmente paralelas con contextos desechables;
- el agente no sabe elegir herramientas o hay instrucciones en conflicto.

([Anthropic, «When to fold subagents into your main agent loop», 22/10/2026](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop))

Eso no contradice a los subagentes: los sitúa. Un subagente es una herramienta de **aislamiento**, no una arquitectura por defecto. Cuando el problema es «demasiado contexto ruidoso», el subagente ayuda. Cuando el problema es «necesito que las decisiones de un paso informen al siguiente», el subagente estorba.

### 4.4. Context rot: por qué el contexto limpio importa

Chroma evaluó 18 modelos y encontró que el rendimiento se degrada de forma no uniforme a medida que crece la entrada, incluso en tareas simples. En LongMemEval, los prompts completos de ~113k tokens rindieron mucho peor que los prompts «enfocados» de ~300 tokens con la misma información relevante; el efecto apareció en todas las familias ([Chroma, Context Rot, 2025](https://research.trychroma.com/context-rot)). La conclusión: **no basta con que la información esté en la ventana; importa cuánto ruido la rodea**.

Eso favorece el aislamiento cuando el subagente puede recibir un contexto pequeño y relevante. Pero también explica por qué un subagente mal diseñado —que recibe un prompt vago y debe explorar a ciegas— puede producir un resumen peor que el padre. La calidad del contexto de entrada es la variable que decide.

## 5. La matriz de decisión

| Situación | Arquitectura recomendada | Por qué |
|---|---|---|
| Buscar en 50 archivos y devolver 5 hallazgos | **Subagente de exploración** | Contexto desechable, salida pequeña, modelo barato |
| Editar tres archivos acoplados | **Hilo principal** | Decisiones compartidas; el resumen perdería detalle |
| Revisar un PR con ojos frescos | **Subagente revisor** | Contexto limpio, permisos de sólo lectura |
| Implementar un cambio pequeño | **Hilo principal** | El arranque cuesta más que la tarea |
| Investigar un tema amplio en paralelo | **Varios subagentes / equipo** | Paralelismo real, contextos independientes |
| Depurar con hipótesis competidoras | **Equipo de agentes** | Discusión y contraste; 7× tokens |
| Tarea larga con mucho ruido de tools | **Subagente + resumen** | Evita context rot y compactación |
| Tarea que depende de 20 turnos de decisiones | **Hilo principal o fork** | El fork conserva el contexto y la caché |
| Necesitas el mismo contexto pero aislado | **Fork** | Comparte caché; más barato que un subagente fresco |
| Necesitas permisos distintos | **Subagente con tools/permissions** | Aislamiento de capacidades |

## 6. Fork, subagente y equipo

| | **Subagente fresco** | **Fork** | **Equipo de agentes** |
|---|---|---|---|
| Contexto | Nuevo, sólo el prompt de delegación | Copia del padre | Nuevo, independiente |
| Caché | Propia, empieza fría | Comparte la del padre | Propia |
| Modelo | Configurable | El del padre | Configurable por teammate |
| Comunicación | Devuelve un resumen | Devuelve un resultado | Mensajes entre pares y lista de tareas |
| Coste | Arranque + trabajo | Trabajo + reutilización de caché | Arranque + trabajo + coordinación |
| Uso ideal | Trabajo desechable | Misma tarea, mismo contexto | Discusión, revisión, ownership dividido |

El fork es la opción infravalorada: comparte el prompt, las herramientas y la conversación, así que su primera petición lee la caché del padre. Para una tarea lateral que necesita el contexto acumulado, suele ser más barato que un subagente fresco. Su límite: arrastra todo el contexto, incluido el ruido.

## 7. Cómo decidir en la práctica

Una secuencia de cuatro preguntas:

1. **¿El contexto es la tarea o es ruido?** Si el subagente necesita el historial para decidir, no lo aísles; usa el hilo principal o un fork. Si el historial es ruido, aísla.
2. **¿El trabajo es independiente?** Si dos subagentes tocan el mismo archivo o dependen del resultado del otro, la coordinación se come el ahorro.
3. **¿Hay un modelo más barato que pueda hacerlo?** Si sí, el enrutamiento es la palanca; el aislamiento es secundario.
4. **¿Cuántos turnos quedan?** Si la sesión termina en cinco turnos, el arranque no se amortiza. Si quedan cincuenta, el contexto aislado pesa.

Si las cuatro respuestas apuntan a «sí», delega. Si alguna apunta a «no», mide antes de delegar.

## 8. Cierre

El subagente es una tecnología de la atención, no del descuento. Compra la posibilidad de mirar un problema sin arrastrar todo lo anterior; cobra por crear un mundo nuevo y por traducir lo que encontró a un resumen. Como toda traducción, pierde algo. La pregunta es si lo que pierde importa más que el ruido que evita.

Cuando la respuesta es no, el subagente ahorra. Cuando es sí, el subagente sólo hace que el problema parezca más ordenado.

---

**Siguiente:** [Aceleración y latencia](../rendimiento/00-aceleracion-y-latencia.md) — qué acelera de verdad el fan-out y qué frena la integración.
