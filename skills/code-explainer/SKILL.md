---
name: code-explainer
description: >-
  Explain a file, module, or subsystem from first principles with multiple
  concrete examples, then show how it fits in the wider codebase. Use when the
  user says code-explainer, explain this module, teach me this code, break this
  down from first principles, or wants examples of how a piece of the system
  works end-to-end.
disable-model-invocation: true
argument-hint: "path, symbol, or topic to explain"
---

# Code Explainer

Teach the user a piece of code so they could re-derive it. Prefer **first
principles + examples** over tour-guide file summaries.

Works with any Agent Skills–compatible tool (Claude Code, Codex, Cursor, etc.).

## Hard rules

1. **First principles first.** Start from the problem the code exists to solve,
   then the minimal concepts required, then the actual implementation.
2. **Every non-trivial claim needs an example.** Prefer concrete inputs,
   outputs, call sequences, and failure cases over abstract adjectives.
3. Use **multiple examples** (at least 2–3) for the core behavior: one normal,
   one edge/failure, one that shows an important branch or interaction.
4. Always explain **how this module fits the whole codebase** (callers, callees,
   data/control flow, boundaries).
5. Break complexity down: big idea → parts → how parts compose → pitfalls.
6. Do not dump huge code blocks. Quote only the smallest snippets that carry
   the idea; narrate the rest.
7. Match the user’s level: if they are junior, define terms once with an
   example before using them freely.
8. If the target is unclear, ask **one** clarifying question (which path /
   symbol / behavior), then proceed.

## Workflow

```
Code explainer:
- [ ] 1. Identify target (file / module / symbol / behavior)
- [ ] 2. Skim neighbors + entrypoints (fit in codebase)
- [ ] 3. Read AGENTS.md once if present (boundaries/conventions)
- [ ] 4. Explain from first principles with examples
- [ ] 5. Walk the real code in small steps + examples
- [ ] 6. Show system fit (map + example request/job path)
- [ ] 7. Pitfalls + “check your understanding” prompts
```

### 1. Identify the target

Resolve what to explain:

- Path(s) the user named, or
- Symbol (class/function), or
- Behavior (“how dialing works”)

State in one line: **Target** + **Job it does**.

### 2. Gather just enough context

Read:

- The target module
- Direct callers and key callees
- Relevant types/schemas/tests that show intended use
- Repo `AGENTS.md` **once** (if it exists) for architecture boundaries that
  affect how you explain placement

Do not read the entire repo. Stop when you can tell a coherent story.

## Explanation structure (required)

Use this shape unless the user asks for a different cut:

### A. The problem (first principles)

- What would go wrong in the product/system without this module?
- What single responsibility does it own?
- One tiny **example** of the world before/after this module exists.

### B. Core concepts

List the 2–5 ideas you must understand (state machine, queue, idempotency,
adapter boundary, etc.). For **each** concept:

- Definition in plain language
- **Example** (concrete)

### C. How it works (mechanics)

Walk the main algorithm/flow in ordered steps. After every important step,
give a mini-example (“given X, this line does Y, resulting in Z”).

Include at least:

1. **Happy-path example** (full walkthrough)
2. **Edge/failure example** (retry, empty input, unauthorized, timeout, …)
3. **Second distinctive example** (alternate branch, second caller, batch vs
   single, etc.)

### D. Fit in the codebase

Explain placement with a small map:

- Who calls this? (entrypoints)
- What does it call? (dependencies)
- Which layer/boundary is it on? (API / domain / infra / worker / UI)
- What must *not* depend on it (if `AGENTS.md` or structure says so)?

Then one **end-to-end example**: a user action or job that enters the system
elsewhere and passes through this module (name real files/symbols).

### E. Invariants and pitfalls

- What must always remain true?
- Ways agent-generated callers often misuse it
- Failure modes with a short example each

### F. Check understanding

Ask 2–3 short questions the user can answer in chat (no need for a formal
quiz UI). Prefer questions that require applying an example, not memorizing
names.

## Example density (non-negotiable)

Bad: “This service handles idempotency for dials.”

Good: “If two workers both try to start a dial for `+1555…` at the same time,
only the first `try_start` returns true; the second gets false — example
trace: …”

If you catch yourself writing two abstract sentences in a row, add an example.

## Depth knobs

| User says | Do this |
|-----------|---------|
| default | One module / tight symbol cluster; ~structure above |
| `deeper` | Expand callees one level; more examples |
| `eli5` | More analogy + even smaller steps; still use real names |
| `just the flow` | Favor D + C happy path; still keep ≥2 examples |

## Coordination

- Whole system / architecture foundation → use **`codebase-explainer`**
- Single module deep-dive → stay on **code-explainer**
- Mid-tour zoom from a codebase chapter → `code-explainer` on one path, then return

## Anti-patterns

- File-by-file narration with no first-principles problem statement
- Explaining without examples
- One giant pasted file with “see above”
- Ignoring how the module connects to the rest of the system
- Hand-waving failures (“errors are handled”) with no example
- Teaching the whole codebase when they asked about one module
