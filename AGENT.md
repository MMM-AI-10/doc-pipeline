# AGENT.md — Pipeline Methodology

> This document is the shared foundation for all documentation pipeline agents.
> Every agent reads this before starting work — not as instructions for one agent, but as their common operating basis.

---

## Principle: Documentation Is a System

In a regular e2e test, a request passes through a chain of services — we verify each step.
In a documentation e2e, a question passes through a chain of files — we verify each answer.

```
Regular e2e:
  request → service A → service B → service C → response
  check: each service returned the correct result?

Documentation e2e:
  question → file A → file B → file C → answer
  check: each file gives a correct, complete, consistent answer?
```

**Core rule:** What's not written doesn't exist. LLMs don't infer, don't extrapolate, don't fill from context. If a rule exists in only one file — for all other participants, it doesn't exist.

---

## The Layer Pyramid

```
README.md           → paragraph + link. Navigation only. > 7 lines → next layer.
business-logic.md   → why, for whom, by what rules. No tech stack.
service-logic.md    → components, architecture, interaction table.
flows.md            → step-by-step scenarios with payloads.
specs               → exact fields, types, topics, tables.
```

---

## Pipeline Agents

| Agent | Role |
|-------|------|
| **DocReader** | Extracts machine understanding — not finding problems, building a model of what the docs say |
| **DocProbe** | Technical consistency, hallucination traps, collapse detection |
| **DocFix** | Fixes contradictions and technical errors found by DocProbe |
| **DocMirror** | Business logic symmetry — does each service know why it exists? |
| **DocOracle** | Answers specific questions — or proves the answer doesn't exist in docs |
| **DocNarrator** | Creates narrative summaries for humans — short, readable, marks oddities |
| **DocFiller** | Fills gaps found by DocMirror / DocAudit — adds only, never edits |

---

## Analysis Phases

### Phase 0 — Study the System (always first)

1. Read root README.md → extract: service list, main flow, key terms, explicit constraints
2. Read the service template → structure is the source of truth
3. Formulate business questions → each question = test case

### Phase 1 — Generate Tests from Documentation

For each file, three questions:

```
CLAIM: what does this file assert?
ECHO: who else should confirm this assertion?
GAP: what SHOULD this file describe but doesn't?
```

Five test types: Presence, Consistency, Completeness, Flow, Contradiction.

### Phase 2 — Traverse the Corpus

```
Round 1: root README → system map
Round 2: services in main flow order → upstream/downstream awareness
Round 3: cross-cutting files → consistency
Round 4: remaining services → orphans, contradictions, gaps
```

Infrastructure services are read MANDATORILY after the main pass — agents organically skip them.

### Phase 3 — Narrative Coherence

Violation patterns:
- **Blind spot** — A describes B, B doesn't mention A
- **Telephone game** — meaning changed somewhere in the chain
- **Orphan** — concept described nowhere else
- **Split personality** — same term, different meaning in different files
- **Flow abyss** — A sends data D, B doesn't describe receiving it

### Phase 4 — Severity Levels

| Level | Meaning |
|-------|---------|
| CRITICAL | Docs contradict themselves |
| HIGH | Important info exists in one place, absent where it must also be |
| MEDIUM | Coherence break, not contradiction |
| LOW | Terminology inconsistency |

### Phase 5 — Fix Principles

1. Minimal intervention — fix only what's broken
2. Preserve voice — same language, same tone
3. Add, don't delete — if info in A but not B → add to B
4. More detailed file = reference
5. Log every change

---

## Iatrogenia — What to Avoid

```
❌ "exactly 15 files" → crushes context
❌ "top 5 problems" → silences the rest
❌ "describe briefly" → nuances lost
❌ rigid output format → bends reality

✅ "files by template" → template = truth
✅ "all findings" → no limit
✅ "write as much as needed" → volume = content
```

---

## Definition of "Documentation Passes E2E"

1. All business questions have unambiguous answers in the right files
2. Answers are consistent — same thing described identically everywhere
3. Relationship graph is symmetric — every A→B described from both sides
4. Flow traceable from entry to exit
5. Terminology unified — one term, one meaning
6. No orphans — every document answers someone's question
