# Collapse Protocol — Superposition Collapse Markup

> Methodological module of the pipeline (TRAP-13).
> When an LLM encounters two different descriptions of the same thing — it doesn't stop.
> It collapses: chooses one and continues with 100% confidence. Without a trace.
> This protocol makes the collapse visible.

---

## What Happens During Collapse

```
Documentation:
  File A: "Orchestrator calls Intent Recognition AFTER Policy Engine"
  File B: "Intent Recognition is called BEFORE permission check"

LLM during assembly:
  [silently chooses File B as more detailed]
  Generates: "Intent Recognition → Policy Engine → Orchestrator"
  Confidence: 100%
  Trace of choice: none
```

**Why this is worse than regular hallucination:**
A hallucination doesn't align with anything — you can catch it.
A collapse aligns with one of the real versions — almost impossible to notice.
When you check "is this in the documentation?" — the answer is YES. Just the wrong version.

**Multiplication pattern:**
Collapsed at point A → all subsequent conclusions build on the collapsed version →
documentation is internally consistent but contradicts reality.
Finding the source without markers is nearly impossible.

---

## Conditions for Collapse

Collapse occurs when **all are true simultaneously:**
```
1. Documentation has 2+ descriptions of the same concept/flow/term
2. Descriptions are semantically close but not identical
3. No explicit marker that one of them is outdated
4. LLM processes them in different chunks
```
At 100+ files this isn't rare — it's a statistical inevitability.

---

## Collapse Levels

### LAYER — Layer collapse

```
Specific to systems with a layer pyramid (README → business-logic → service-logic → specs).

Occurs when:
- README and business-logic describe the same thing (business-logic not filled,
  everything written in README — TRAP-14)
- Agent reads both files, sees no difference, collapses into one description
- Lost understanding: README = navigation + brevity, business-logic = completeness

Examples:
  README: "Service checks user permissions before execution."
  business-logic.md: "Service checks user permissions before execution." (copied)
  → Agent doesn't know business-logic should be deeper. Both versions are identical.

  README: "Three order states: NEW → PROCESSING → DONE"
  service-logic.md: empty
  → Agent knows three states. Doesn't know transition rules. Will invent.

Action: continue, mark. Add to REPORT.md as TRAP-14 + COLLAPSE:LAYER.
Marker: [COLLAPSE:LAYER ...]
```

### RED — changes architectural decision

```
Examples: call order, who initiates, sync vs async, data owner

Action: STOP. Emit marker. Do not continue until resolved.
Marker: [COLLAPSE:RED ...]
```

### YELLOW — changes implementation details

```
Examples: different names for same field, different parameter sets, different conditions

Action: continue, explicitly mark the choice.
Marker: [COLLAPSE:YELLOW ...]
```

### GRAY — terminology discrepancy

```
Examples: different words for one concept, different enumeration order

Action: continue, note for glossary.
Marker: [COLLAPSE:GRAY ...]
```

### UNRESOLVED — noticed but source unclear

```
When agent sees a discrepancy but can't pinpoint the exact source —
a minimal marker without full analysis is acceptable:

[COLLAPSE:UNRESOLVED]
Discrepancy noticed: two files describe service behavior differently.
Sources not identified — requires human review.

Minimal marker is better than silence.
The fact that a discrepancy was noticed is more important than the details.
```

---

## Marker Format

```
[COLLAPSE:LEVEL]
CHOSEN: [what was chosen] — source: [file:line]
ALTERNATIVE: [what was discarded] — source: [file:line]
REASON: [why this source and not that one]
```

### Examples

**When answering a question about a flow:**
```
"Order: Orchestrator → Policy Engine → Intent Recognition

[COLLAPSE:RED]
CHOSEN: Policy Engine first — orchestrator/service-logic.md:12
ALTERNATIVE: Intent Recognition first — intent-recognition/README.md:8
REASON: service-logic.md is more detailed, explicitly describes steps"
```

**When summarizing:**
```
"Policy Engine returns allowed_intents.

[COLLAPSE:YELLOW]
CHOSEN: { allowed_intents: string[] } — policy-engine/api-contracts.md
ALTERNATIVE: { permissions: string[], context: object } — policy-engine/entities.md
REASON: api-contracts as public contract takes priority over entities"
```

