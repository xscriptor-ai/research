---
title: "Cases and variables — programming, writing, apps and papers"
description: "Scenario matrix for deciding when 1→2→4 accelerates and when it only adds coordination: languages, application types, documentation and research papers."
date: "2026-10-30"
tags: ["technology", "agents", "performance", "programming", "writing"]
---

# Cases and variables — programming, writing, apps and papers

> *There is no architecture for everything. There are tasks that can be cut and tasks that can only be told.*

## 1. The right question

“Does 1→2→4 work for a webapp?” is an incomplete question. The useful one is:

**Which variables of this task allow it to be split without losing decisions, and how much does it cost to verify that the split did not break anything?**

The same language can be a good or bad case depending on the project. A paper and a distro have more in common than they appear: both have a parallelisable part (literature, packages) and a sequential part (argument, build). The difference is where the critical path lives.

## 2. The five variables

| Variable | What it measures | High when… | Low when… |
|---|---|---|---|
| **D — Decomposability** | Can it be split into subtasks with clear deliverables? | Modules, chapters, packages, endpoints | Monolith, single argument, shared state |
| **C — Coupling** | How much does one part depend on another's decisions? | Implicit contracts, same files, same graph | Stable interfaces, disjoint files |
| **V — Verifiability** | Is there an objective gate? | Tests, types, compiler, rubric, citations | “Looks good”, subjective review |
| **L — Context locality** | Does the subtask need little external context? | Function, chapter, component | Cross-cutting refactor, global coherence |
| **I — Iteration cost** | How expensive is a mistake and a retry? | Slow build, hardware, migrations | Script, draft, fast test |

A fitness heuristic for 1→2→4, on a 0–5 scale:

$$
F = 0.30D + 0.25V + 0.20L + 0.15(5-C) + 0.10(5-I)
$$

The weights are a convention, not a law. Change them for your project; what matters is that the decision is not made by “I want to parallelise” alone.

## 3. Case matrix

Scale 1–5. “Fit” is affinity with 1→2→4, not with any subagent.

| Case | D | C | V | L | I | Fit | Recommended topology |
|---|---:|---:|---:|---:|---:|---:|---|
| Greenfield webapp | 5 | 2 | 5 | 3 | 2 | **4.1** | 1→2 (frontend/backend) → 4 disjoint modules |
| Existing webapp | 3 | 4 | 4 | 2 | 3 | **2.7** | 1→2 max; parallel review |
| CLI / tool | 4 | 2 | 5 | 4 | 1 | **4.1** | 1→3 (commands, tests, docs) |
| Library / SDK | 4 | 2 | 5 | 4 | 2 | **4.0** | 1→2 (API, tests) + review |
| Backend / service | 4 | 3 | 5 | 3 | 2 | **3.7** | 1→2 (endpoints, infra) |
| Mobile app | 3 | 3 | 4 | 3 | 3 | **3.0** | 1→2 (UI, logic) + device |
| Desktop app | 3 | 3 | 4 | 3 | 2 | **3.1** | 1→2 (UI, core) |
| Embedded / firmware | 1 | 5 | 2 | 2 | 5 | **1.2** | Single thread + review; hardware-in-the-loop |
| Data / ML pipeline | 4 | 3 | 3 | 3 | 3 | **3.1** | 1→2 (data, model) + validation |
| Linux distro: build | 1 | 5 | 2 | 1 | 5 | **1.0** | Serial pipeline with gates |
| Linux distro: research | 4 | 1 | 4 | 4 | 1 | **4.0** | 1→N per package/family |
| Technical documentation | 5 | 1 | 4 | 4 | 1 | **4.3** | 1→N chapters + one editor |
| Essay / book | 3 | 4 | 2 | 2 | 2 | **2.3** | Parallel research; single writer |
| Research paper | 3 | 4 | 4 | 2 | 2 | **2.8** | Parallel literature; serial argument and results |
| Translation | 5 | 2 | 3 | 4 | 1 | **3.9** | 1→N segments + glossary + one reviewer |

The table says what intuition often forgets: **technical documentation is the most parallelisable case on the list**, and **a paper is more sequential than a mobile app** because the argument and citation verification are the bottleneck.

## 4. Programming by language

The language matters less than the contract and the gate, but it is not neutral. Iteration cost, verification quality and the risk of “it compiles and breaks” change.

| Language | Iteration | Verification | Typical risk | 1→2→4 fit |
|---|---|---|---|---|
| **Python** | Very fast | Tests + optional types | Runtime errors; dynamic imports | High for scripts, data, automation |
| **TypeScript / JS** | Fast | `tsc` + tests; type contract | Type drift between modules; `any` | Very high with a contract and monorepo |
| **Go** | Fast | Compiler + tests; simple tooling | Implicit interfaces; concurrency | High for backend/CLI; disjoint packages |
| **Rust** | Slow | Compiler + borrow checker | The agent iterates a lot; compile cycles | Medium: good for scoped review/refactor, bad for 8 leaves |
| **Java / Kotlin** | Medium | Compiler + tests; Gradle/Maven | Heavy context; configuration | Medium-high by modules, if the build is untouched |
| **C / C++** | Slow | Compiler + tests + sanitizers | UB, memory, build systems | Low for parallel writing; high for review |
| **SQL / data** | Medium | Constraints + data tests | Migrations and shared schema | Low for schema changes; high for queries |
| **Shell** | Very fast | `shellcheck` + tests | Side effects, portability | Medium; better one agent per script |

