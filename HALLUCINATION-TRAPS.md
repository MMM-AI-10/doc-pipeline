# Hallucination Traps in Documentation

> Methodological module of the pipeline.
> Applied by all agents when reading every file.
> Task: find places where an LLM will **guaranteed** start hallucinating.

A hallucination is not an LLM bug. It's a predictable response to uncertainty in text.
Task: find all uncertainty points **before** they cause an error.

---

## Checklist — run on every file

```
[ ] TRAP-01: no floating pronouns (it/this/the system) when 2+ subjects exist
[ ] TRAP-02: no "etc." without full enumeration or link
[ ] TRAP-03: no "may"/"sometimes" without describing when exactly
[ ] TRAP-04: no "see below"/"as described above" without exact file#anchor
[ ] TRAP-05: section heading doesn't contradict its body
[ ] TRAP-06: all numbers have units and context
[ ] TRAP-07: examples are explained; abstract descriptions are illustrated
[ ] TRAP-08: exceptions described after the normal case, not instead of it
[ ] TRAP-09: no passive voice without explicit actor
[ ] TRAP-10: architectural and implementation levels not mixed in one sentence
[ ] TRAP-11: no contradictions between files without [DEPRECATED] marker
[ ] TRAP-12: no circular definitions (A through B, B through A)
[ ] TRAP-13: COLLAPSE — see COLLAPSE-PROTOCOL.md

── Layer pyramid traps ──────────────────────────────────────────────
[ ] TRAP-14: information in the correct file for its layer (business logic not in README)
[ ] TRAP-15: every downward link leads to a non-empty file (no links to stubs)
[ ] TRAP-16: no template stubs (___) accepted as real content

── Confident-fill traps ─────────────────────────────────────────────
[ ] TRAP-17: no confabulation — every fact is cited from a specific file
[ ] TRAP-18: every silent collapse is marked with a marker, even without details
```

---

## Trap Catalog

### TRAP-01 · Floating Pronoun

**Symptom:** "it/this/the system/the service/the component" without explicit antecedent when multiple subjects exist.

```
❌ "Orchestrator receives the message. It passes it for processing.
   After that, the system checks permissions."
  → Who is "it"? Which "system"? LLM picks by its own weights.

✅ "Orchestrator receives the message. Orchestrator passes it to Policy Engine.
   Policy Engine checks user permissions."
```

**Regex signal:**
```
\b(it|this|the system|the service|the component)\s+(calls|sends|receives|checks|returns|passes)
```

**Danger:** 🔴 CRITICAL

---

### TRAP-02 · Incomplete Enumeration

**Symptom:** "and others", "etc." without a full list or link.

```
❌ "Policy Engine checks roles, permissions, context and other parameters."
  → Which "others"? LLM invents from its weights.

✅ "Policy Engine checks: role, allowed_intents, tenant_id.
   Full list: see policy-engine/entities.md#CheckRequest"
```

**Danger:** 🟠 HIGH

---

### TRAP-03 · Indefinite Modal Verb

**Symptom:** "may", "sometimes", "in some cases" without describing conditions.

```
❌ "Orchestrator may call the RAG system."
  → When exactly? LLM adds the call in one place, skips it in another.

✅ "Orchestrator calls RAG only if intent_type == 'knowledge_query'."
```

**Regex signal:**
```
\b(may|sometimes|in some cases|if needed|as a rule|usually)\b
```
For each match: is there a "when exactly" condition in the same paragraph?

**Danger:** 🟠 HIGH

---

### TRAP-04 · Empty Reference

**Symptom:** "see below", "described above", "see the documentation" without a concrete path.

```
❌ "Request format is described in the API documentation."
  → LLM fills from general knowledge, not from your system.

✅ "Request format: see policy-engine/api-contracts.md#POST-/check"
```

**Regex signal:**
```
(see below|described above|see documentation|in the relevant|as described previously)
```

**Danger:** 🟠 HIGH

---

### TRAP-05 · Heading Contradicts Body

**Symptom:** section heading asserts X, but text inside describes Y.

