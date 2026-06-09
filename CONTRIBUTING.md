# Contributing to doc-pipeline

First: thank you. This project exists because documentation for LLMs is a real, unsolved problem, and every contribution matters.

---

## What We're Looking For

- **New hallucination traps** — found a pattern where LLMs consistently hallucinate on docs? Add it to `HALLUCINATION-TRAPS.md`
- **New collapse patterns** — discovered a way documentation superposition manifests? Add it to `COLLAPSE-PROTOCOL.md`
- **Agent improvements** — better prompts, new analysis phases, more precise finding formats
- **Templates** — new document types that make LLM consumption more deterministic
- **Examples** — real-world patterns (anonymized) that illustrate traps or good practices

---

## How to Contribute

### 1. Open an Issue First

Before writing code or docs, open an issue describing what you want to add or change. This saves everyone time if the direction doesn't align with the project's goals.

### 2. Keep the Principles

Every contribution should align with the four system principles:

1. **Documentation is code** — structured, versioned, reviewed
2. **Determinism over completeness** — less text that's unambiguous beats more text that's vague
3. **Layer pyramid** — each file has a role; information lives in the right layer
4. **Collapse visibility** — when uncertainty exists, mark it; never silently resolve it

### 3. Follow the Style

- Write for practitioners, not academics
- Use concrete examples, not abstractions
- Keep language direct — no hedging, no filler
- Templates use `___` for fill-me-later fields
- Agent prompts follow the structure in `agents/AGENT.md`

### 4. Language

The repository is in English. If you're contributing examples from non-English projects, translate them. The target audience is international.

### 5. No Leaks

- Do not include proprietary information, company names, or internal project references in examples
- Use neutral service names: `order-service`, `payment-service`, `notification-service`, `user-service`, etc.
- If adapting a real-world pattern, anonymize thoroughly

---

## File-Specific Guidelines

### Adding a Hallucination Trap

Add to `HALLUCINATION-TRAPS.md`:
1. Number it sequentially (TRAP-19, TRAP-20, ...)
2. Include: Symptom, Bad example, Good example, Regex signal (if applicable)
3. Add to the checklist at the top

### Adding a Collapse Level

Add to `COLLAPSE-PROTOCOL.md`:
1. Define the level name and color
2. Give examples
3. Specify the action and marker format
4. Add to the priority order in the Summary section

### Adding an Agent

1. Create `agents/[AgentName].md`
2. Follow the structure from existing agents:
   - Core Identity
   - Start Sequence
   - Analysis Phases
   - Finding Types (if applicable)
   - Output Format
   - Constraints
3. Add the agent to `agents/AGENT.md` pipeline diagram

### Adding a Template

1. Create `templates/[name]-template.md`
2. Use `___` for fields to fill
3. Include GOOD example comments (see existing templates)
4. Cross-reference other templates where links would go

---

## Pull Request Process

1. One concern per PR — don't mix trap additions with template changes
2. Update the README if you're adding a new section or file
3. If changing an agent prompt, explain what scenario the change improves
4. Examples must be runnable/followable without external context

---

## Questions?

Open an issue with the label `question`. We respond.

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