**When layer collapse:**
```
"Service checks user permissions. [taken from README]

[COLLAPSE:LAYER]
CHOSEN: README.md — brief mention
ALTERNATIVE: business-logic.md — empty (only template)
REASON: business-logic.md not filled, README is the only source
RISK: permission check rules not documented at business-logic level —
 agent builds incomplete model"
```

**When silent collapse without details (minimal marker):**
```
[COLLAPSE:UNRESOLVED]
Discrepancy noticed: two files describe service behavior differently.
Sources not identified — requires human review.
```

---

## When to Emit a Marker

```
YES — read 2+ files, they describe the same thing differently
YES — choosing the more "plausible" of two valid sources
YES — current conclusion contradicts something read earlier
YES — using heuristic "this file is more authoritative" without explicit documentation

NO — contradiction already marked [DEPRECATED] in documentation
NO — different files describe different aspects (not competing versions of the same thing)
NO — difference only in detail level without substantive discrepancy
```

**Formulation:** specific files and lines. Explain the choice.
Don't judge: "file X and file Y diverge" — not "file X is wrong".

---

## Detecting Potential Collapse Points

### Method 1 — Duplicate flow descriptions

One and the same service in different positions in different files = collapse point.

### Method 2 — Duplicate definitions

Collect all "[Term] is..." statements. Group by term.
Group > 1 with non-identical definitions = collapse point.

### Method 3 — One relationship described from both sides

For relationship A → B:
- Description from A's side (integrations.md, service-logic.md)
- Description from B's side (api-contracts.md, README.md)
Order, conditions, or format diverge = collapse point.

### Signal phrases

```
(similarly|same as|as described in|in a similar way|by the same scheme|analogous mechanism)
```
Each = candidate for checking: consistent mirror or drifting copy?

---

## Source Priority by Layer Pyramid

When choosing between two versions of the same fact — use this hierarchy:

```
Highest priority (most precise):
  specs: api-contracts.md · entities.md · dataflow.md · databases.md · rbac.md
  ↓
  service-logic.md — technical structure
  ↓
  business-logic.md — business meaning
  ↓
  flows.md — scenarios (may simplify)
  ↓
  README.md — navigation and brevity (least detailed source)

Rule: a spec is always more precise than a description.
  service-logic is more precise than README.
  If README contradicts service-logic — README is outdated or superficially written.
```

**Exception:** if a spec is explicitly marked `[DRAFT]` or `[TODO]` —
it's less authoritative than a descriptive file.

**With COLLAPSE:LAYER** — a source from a deeper layer takes priority,
but if the deep layer is empty — that's TRAP-15 (link to empty file),
not grounds to ignore README.

---

## How to Fix by Markers

```
[COLLAPSE:LAYER] markers:
  1. Check: is business-logic.md or service-logic.md empty?
  2. If yes → move content from README to the proper file
  3. After move: README keeps paragraph + link

[COLLAPSE:RED] markers:
  1. Read both sources (CHOSEN and ALTERNATIVE)
  2. Decide: which description is CORRECT?
  3. Make the correct one canonical — add explicit "this is the reference" marker
  4. In the alternative: replace with link to reference or mark [DEPRECATED]

[COLLAPSE:YELLOW] markers:
  Unify field names using entities.md as source of truth

[COLLAPSE:GRAY] markers:
  Candidates for glossary

[COLLAPSE:UNRESOLVED] markers:
  Require manual human review before the agent continues.
  Do not ignore — this is a signal that the model is not confident in its choice.
```

---

## Summary

```
Without the protocol:
  agent reads files → collapses → confident output
  human doesn't know where collapse occurred → errors multiply silently

With the protocol:
  agent collapses → MARKS (even minimally) → human sees map of uncertainty points
  every RED = concrete place where documentation needs to be made singular
  every LAYER = place where pyramid is broken
  every UNRESOLVED = honest signal "I'm not sure, check this"
  after fixing: one question → one answer regardless of reading order
```

**Levels in priority order of response:**
```
COLLAPSE:RED       → STOP, do not continue
COLLAPSE:LAYER     → continue, mark, add to fix plan
COLLAPSE:YELLOW    → continue, mark
COLLAPSE:GRAY      → continue, add to glossary
COLLAPSE:UNRESOLVED → continue, mark, requires human review
```

**Goal: deterministic documentation.**
Any agent, reading files in any order, arrives at one understanding of the system.
