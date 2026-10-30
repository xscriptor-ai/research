---
title: "Fuentes — qué sabemos, cómo lo sabemos y qué falta"
description: "Bibliografía, calidad de la evidencia, tabla de afirmaciones y límites de esta investigación."
date: "2026-10-30"
tags: ["tecnología", "fuentes", "investigación", "agentes"]
---

# Fuentes — qué sabemos, cómo lo sabemos y qué falta

> *Una cifra sin fuente es una opinión con decimales.*

## 1. Cómo se construyó este dossier

- **Documentación primaria** de Anthropic, OpenAI y OpenCode, leída el **30 de octubre de 2026**.
- **Precios de lista** de las páginas oficiales, en USD por millón de tokens, con la fecha de lectura.
- **Mediciones independientes** de terceros, etiquetadas como tales y tratadas como órdenes de magnitud, no como benchmarks.
- **Repositorios locales** de xscriptor, clonados el 30/10/2026, para el estudio de caso del conversor.
- **Cálculos propios** a partir de las tarifas publicadas; las fórmulas y los supuestos están en los documentos 01 y 02.

La regla fue: ninguna afirmación de coste sin una categoría de token y un precio; ninguna afirmación de comportamiento sin una fuente o una prueba local.

## 2. Calidad de la evidencia

| Nivel | Tipo | Uso en el dossier |
|---|---|---|
| **A** | Documentación oficial del proveedor | Features, campos, precios, límites |
| **B** | Artículo técnico del proveedor con metodología | Multiagente 15×, 90,2%, context rot |
| **C** | Medición independiente con muestra pequeña | Arranque 12,4k–40,8k, anécdotas de cuota |
| **D** | Prensa/terceros | Fechas de GA, migración OpenCode, comparativas |
| **E** | Cálculo propio | Puntos de equilibrio, fan-out, coste por tarea |

Las fuentes A y B sostienen las conclusiones; las C y D ilustran; las E son reproducibles con las fórmulas.

## 3. Fuentes primarias

### Anthropic — Claude Code

1. **Create custom subagents.** https://code.claude.com/docs/en/sub-agents — campos, carga de contexto, fork, profundidad, concurrencia, modelos.
2. **Manage costs effectively.** https://code.claude.com/docs/en/costs — $13/día, $150–250/mes, 7× equipos, atribución, recomendaciones.
3. **How Claude Code uses prompt caching.** https://code.claude.com/docs/en/prompt-caching — prefijos, TTL, invalidación, subagentes y fork.
4. **Orchestrate teams of Claude Code sessions.** https://code.claude.com/docs/en/agent-teams — equipos, 7×, límites, mensajes.
5. **Extend Claude with skills.** https://code.claude.com/docs/en/skills — `SKILL.md`, frontmatter, `context: fork`, carga progresiva.
6. **How Claude remembers your project.** https://code.claude.com/docs/en/memory — `CLAUDE.md`, `AGENTS.md`, orden de lectura.
7. **Pricing.** https://platform.claude.com/docs/en/about-claude/pricing — tabla de precios, caché, batch, long context, tokenizador.

### Anthropic — investigación y práctica

8. **How we built our multi-agent research system.** https://www.anthropic.com/research/multi-agent-research-system — 4×/15×, 80% de varianza, 90,2%, límites en coding. Publicado el 13/06/2025.
9. **Building agents: When to fold subagents into your main agent loop.** https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop — agente único + skills, 2× más rápido, cuándo quedarse con subagentes. 22/10/2026.
10. **How and when to use subagents in Claude Code.** https://claude.com/blog/subagents-in-claude-code — patrones, cuándo no delegar, «too many specialist agents». 07/04/2026.

### OpenAI — Codex

11. **Subagents.** https://learn.chatgpt.com/docs/agent-configuration/subagents — roles, TOML, `[agents]`, herencia, concurrencia.
12. **Pricing.** https://learn.chatgpt.com/docs/pricing — planes, créditos, tarifas, Fast/Ultrafast, límites.
13. **Import from another agent.** https://developers.openai.com/codex/import — importación desde Claude Code/Cursor, subagentes, advertencias.
14. **API pricing.** https://platform.openai.com/docs/pricing — precios por token, caché, batch, contexto largo.