### 4.1. What actually changes

- **TypeScript** has a huge advantage for 1→2→4: the compiler is a cheap gate. If the type contract is fixed, two agents can work on frontend and backend without talking. If not, `any` hides the incompatibility until integration.
- **Rust** has the strongest gate, but also the most expensive iteration cycle. An agent fighting the borrow checker can consume more turns than the original work. For 1→2→4, split by *crates* with stable interfaces, not by files.
- **Go** is the most favourable case for backend/CLI: fast compilation, uniform tooling and packages with clear boundaries. The risk is concurrency and implicit interfaces.
- **Python** is ideal for scripts and data, but optional typing makes the gate depend on tests. Without tests, the subagent “works” until it does not.
- **C/C++ and embedded** are the worst case for write fan-out: shared state, build and hardware make integration the work. There, the subagent is for review, not parallel editing.
- **Java/Kotlin** scales well by modules, but Spring/Gradle context is heavy; the agent needs more context and the build is a bottleneck.

## 5. Building applications

### 5.1. By application type

| Type | Parallelisable part | Serial part | Fit |
|---|---|---|---|
| Greenfield webapp | Frontend, backend, tests, docs | API contract, integration, auth | **High** |
| API / backend | Endpoints, repositories, tests | Schema, migrations, authentication | **High-medium** |
| CLI | Commands, parsers, tests | Command UX, integration | **High** |
| Library | Modules, docs, examples | Public API, semver | **High** |
| Mobile app | Screens, logic, tests | Navigation, state, device | **Medium** |
| Desktop | UI, core, packaging | OS integration, updater | **Medium** |
| Embedded | Isolated drivers, host tests | Hardware, timing, memory | **Low** |
| Data/ML | Ingestion, features, evaluation | Schema, leakage, validation | **Medium** |

### 5.2. Polyglot: the contract rules

A project with TypeScript on the frontend, Go on the backend and Python for data is not split “by language”. It is split **by contract**:

1. Define the API schema and shared types.
2. Assign one owner per service or package.
3. Parallelise only what shares no files or migrations.
4. Integrate in a single thread and run the full gate.

If the contract does not exist, three agents in three languages produce three interpretations of the same field. Language diversity is not the problem; the absence of a contract is.

## 6. Writing and papers

### 6.1. Technical documentation

It is the most parallelisable case on the list. Chapters, references, examples and guides are independent; the risk is **voice** and **terminology**. Solution: glossary, style guide and one editor who integrates. 1→N works better than 1→2→4; you do not need a hierarchy, you need disjoint segments.

### 6.2. Essay / book

Research can run in parallel; writing cannot. Voice, argument and rhythm are global state. If four agents write four chapters, the book reads like four books. Topology: one main writer, research and verification subagents, one editor.

### 6.3. Research paper

A paper has four phases with different fit:

| Phase | Parallelisable | Topology | Risk |
|---|---|---|---|
| Literature review | **Yes** | 1→N searches/reads | Invented citations, duplicate sources |
| Argument / outline | **No** | Single thread | Coherence, originality |
| Methods / results | **Partial** | 1→2 (methods, experiments) | Reproducibility, leakage |
| Writing | **Partial** | One writer + reviewers | Voice, precision |
| Citations / formatting | **Yes** | 1→N verification | Misattributed references |

The paper is a **medium** case: literature and verification accelerate, the thesis does not. The rule is the same as in code: **the argument is the contract; the evidence is the gate**.

### 6.4. Translation

Highly parallelisable by segments, with two conditions: a fixed glossary and one reviewer who unifies. Without a glossary, 8 agents produce 8 translations of the same term. Verification is linguistic, not automatic; the gate is review.

## 7. Scores and decision

With the heuristic formula:

| Case | D | C | V | L | I | F |
|---|---:|---:|---:|---:|---:|---:|
| Technical documentation | 5 | 1 | 4 | 4 | 1 | **4.3** |
| Greenfield webapp | 5 | 2 | 5 | 3 | 2 | **4.1** |
| CLI | 4 | 2 | 5 | 4 | 1 | **4.1** |
| Distro: research | 4 | 1 | 4 | 4 | 1 | **4.0** |
| Translation | 5 | 2 | 3 | 4 | 1 | **3.9** |
| Mobile app | 3 | 3 | 4 | 3 | 3 | **3.0** |
| Paper | 3 | 4 | 4 | 2 | 2 | **2.8** |
| Essay | 3 | 4 | 2 | 2 | 2 | **2.3** |
| Embedded | 1 | 5 | 2 | 2 | 5 | **1.2** |
| Distro: build | 1 | 5 | 2 | 1 | 5 | **1.0** |

The conclusion is not “programming yes, writing no”. It is:

- **High**: documentation, distro research, greenfield webapp, CLI, libraries, translation.
- **Medium**: paper, mobile app, backend, data/ML, Rust/Java by modules.
- **Low**: embedded, distro build, single-voice essay, coupled refactor, parallel C/C++.

## 8. Closing

Language, application type and writing genre change the cost and the gate, but not the law: **parallelise what has clear boundaries, and write serially what has voice or global state**. A paper and a distro have more in common than they appear; a webapp and a book, less.

The final question remains the same: **which part cannot be cut?** That part decides the topology. Everything else is fan-out.

---

**Back:** [Quality and project performance](01-quality-and-project-performance.md) · [Index](../README.md)