```
❌ ## Synchronous Policy Engine Call
   Orchestrator sends an event to Kafka topic policy.check...
   → Heading says "synchronous", body describes async Kafka pattern.
   → LLM takes the heading (it has more weight in transformers).

✅ ## Asynchronous Policy Engine Call via Kafka
```

**Danger:** 🔴 CRITICAL

---

### TRAP-06 · Number Without Unit

**Symptom:** numbers, timeouts, limits without measurement units.

```
❌ "Timeout: 30" / "Maximum: 100" / "Retry: 3"
  → 30 what? 100 what? LLM substitutes the most probable unit.

✅ "Timeout: 30 seconds" / "Maximum: 100 requests/sec per tenant" / "Retry: 3 attempts"
```

**Regex signal:**
```
(timeout|limit|max|retry):\s*\d+(?!\s*(sec|ms|min|s|rpm|rps|bytes|kb|mb))
```

**Danger:** 🟡 MEDIUM

---

### TRAP-07 · Example Without Explanation / Explanation Without Example

**Symptom:** JSON block without description of what it is; or abstract description without a single example.

```
❌ ```json
   { "user_id": "usr_123", "context": { "tenant": "acme" } }
   ```
   (and nothing else — what is this? request? response? for what?)

✅ "Request to Context Manager (POST /context/get):
   ```json
   { "session_id": "sess_abc", "keys": ["last_intent"] }
   ```
   Returns: { \"last_intent\": \"crm_search\" }"
```

**Danger:** 🟡 MEDIUM

---

### TRAP-08 · Exception Without Rule

**Symptom:** document describes an edge case, but doesn't describe the normal case.

```
❌ "If Orchestrator is unavailable, the Bot sends directly."
  → What about the normal case? LLM knows only the exception and applies it as rule.

✅ "Normal flow: Bot → Orchestrator → ... → response.
   Exception (Orchestrator unavailable): escalation directly via Kafka: escalation.created"
```

**Danger:** 🟡 MEDIUM

---

### TRAP-09 · Passive Voice Without Actor

**Symptom:** something happens, but it's unclear who does it.

```
❌ "The message is normalized and passed for processing."
   "Permissions are checked before the action is executed."
  → Who normalizes? Who checks?

✅ "The Bot normalizes the message and passes it to Orchestrator."
   "Policy Engine checks permissions before Orchestrator builds a plan."
```

**Regex signal:**
```
(is normalized|is checked|is updated|is passed|is executed|is formed|is processed)
(?!\s*(through|using|by|via))
```

**Danger:** 🟠 HIGH

---

### TRAP-10 · Mixed Abstraction

**Symptom:** one sentence mixes architectural level with implementation level.

```
❌ "Policy Engine is the central permission authority, which does
   SELECT * FROM permissions WHERE user_id = $1 and returns allowed_intents."
  → LLM can't tell: is this architecture? contract? implementation?

✅ "Policy Engine is the central permission authority. Returns allowed_intents for user_id.
   Internal implementation: see policy-engine/databases.md"
```

**Danger:** 🟢 LOW

---

### TRAP-11 · Stale Artifact Without Marker

**Symptom:** description contradicts other files, but isn't marked [DEPRECATED].

```
❌ File A (old): "Intent Recognition is called by user directly"
   File B (new): "Intent Recognition is called only by Orchestrator"
  → Both files are valid. LLM averages them — gives wrong answer with 50/50 confidence.

✅ File A: [DEPRECATED — see intent-recognition/README.md]
```

**Danger:** 🔴 CRITICAL

---

### TRAP-12 · Circular Definition

**Symptom:** A is defined through B, B is defined through A.

```
❌ "Intent — user's intention, recognized through dialog context"
   "Dialog context — state formed based on user intents"
  → LLM cannot build understanding of either term.
```

**Danger:** 🟡 MEDIUM

---

### TRAP-14 · Information Not on Its Layer

**Symptom:** business logic, technical details, or step-by-step scenarios written in README instead of the proper file. Agent reads README, thinks it knows everything, moves on. Details in actual files are absent — agent doesn't know.