### OpenCode

15. **Agents (v1).** https://opencode.ai/docs/agents/ — primary/subagent, `permission`, `mode`, `steps`, `hidden`, built-ins.
16. **Agents (v2).** https://opencode.ai/v2/docs/agents/ — `agents`, `permissions`, `system`, `steps`, `disabled`, built-ins.
17. **Rules.** https://opencode.ai/docs/rules/ — `AGENTS.md`, fallback `CLAUDE.md` en v1, precedencia.
18. **Console pricing.** https://opencode.ai/docs/zen/ — tabla de modelos, caché, Go, free models.

### Estándares y ecosistema

19. **AGENTS.md.** https://agents.md/ — formato, 60k+ proyectos, Agentic AI Foundation/Linux Foundation, agentes compatibles.
20. **Rulesync.** https://github.com/dyoshikawa/rulesync — CLI de conversión, soporte de subagentes para Claude Code, Codex CLI, OpenCode y OpenCode V1.
21. **Cognition — Don't Build Multi-Agents.** https://www.cognition.ai/blog/dont-build-multi-agents — principios de contexto compartido y decisiones implícitas. 12/06/2025.
22. **Chroma — Context Rot.** https://research.trychroma.com/context-rot — 18 modelos, degradación por longitud, LongMemEval. 14/07/2025.

## 4. Fuentes secundarias

23. **Tokenade / DEV — Claude Code Subagents Token Usage: What They Really Cost.** https://dev.to/paul_irolla_6b5ae261d0224/claude-code-subagents-token-usage-what-they-really-cost-fp0 — 19 subagentes, 12.427–40.763 tokens de arranque, 66k–84k por ejecución (cita un informe de julio de 2026), deduplicación por `message.id`. 25/10/2026.
24. **Reveneau — Giving a Claude Code Subagent a Smaller Model.** https://reveneau.com/guides/claude-code-agent-costs/giving-a-subagent-a-smaller-model — aritmética de precios, aviso del tokenizador +30%. 04/10/2026.
25. **Shattered — OpenCode 2 Upgrade: 12 Steps, 50 Min.** https://shattered.io/opencode-2-upgrade-from-1x-12-steps-2026/ — migración v1→v2, permisos, AGENTS.md, plugins, fecha de v2. 27/10/2026.
26. **The Vibelog — OpenCode 2.0 vs Claude Code.** https://thevibelog.dev/blog/opencode-2-vs-claude-code-2026 — comparación de features y coste. 16/10/2026.
27. **Daniel Vaughan — Codex CLI in 2026.** https://codex.danielvaughan.com/2026/03/27/codex-cli-in-2026-whats-new/ — GA de subagentes en v0.115.0, 16/03/2026. 27/03/2026.

## 5. Repositorios locales

28. `xscriptor-ai/agents` — 205 agentes, 142 con `edit: allow`, 105 con `task: allow`, 202 con `webfetch: allow`, ~17.768 caracteres de descripciones; conversor en `claude/tools/xscriptor-convert.py`.
29. `xscriptor-ai/skills` — skills senior y web-fullstack; destino del conversor.
30. `xscriptor-ai/scripts` — `install-agents.sh`, instalación en `~/.config/opencode/agents`.
31. `xscriptor-ai/research` — este dossier.

Los conteos se hicieron con `find`/`grep` sobre los repos clonados el 30/10/2026. No se ejecutó el conversor completo porque el entorno no tenía `PyYAML` instalado; el análisis de su comportamiento se basa en la lectura del código.

## 6. Tabla de afirmaciones

