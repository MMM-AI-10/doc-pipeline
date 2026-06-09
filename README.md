# doc-pipeline

A methodology and toolset for building LLM-deterministic documentation — so any AI agent, reading your docs in any order, builds the same mental model of your system.

---

## The Problem

LLMs hallucinate on bad documentation. Not because they're broken — because documentation is ambiguous, incomplete, or contradictory in ways humans tolerate but machines can't. A floating pronoun, an "etc." without a full list, a link to an empty file — each one is a point where an LLM fills the gap from its own weights instead of your actual system. The result: agents generate code that doesn't match reality, answer questions with plausible but wrong facts, and silently choose one version when two documents disagree.

A linter checks form — file exists, link works, syntax valid. This system checks **meaning**: if an LLM reads this documentation, will it understand the system correctly?

---

## Four Principles

**1. E2E by Documentation.** A question is a test case. No answer in docs = red. Contradiction between two files = red. Just like an e2e test traces a request through services, a documentation e2e traces a question through files.

**2. Determinism.** Any agent, reading files in any order, must arrive at one model of the system. If two agents read the same docs and build different models — the documentation is non-deterministic.

**3. What's Not Written Doesn't Exist.** LLMs don't infer, don't extrapolate, don't fill from context. If a rule exists in only one file — for every other participant, it doesn't exist. An empty file = a non-existent feature.

**4. Absolute Coverage.** No service is unimportant, no service is auxiliary. An incomplete analysis isn't called "analysis" — it's called "read N of M, the rest requires a second pass."

---

## The Layer Pyramid

Every service is documented as a pyramid of files. Each layer expands the previous one — never repeats it.

```
README.md
  Entry point. One paragraph per aspect + link.
  Rule: if something takes more than 7 lines, it belongs in the next layer.

business-logic.md
  Business meaning in plain words. Why the service exists, for whom, by what rules.
  No tech stack. No HTTP/Kafka/Redis. Only meaning.
  Readable by a non-technical manager.

service-logic.md
  Technical structure. Components, architecture, full interaction table.
  This is where all specs below grow from.

flows.md
  Step-by-step scenarios with concrete requests and payloads.
  Reads like a test case: trigger → step 1 → step 2 → final state.

entities.md / api-contracts.md / dataflow.md / databases.md
  Specs. Exact fields, types, topics, tables.
  Only facts, no explanations — those live above.

rbac.md / errors.md / security.md / configs.md / observability.md / ...
  Specialized files. Each covers its own domain and audience.
```

The #1 disease of documentation: **README gravity**. Information accumulates in README because it's the first file people open. business-logic.md stays empty. Specs contain only template stubs. When this happens — the pyramid is broken, and agents build incomplete models with 100% confidence.

---

## Quick Start

1. Read [METHODOLOGY.md](./METHODOLOGY.md) — the operating principles, analysis phases, and what "docs pass e2e" means.
2. Run [HALLUCINATION-TRAPS.md](./HALLUCINATION-TRAPS.md) as a checklist on every file — 18 traps that reliably cause LLM hallucinations.
3. Use [COLLAPSE-PROTOCOL.md](./COLLAPSE-PROTOCOL.md) when you find two files describing the same thing differently.
4. Pick templates from [/templates/](./templates/) to structure your service docs.
5. Pick agents from [/agents/](./agents/) to analyze and fix your documentation.

---

## Contents

| File | What it is |
|------|-----------|
| [METHODOLOGY.md](./METHODOLOGY.md) | Operating principles, analysis phases, test generation, fix rules |
| [HALLUCINATION-TRAPS.md](./HALLUCINATION-TRAPS.md) | 18 traps that cause LLM hallucinations, with regex signals and examples |
| [COLLAPSE-PROTOCOL.md](./COLLAPSE-PROTOCOL.md) | Protocol for marking silent LLM choices between contradictory descriptions |
| [/templates/](./templates/) | Documentation templates for each pyramid layer |
| [/agents/](./agents/) | Agent prompts for analyzing, fixing, and creating documentation |
| [/examples/](./examples/) | Good docs, bad docs, and collapse examples |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | How to contribute |
| [LICENSE](./LICENSE) | MIT |