```
Correct pyramid (proper distribution):

README.md          → paragraph + link. Navigation and one line per aspect only.
business-logic.md  → why, for whom, by what rules. No tech stack.
service-logic.md   → components, architecture, interaction table.
flows.md           → step-by-step scenarios with payloads.
specs              → exact fields, types, topics, tables.
```

```
❌ README.md contains:
   "## Business Rules
   If a user is inactive for 30 days — account is frozen.
   Freeze is lifted only through support or after 90 days.
   On freeze, all active sessions are invalidated immediately."
  → Agent read README and thinks it knows the business logic.
  → business-logic.md is empty. Nobody goes there.

✅ README.md:
   "Account lifecycle — freeze, recovery, deletion rules.
   Details → [business-logic.md#account-lifecycle](./business-logic.md#lifecycle)"

   business-logic.md contains the full rule description.
```

**How to detect:**
```
1. README longer than 150 lines → suspicion
2. README has H2/H3 headings with business terms (not "Navigation", not "Links") → TRAP-14
3. business-logic.md is empty or contains only stubs while README is rich → TRAP-14
4. README contains tables with fields/data types → should be in entities.md
5. README has steps "1. → 2. → 3." → should be in flows.md
```

**Risk:** agent builds a model of the system from README and doesn't know that real business logic is undocumented anywhere. When generating code — invents rules from general knowledge.

**Danger:** 🔴 CRITICAL

---

### TRAP-15 · Link to Empty File

**Symptom:** file A links to file B for details, but file B is empty (contains only template). Agent reads the link, interprets as "details exist", moves on without reading B. Or reads B, gets a template — and still considers it "documentation".

```
❌ README.md:
   "Authorization rules → [rbac.md](./rbac.md)"

   rbac.md contains:
   "| Role | Description |
   |------|-------------|
   | `___` | `___` |"
  → Link exists. Content doesn't. Agent won't notice the difference.

✅ If file is empty — link is explicitly marked:
   "Authorization rules → [rbac.md](./rbac.md) ⚠️ in progress"

   Or link doesn't exist until file is filled.
```

**How to detect:**
```
For every link [→ file.md] in README and other files:
  Read the target file.
  If file contains only lines with "___" or "*(repeat block)*" → TRAP-15.
  If file consists only of headings without content → TRAP-15.
  If file is shorter than 20 lines for a non-trivial service → suspicion.
```

**Risk:** agent is confident details are documented — they're "at the link". When answering questions, fills gaps from general knowledge. Unnoticeably.

**Danger:** 🔴 CRITICAL

---

### TRAP-16 · Template Stub Accepted as Fact

**Symptom:** file contains template placeholders (`___`, `[description]`, `*(repeat block)*`, `{service-name}`) that were never replaced with real content. Agent reads them as real content or silently skips — both are dangerous.

```
❌ entities.md:
   "| `___` | `string` | ✅ | ❌ | `min:1, max:255` | | |"
  → Agent sees a table row. Treats as "field exists".
  → When generating code, creates a field named "___".

✅ Unfilled files should contain an explicit marker:
   "<!-- TODO: not filled -->" or not exist at all.
```

**How to detect:**
```
grep -r "^\| \`___\`\|^- \`___\`\|^\`___\`\|{service-name}" docs/**/*.md
```
Every match = TRAP-16. File contains template instead of data.

**Risk:** agent builds a wrong model ("fields exist, I saw them") or builds no model at all ("no invariants") — both lead to errors in code.

**Danger:** 🟠 HIGH

---

### TRAP-17 · Confabulation — Fact Without Source

**Symptom:** agent asserts something concrete about the system without quoting a file. Not "documentation doesn't describe X", but confident "X works like this" — from its head.

Confabulation differs from hallucination in that it sounds plausible and is consistent with the read context. You can't catch it without checking the source.

```
❌ Agent: "Payment Service uses idempotency via Redis with 24-hour TTL."
  → Plausible. Written nowhere. Agent filled from typical patterns.

❌ Agent: "When Warehouse Service crashes, the order moves to PENDING status."
  → Logical. But flows.md doesn't describe this scenario. Agent made it up.

✅ Agent: "Payment Service uses idempotency via Redis.
   [payment-service/service-logic.md#caching — TTL not specified, needs clarification]"
```