| Afirmación | Valor | Fuente | Nivel |
|---|---|---|---|
| Agentes vs chat | ~4× tokens | Anthropic 8 | B |
| Multiagente vs chat | ~15× tokens | Anthropic 8 | B |
| Equipos vs sesión | ~7× tokens | Anthropic 2, 4 | A |
| 90,2% mejora multiagente | 90,2% | Anthropic 8 | B |
| Varianza explicada por tokens | 80% | Anthropic 8 | B |
| Arranque de subagente | 12.427–40.763 tokens | Tokenade 23 | C |
| Ejecución de subagente | 66k–84k tokens | Tokenade 23 (secundario) | C/D |
| Claude Code coste medio | $13/día; $150–250/mes | Anthropic 2 | A |
| Precio Haiku 5.5 ≤100k | $0,10/$0,50 | Anthropic 7 | A |
| Precio Sonnet 5.5 | $2,00/$10,00 | Anthropic 7 | A |
| Precio Opus 5.5 | $4,00/$20,00 | Anthropic 7 | A |
| Precio Fable 5.1 | $10,00/$50,00 | Anthropic 7 | A |
| Precio GPT-6.1 Sol | $2,00/$10,00 | OpenAI 14 | A |
| Precio GPT-6 Luna | $0,10/$0,50 | OpenAI 14 | A |
| Créditos Codex Sol | 50/2,5/250 | OpenAI 12 | A |
| GA de subagentes Codex | 16/03/2026 | Vaughan 27 | D |
| Claude lee AGENTS.md | Sí, si no hay CLAUDE.md | Anthropic 6 | A |
| OpenCode v2 sólo AGENTS.md | Sí | Shattered 25 | D |
| Rulesync soporta subagentes | Claude/Codex/OpenCode | Rulesync 20 | A |
| Codex importa subagentes | Sí | OpenAI 13 | A |
| xscriptor 205 agentes | 205 | Local 28 | A |
| Descripciones xscriptor | ~17.768 chars | Local 28 | A |
| Conversor xscriptor | OpenCode v1 → Claude | Local 28 | A |

## 7. Lo que no pudimos verificar

- **Coste por subagente en Codex.** No hay una vista pública equivalente a las transcripciones de Claude Code; el análisis se hace por hilo y modelo.
- **Comportamiento exacto de `CLAUDE.md` en OpenCode v2.** La documentación v1 lo soporta como fallback; la guía de migración v2 dice que sólo se descubre `AGENTS.md`. La página oficial de reglas v2 no estaba disponible en la fecha de lectura. Verifica en tu versión.
- **Carga real de los 205 agentes de xscriptor en Claude Code.** El conversor requiere `PyYAML`; no se ejecutó completo. La crítica de permisos se basa en la lectura del código.
- **Conversión de cuota de suscripción a dólares.** No está publicada; cualquier cifra que la presente como dólares es una estimación.
- **Efectos de calidad a largo plazo.** No hay un ensayo aleatorizado grande; los datos de Anthropic son internos y los de Chroma son de laboratorio.
- **Precios futuros.** Las tablas son una foto del 30/10/2026; los modelos y los precios cambian.
- **La cifra de 66k–84k.** Es secundaria (Tokenade citando un informe de julio de 2026); úsala como orden de magnitud.

## 8. Cómo citar este dossier

```bibtex
@techreport{xscriptor2026multiagents,
  title   = {Multiagents Panorama: The price and speed of delegating},
  author  = {Xscriptor},
  year    = {2026},
  month   = {October},
  note    = {Research dossier, xscriptor-ai/research},
  url     = {https://github.com/xscriptor-ai/research}
}
```

## 9. Cierre

La evidencia es suficiente para una conclusión y insuficiente para una ley. Sabemos que los subagentes aíslan contexto, que cuestan arranque, que el modelo pesa más que el aislamiento y que la conversión pierde semántica. No sabemos cuánto de eso se traduce en tu repositorio sin medirlo.

Esa ignorancia es la parte honesta del dossier. La otra parte es la fórmula.

---

**Volver:** [Índice](README.md) · [Resumen ejecutivo](economia/00-resumen-ejecutivo.md)
