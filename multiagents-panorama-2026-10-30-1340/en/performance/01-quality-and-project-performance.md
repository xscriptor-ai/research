---
title: "Quality and project performance — beyond the invoice"
description: "What happens to quality, reliability and delivery when agents accelerate a project, and how to measure it without self-deception."
date: "2026-10-30"
tags: ["technology", "agents", "quality", "performance", "projects"]
---

# Quality and project performance — beyond the invoice

> *A project is not measured by how fast the agent writes, but by how fast what it wrote stops hurting.*

## 1. What “project performance” means

It is not a single metric. It is a set of dimensions that move in different directions:

| Dimension | What it measures | Sign of improvement |
|---|---|---|
| **Quality** | Correctness, security, maintainability | Fewer escaped defects, less rework |
| **Reliability** | Consistency across runs | Fewer intermittent failures, less “works sometimes” |
| **Delivery** | Lead time, frequency, review time | Less time from idea to merge |
| **Change failure rate** | Percentage of changes that break something | Fewer rollbacks and hotfixes |
| **MTTR** | Recovery time | Less time to diagnose and fix |
| **Maintainability** | Cost of changing the code later | Less coupling, clearer tests |
| **Team sustainability** | Review and context load | Fewer meetings, less rework, less fatigue |

Acceleration without quality is debt: the project moves faster toward the next fire. Quality without delivery is a museum. The goal is **sustainable flow**, not raw speed.

## 2. The evidence for

- **Review coverage**: Anthropic reports that its multi-agent research system beat the single agent by **90.2%** on its internal research evaluation ([multi-agent research system](https://www.anthropic.com/research/multi-agent-research-system)). The reason is not that each agent is smarter, but that several independent branches cover more surface.
- **Clean context**: a reviewer that does not drag the implementation catches biases the author can no longer see. It is the same “fresh eyes” argument used in human review.
- **Verification parallelism**: tests, static analysis, security review and performance review can run in separate branches and converge into one report. Coverage rises if the branches do not collide.
- **Less noisy context**: Chroma shows that performance degrades with context length and noise; a subagent with a focused context can perform better on its subtask ([Context Rot](https://research.trychroma.com/context-rot)).

## 3. The evidence against

- **Lossy handoff**: the subagent returns a summary, not its trace. Implicit decisions — why it chose an API, what it discarded, what assumption it made — are lost. Cognition calls this the “telephone game” and warns that two agents can make incompatible decisions without knowing it ([Don't Build Multi-Agents](https://www.cognition.ai/blog/dont-build-multi-agents)).
- **Write conflicts**: two agents editing the same file or package produce merges, regressions and rework. Generation speed does not compensate for reconciliation speed.
- **Style inconsistency**: four agents designing UI without a design system produce four styles. Visual quality drops even if each screen is correct in isolation.
- **Review bottleneck**: if agents produce ten times more code, the human reviewer is still one. Project throughput does not improve; it moves to the reviewer.
- **Coordination complexity**: more agents = more states, messages and dependencies. System quality can fall even if each piece improves.

Anthropic itself found that, across several enterprise deployments, **a single agent with skills outperformed the subagent architecture on quality, and often at lower cost and latency**; the subagent remains for isolation, parallelism or permissions ([When to fold subagents](https://claude.com/resources/articles/building-agents-when-to-fold-subagents-into-your-main-agent-loop)). It is the most important correction in the dossier: more agents is not more quality by default.

## 4. The effect on the project

### 4.1. Webapp

| Phase | Effect of 1→2→4 | Condition |
|---|---|---|
| Discovery | **Improves** | Independent research branches |
| API design | **Improves with a contract** | One owner of the schema; agents do not invent it |
| Frontend | **Partial improvement** | Disjoint components and fixed design tokens |
| Backend | **Partial improvement** | Disjoint endpoints and stable contract |
| Integration | **Worsens** | More branches = more merges and cross-tests |
| Review | **Improves** | Independent reviewers with clean context |
| Maintenance | **Risk** | Generated code without an owner or tests |

Greenfield webapp is the best case: boundaries are clear and the contract can be fixed before fan-out. An existing webapp is worse: contracts are implicit in the code and the agent discovers them late.

### 4.2. Linux distro

| Phase | Effect of 1→2→4 | Condition |
|---|---|---|
| Package research | **Improves** | Independent branches per package/family |
| Licences and compliance | **Improves** | Parallel review with fixed criteria |
| Kernel/initramfs | **Risk** | Shared configuration and dependencies |
| Installer | **Risk** | State and integration tests |
| Reproducible build | **Worsens** | One dependency graph; agents compete |
| Branding/docs | **Improves** | Independent branches |
| QA in VMs | **Partial improvement** | Parallel test matrix, but failure triage is the bottleneck |

A distro is a **pipeline with gates**, not a fan. Quality depends on reproducibility and integration, not on how many agents write PKGBUILDs in parallel.

## 5. Metrics that do not lie

| Metric | How to measure | What it indicates |
|---|---|---|
| **Acceptance rate** | Accepted changes / proposed changes | Real quality, not volume |
| **Rework** | Correction hours per task | Hidden cost |
| **Change failure rate** | Changes requiring rollback/hotfix | Fragility |
| **Lead time** | Idea → merge | Project flow |
| **Review time** | Wait + human review | Bottleneck |
| **Test coverage** | % of behaviour covered | Safety net |
| **Escaped defects** | Bugs found post-merge | Verification quality |
| **MTTR** | Recovery time | Operability |
| **Sustainability** | Review hours per week | Human cost |

They do not all move together. A fan-out can improve acceptance and worsen lead time; it can reduce exploration time and increase integration time. Measure the **whole flow**, not the phase the agent accelerated.

## 6. The design that works

1. **Contract first.** API schema, types, design tokens and a file list per agent. Without a contract, parallelism produces incompatible interpretations.
2. **One owner per file.** If two agents can touch the same file, it is not parallelisable.
3. **Independent review.** The reviewer should not see the implementation; it should see the diff and the contract.
4. **One integrator.** Integration is serial by design; do not distribute it across agents.
5. **Tests as a gate.** An agent is not “done” until the test passes; a summary without verification is not a result.
6. **Small roster.** 5–10 agents with distinct triggers; the rest, skills.
7. **Measure per accepted task.** Generation speed is not the metric; the accepted change is.

## 7. Closing

Quality does not scale with the number of agents; it scales with the quality of the boundaries. A well-designed 1→2→4 can cover more surface, review with fresh eyes and reduce discovery time. The same tree, badly designed, can produce four styles, eight conflicts and an exhausted reviewer.

The difference is not in the fan-out. It is in the contract, the file and the gate.

---

**Next:** [Cases and variables](02-cases-and-comparisons.md) — languages, apps, documentation and papers.

**Back:** [Acceleration and latency](00-acceleration-and-latency.md) · [Index](../README.md)