**Rule for agents:**
```
Every concrete fact about the system must have a source:
  FACT [file:section]

If fact not found in documents — the only acceptable conclusion:
  "Not documented in [expected file]."

Forbidden: guessing typical behavior, applying general patterns without quote,
 filling gaps "from common sense".
```

**How to detect in another agent's output:**
```
For every assertion about the system in the agent's response:
  Is there a link to a specific file?
  If no → potential confabulation.
  Check: did the agent read the corresponding file?
  If file is empty or doesn't exist → confabulation confirmed.
```

**Danger:** 🔴 CRITICAL

---

### TRAP-18 · Silent Collapse Without Marker

**Symptom:** agent encountered two descriptions of the same thing and chose one — without a [COLLAPSE] marker. No trace of the choice in the response or REPORT.md. Human doesn't know a choice was made.

This differs from TRAP-13 (which is about the collapse itself). TRAP-18 is about when a collapse happened but **was not marked**.

```
❌ Agent read:
   README.md: "Orchestrator calls Policy Engine synchronously"
   service-logic.md: "Policy Engine is notified via Kafka"

   Agent's response: "Orchestrator calls Policy Engine synchronously."
  → Choice made. No marker. Alternative lost.

✅ Agent's response: "Orchestrator calls Policy Engine synchronously.
   [COLLAPSE:RED]
   CHOSEN: synchronously — README.md
   ALTERNATIVE: via Kafka — service-logic.md
   REASON: README as entry point, but contradiction requires resolution"
```

**Special case — collapse without details:**
When an agent sees a discrepancy but can't pinpoint the exact source — a minimal marker without full analysis is acceptable:

```
[COLLAPSE:UNRESOLVED]
Discrepancy noticed in: [description of contradiction]
Sources not identified — requires human review.
```

This is better than silence. The fact that a discrepancy was noticed is more important than the details.

**Rule:**
```
Any choice between two versions of one fact = marker.
No matter how obvious the choice is.
No matter that "one version is clearly more correct".
If there was a choice — marker is mandatory.
A minimal marker is better than no marker.
```

**Danger:** 🔴 CRITICAL

---

## Priority by Danger

| Trap | Danger | Reason |
|------|--------|--------|
| TRAP-05 Heading vs body | 🔴 | Headings carry more weight in transformers |
| TRAP-11 Stale artifact | 🔴 | LLM doesn't see that a file is outdated |
| TRAP-01 Floating pronoun | 🔴 | With 2+ services — hallucination guaranteed |
| TRAP-13 Collapse | 🔴 | see COLLAPSE-PROTOCOL.md |
| TRAP-14 Info on wrong layer | 🔴 | Agent builds incomplete model with 100% confidence |
| TRAP-15 Link to empty file | 🔴 | Agent is confident details exist — they don't |
| TRAP-17 Confabulation | 🔴 | Sounds plausible, indistinguishable from fact without verification |
| TRAP-18 Silent collapse | 🔴 | Choice made, no trace left, human doesn't know |
| TRAP-02 Incomplete enumeration | 🟠 | LLM fills the gap from its own knowledge |
| TRAP-03 Indefinite modal | 🟠 | LLM adds or skips behavior inconsistently |
| TRAP-04 Empty reference | 🟠 | LLM fills from general knowledge |
| TRAP-09 Passive without actor | 🟠 | Agent can't determine who acts |
| TRAP-16 Stub as fact | 🟠 | Agent treats template as documented data |
| TRAP-06 Number without unit | 🟡 | LLM substitutes most probable unit |
| TRAP-07 Example unexplained | 🟡 | LLM can't determine what example illustrates |
| TRAP-08 Exception without rule | 🟡 | LLM applies exception as rule |
| TRAP-12 Circular definition | 🟡 | LLM can't build understanding of either term |
| TRAP-10 Mixed abstraction | 🟢 | LLM confused about which level applies |
