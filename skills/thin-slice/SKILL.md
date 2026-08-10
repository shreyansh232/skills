---
name: thin-slice
description: >-
  Before a big feature, design and fully ship the smallest end-to-end working
  slice first, then use that slice as the reference for a larger implementation
  plan. Use when the user says thin-slice, smallest working feature, walking
  skeleton, tracer bullet, spike the feature, or wants to prove a path before
  planning the full build.
disable-model-invocation: true
argument-hint: "feature description (optional)"
---

# Thin Slice

Big features fail when you plan the cathedral before you’ve walked one hallway.
This skill forces a **smallest fully working vertical slice** first, then builds
the implementation plan **from that proven reference** — not from imagination.

Works with any Agent Skills–compatible tool (Claude Code, Codex, Cursor, etc.).

## Hard rules

1. **Slice before plan.** Do not write a full multi-phase implementation plan
   until the thin slice runs end-to-end in this repo (or the user explicitly
   accepts a documented blocker).
2. The slice must be **vertical**: real entry → real logic → real side effect or
   response — not a mock-only diagram and not “all the models with no API.”
3. **Smallest** means cut scope aggressively: one happy path, one actor, one
   data path. Defer variants, polish, and admin surfaces.
4. “Working” means you can **demonstrate** it (command, request, UI click, or
   test) and state exact steps to reproduce.
5. Read `AGENTS.md` **once** and respect boundaries while choosing the slice.
6. Prefer consistency with existing codebase patterns (same layers, helpers,
   test style).
7. After the slice works, the plan must **cite the slice** (files, APIs, tests)
   as the template for remaining work.

## When to use

- Starting a large feature or refactor
- Unclear design with multiple possible shapes
- Agent is about to generate a huge scaffold
- You want a concrete reference before estimating or sequencing work

Not for: typo fixes, tiny bugs, or a feature that *is* already one small slice
(just build it; use `push-ready build` if you want iterative review).

## Workflow

```
Thin slice:
- [ ] 1. Capture the big-feature goal
- [ ] 2. Read AGENTS.md once + scout existing nearby patterns
- [ ] 3. Propose 2–3 candidate thin slices; pick ONE with the user
- [ ] 4. Define done for the slice (repro steps)
- [ ] 5. Implement only the slice (optionally with push-ready build)
- [ ] 6. Prove it works (run/repro); fix until green
- [ ] 7. Write the larger implementation plan using the slice as reference
```

### 1. Big-feature goal

In 3–6 bullets:

- User/system problem
- Full success criteria (the eventual feature)
- Non-goals
- Constraints (time, compatibility, `AGENTS.md` boundaries)

### 2. Scout

- Existing modules that will likely be touched
- Similar features to copy structurally
- Test and run commands from project docs / `AGENTS.md`

### 3. Choose the thin slice

Propose candidates scored by:

| Candidate | Proves what? | Touches which layers? | Size risk |
|-----------|--------------|------------------------|-----------|
| A | … | API + service + DB | S/M/L |
| B | … | … | … |

Pick the slice that:

- Proves the **riskiest unknown** (integration, data shape, provider boundary)
- Is still shippable in a short loop
- Leaves an obvious extension path

**Good slice examples:** one API endpoint that persists one entity and returns
it; one worker message processed end-to-end; one UI action hitting a real
backend path.

**Bad slices:** “create all tables”; “build the whole UI shell”; “abstract
interfaces with no caller”; “docs-only design.”

Get explicit user agreement on the chosen slice before coding.

### 4. Slice contract (done definition)

Write before implementing:

```markdown
## Thin slice contract
- Name:
- Happy path (1 sentence):
- Entrypoint:
- Persistence / side effects:
- Response / observable result:
- Repro steps:
  1. ...
  2. ...
- Explicitly out of scope:
- Risks this slice is meant to kill:
```

### 5. Implement

- Build **only** what the contract needs
- Match existing patterns; no speculative abstractions
- Add the minimum test or manual repro harness the repo expects
- Recommended: run under **`push-ready build`** (small chunk → review → next)

### 6. Prove it

- Execute the repro steps (or automated tests)
- Paste/show evidence (pass output, status codes, DB row, screenshot note)
- If blocked by environment, document the exact blocker and the smallest
  substitute proof the user accepts

Only then mark: `THIN-SLICE: working`

### 7. Implementation plan (from the reference)

Now draft the larger plan. Every phase should say **how it extends the slice**.

```markdown
## Implementation plan (reference: thin slice)
### Reference slice
- Paths / symbols:
- Repro:
- Invariants we learned:

### Phase 1 — …
- Extends slice by:
- Files likely touched:
- Done when:

### Phase 2 — …
…

### Risks & open questions
- …

### Test plan
- …
```

Plan rules:

- Prefer extending the working path over rewriting it
- Order by dependency and risk, not by “clean architecture purity”
- Call out where the slice’s shortcuts must be hardened (auth, idempotency,
  scale, edge cases)
- Keep phases small enough that each could be another thin slice if needed

## Output checkpoints

1. After step 3: candidate table + recommended slice (wait for approval)
2. After step 4: slice contract
3. After step 6: `THIN-SLICE: working` + repro evidence
4. After step 7: implementation plan tied to the slice

## Coordination

| Skill | Role |
|-------|------|
| `thin-slice` | Choose + ship smallest working path, then plan |
| `push-ready` | Review/build quality while implementing the slice |
| `code-explainer` / `codebase-explainer` | Learn existing patterns before choosing the slice |

## Anti-patterns

- Writing a 10-phase plan with no running code
- Horizontal slices (“all models first”)
- Gold-plating the slice (extra variants, perfect abstractions)
- Throwing away the slice and planning a greenfield rewrite without reason
- Expanding scope mid-slice without updating the contract
- Declaring success without a repro
