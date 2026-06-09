# Doc Pipeline — Methodology

> This document describes the **principles and methodology** for working with documentation as a deterministic system. It is read by all pipeline agents as their foundation — not as instructions for a specific agent, but as their shared basis.

---

## Principle: Documentation Is a System

In a regular e2e test, a request passes through a chain of services — we verify each step. In a documentation e2e, a question passes through a chain of files — we verify each answer.

```
Regular e2e:
  request → service A → service B → service C → response
  check: did each service return the correct result?

Documentation e2e:
  question → file A → file B → file C → answer
  check: does each file give a correct, complete, consistent answer?
```

**Key difference from a linter:** A linter checks form — does the file exist, are links intact. Documentation e2e checks meaning — is the narrative consistent, does the documentation answer business questions, can a machine understand it correctly?

**Core rule:** What's not written doesn't exist. LLMs don't infer, don't extrapolate, don't fill from context. If a rule exists in only one file — for every other participant, it doesn't exist.

---

## The Layer Pyramid — Law of Distribution

Every service is built as a pyramid of layers. Each layer expands the previous one — never repeats it.

```
README.md      → paragraph + link. Navigation only. If > 7 lines → next layer.
business-logic.md → why, for whom, by what rules. No tech stack.
service-logic.md  → components, architecture, interaction table.
flows.md          → step-by-step scenarios with payloads.
specs             → exact fields, types, topics, tables.
```

**Three pyramid traps (critical for all agents):**

```
TRAP-14 — Information on the wrong layer:
  Business rules in README instead of business-logic.md.
  Agent builds an incomplete model with 100% confidence.
  Detector: README > 150 lines or contains business rules.

TRAP-15 — Link to an empty file:
  README says "details → business-logic.md", but that file contains only template stubs.
  Agent is confident details exist — they don't.
  Detector: target file contains only ___ without real content.

TRAP-16 — Template stub accepted as fact:
  File contains unfilled ___ or {placeholder}.
  Agent sees "a table row" and considers the field documented.
  Detector: grep for "___" patterns in file.
```

**Two rules for all agents:**

```
TRAP-17 (confabulation): every concrete fact about the system = quote from file.
  If fact not found → "Not documented in [file]". Never guess.

TRAP-18 (silent collapse): any choice between two versions → COLLAPSE marker.
  Minimal [COLLAPSE:UNRESOLVED] is better than silence.
  COLLAPSE:LAYER — when README and business-logic describe the same thing (pyramid violated).
```

---

## Phase 0 — Study the System (always first)

Before any work with documentation — build a map of the system.

**Step 0.1 — Map.** Read the root README.md. Extract:
- List of all services and their architectural layers
- Main message flow (critical path)
- Key terms and concepts
- What the system explicitly does NOT do

This is your reference. Everything else is checked against it.

**Step 0.2 — Service template.** Read the documentation template. It defines how many files each service should have. No hardcoded number — the template is the source of truth about structure.

**Step 0.3 — Business questions.** Formulate questions that grow from the README text — not from a checklist. Each question = a test case. No answer = red. Contradictory answer = red.

---

## Phase 1 — Generate Tests from Documentation

Tests are born from the documentation itself, not from external requirements.

For each file, three questions:

```
CLAIM: what does this file assert?
  (facts, step order, dependencies, roles)

ECHO: who else should confirm this assertion?
  (A says "we call B" → B must have "A calls us")

GAP: what SHOULD this file describe but doesn't?
  (by the file's role: entities.md should have all fields from api-contracts.md)
```

Five types of tests:

| Type | What it checks |
|------|---------------|
| Presence | Does file X contain description of Y? |
| Consistency | Does assertion in X match assertion in Y? |
| Completeness | Is concept from A also described in B, C, D? |
| Flow | Is step order identical across all files? |
| Contradiction | Is there a file that asserts the opposite? |

---

## Phase 2 — Traverse the Corpus

### Reading order

```
Round 1: root README → system map + main flow + service list
Round 2: services in main message flow (in flow order) → upstream/downstream awareness
Round 3: cross-cutting files (system/, entities, api-contracts) → consistency
Round 4: all remaining services → orphans, contradictions, gaps
```

### Infrastructure layer — mandatory

Infrastructure services (audit, monitoring, logging) never appear in the main flow by link — agents organically skip them. **They must be read explicitly after the main pass:**

```
audit-service/    → rollback mechanism on failure (not just logs)
monitoring-system/ → system self-awareness, self-recovery
log-aggregator/   → operational memory for diagnostics
```

### Coverage Checkpoint

After the main pass:

```
glob("docs/*/README.md") → full list of services
Compare with what was read → who was missed?
```

An unread service = explicit mark in the report. False completeness is worse than an honest gap.

### File contracts by role

| File | Must contain | Who confirms |
|------|-------------|-------------|
| `README.md` | purpose, place in system, callers, callees | READMEs of related services |
| `service-logic.md` | business rules, algorithm, step order | README.md of same service |
| `entities.md` | all entities, fields, types | api-contracts.md |
| `api-contracts.md` | all endpoints/topics, request+response | integrations.md of callers |
| `dataflow.md` | all topics, producers and consumers | system/kafka-topics.md |
| `errors.md` | all error codes, meaning, handling | errors.md of callers |
| `integrations.md` | who it calls, what it expects, what it does with result | integrations.md of callees |
| `rbac.md` | roles, permissions | system/rbac-global.md |

---

## Phase 3 — Narrative Coherence

Documentation must tell one story in different words from different perspectives.

### Violation patterns

**Blind spot** — A describes interaction with B in detail. B doesn't mention A at all.

**Telephone game** — README: "X does P through Q". service-logic: "X does P directly". Somewhere in the chain, the meaning changed.

**Orphan** — File Z describes important concept C. C is not mentioned anywhere else. Either C is unnecessary, or nobody knows about it.

**Split personality** — In file A, term T means X. In file B, term T means Y.

**Flow abyss** — A passes data D to B. B doesn't describe how it receives D or what it does with it.

**Asymmetric awareness** — A knows B exists. B doesn't know A uses it. Normal for external libraries. Not normal for internal services.

---

## Phase 4 — Severity Levels

| Level | What it means | Example |
|-------|-------------|---------|
| **CRITICAL** | Documentation contradicts itself | A says "X calls Y", B says "Y calls X" |
| **HIGH** | Important info exists in one place, absent where it must also be | api-contracts describes `user_id`, entities doesn't contain it |
| **MEDIUM** | Coherence break, not contradiction | A mentions B, B doesn't know about A |
| **LOW** | Terminology inconsistency | "intent" / "intention" for one concept |

### Finding format

```
[SEVERITY] Violation type

FILES:
 - path/to/file-a.md: "[quote]"
 - path/to/file-b.md: "[quote]"

VIOLATION: [what exactly is wrong]
EXPECTED: [what should be]
FIX: [concrete text or direction of edit]
```

---

## Phase 5 — Fix Principles

1. **Minimal intervention** — fix only what's broken.
2. **Preserve voice** — don't change tone, only eliminate inconsistency.
3. **Add, don't delete** — if info exists in A but not B — add to B.
4. **Reference = more detailed file** — service-logic.md is more detailed than README.md.
5. **Log every change** — each edit in the log.

---

## Iatrogenia — What to Avoid

Iatrogenia: treating one disease, causing another.

```
❌ "exactly 15 files" → crushes context, collapses information
❌ "top 5 problems" → agent picks 5, silences the rest
❌ "describe briefly" → nuances lost
❌ rigid output format → agent bends reality to fit format

✅ "files by template" → template = source of truth about structure
✅ "all findings" → no quantity limit
✅ "write as much as needed" → volume determined by content
```

---

## Definition of "Documentation Passes E2E"

1. All business questions have unambiguous answers in the right files
2. Answers are consistent — the same thing is described identically everywhere
3. The relationship graph is symmetric — every A→B is described from both sides
4. The flow is traceable from entry to exit
5. Terminology is unified — one term, one meaning
6. No orphans — every document answers someone's question
7. Autonomy is described — the cycle of detect→localize→rollback→recover is documented
