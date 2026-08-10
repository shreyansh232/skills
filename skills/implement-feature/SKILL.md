---
name: implement-feature
description: >-
  End-to-end feature delivery pipeline: grill-me clarification → written spec →
  thin-slice (TDD) → implementation plan with files-to-touch → ponytail build
  with push-ready reviews mid-flight and a final push-ready pass. Use when the
  user says implement-feature, ship this feature, build this feature end to
  end, or wants the full clarify→slice→plan→implement loop.
disable-model-invocation: true
argument-hint: "feature description"
---

# Implement Feature

Orchestrates a full feature lifecycle. Do not skip phases. Do not jump to
coding before the grill + spec + thin slice are done.

## Prerequisites (must be installed)

| Skill | Source | Role here |
|-------|--------|-----------|
| `grill-me` (+ `grilling`) | [mattpocock/skills](https://github.com/mattpocock/skills) | Clarify expectations |
| `thin-slice` | this repo (`shreyansh232/skills`) | Smallest working path |
| `ponytail` | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | Minimal correct implementation |
| `push-ready` | this repo | Iterative + final review |

If a required skill is missing, stop and tell the user how to install it:

```bash
npx skills add mattpocock/skills -g -s grill-me -s grilling -y
npx skills add https://github.com/DietrichGebert/ponytail -g -s ponytail -y
npx skills add shreyansh232/skills -g --all
```

Then **read and follow** each skill’s own `SKILL.md` when that phase runs
(load the installed skill file; do not reinvent it).

## Hard rules

1. Phase order is fixed. Never implement the full feature before
   `THIN-SLICE: working`.
2. **TDD everywhere code is written** (thin slice + full build):
   failing tests first → minimal code to pass → refactor → next.
3. Read repo `AGENTS.md` **once** at the start; keep a checklist for the session.
4. Mid-build: invoke **`push-ready`** after each meaningful slice/phase.
5. End: invoke **`push-ready review`** on the full change for a final verdict.
6. During full implementation, keep **`ponytail`** active (default **full**
   unless user asks lite/ultra). Never strip trust-boundary validation, data-loss
   handling, security, or accessibility to “be lazy.”
7. Get explicit user confirmation at the checkpoints listed below before
   continuing.

## Pipeline checklist

```
implement-feature:
- [ ] 0. Scout AGENTS.md + confirm prerequisite skills
- [ ] 1. grill-me until shared understanding (user confirms)
- [ ] 2. Write FEATURE SPEC (user confirms)
- [ ] 3. thin-slice + TDD until THIN-SLICE: working (user confirms)
- [ ] 4. Learn from slice → IMPLEMENTATION PLAN with files-to-touch (user confirms)
- [ ] 5. ponytail + TDD build per plan phase
- [ ] 6. push-ready after each phase
- [ ] 7. Final push-ready on the whole feature → PUSH-READY: yes/no
```

---

## Phase 0 — Session setup

1. Resolve feature ask from the user (one paragraph restatement).
2. Read `AGENTS.md` once (boundaries, test commands, architecture).
3. Verify `grill-me`/`grilling`, `thin-slice`, `ponytail`, `push-ready` are
   available; if not, print install commands and stop.

---

## Phase 1 — Clarify with `grill-me`

1. Load and follow Matt Pocock’s **`grill-me`** (runs a **`grilling`** session).
2. Subject of the grill: the feature the user wants — scope, actors, success
   criteria, non-goals, edge cases, data, auth, failure modes, rollout, and
   anything else needed to implement end-to-end without silent assumptions.
3. Look up codebase facts yourself; only put **decisions** to the user.
4. Stop when the grilling frontier is empty and the user confirms shared
   understanding.

**Checkpoint:** user says the understanding is correct.

---

## Phase 2 — Feature spec

Write a concise spec from the grilled decisions (do not start coding):

```markdown
# Feature spec: <name>

## Problem
## Goals / success criteria
## Non-goals
## Users / actors
## Happy path
## Edge cases & failure modes (must handle)
## Data & contracts (APIs, events, schemas)
## Auth / tenancy / safety
## Observability
## Out of scope for v1
## Acceptance checks (testable)
## Open questions (must be empty or explicitly deferred)
```

**Checkpoint:** user approves the spec.

---

## Phase 3 — Thin slice (TDD)

1. Load and follow **`thin-slice`**.
2. Choose the smallest vertical slice that proves the riskiest unknown in the
   spec.
3. **TDD for the slice:**
   - Write the smallest failing test(s) that encode the slice contract
   - Implement the minimum to pass
   - Repro / run tests until green
4. Optionally use `push-ready build` while slicing.
5. Mark `THIN-SLICE: working` only with evidence (test output / repro).

**Checkpoint:** user accepts the working slice.

---

## Phase 4 — Implementation plan (from the slice)

Using the working slice as the **reference**, write a robust plan:

```markdown
# Implementation plan: <name>

## Reference thin slice
- Paths / symbols:
- Tests:
- Invariants learned:
- Shortcuts to harden later:

## Robustness & edge cases
- List each edge/failure from the spec and how it will be handled
- Concurrency / idempotency / retries / partial failure notes

## Phases
### Phase A — …
- Goal:
- Files to touch (create/modify) — guide for the agent:
  - `path/a.py` — why
  - `path/b.py` — why
  - `tests/...` — why
- Tests to add first (TDD):
- Done when:
- Risks:

### Phase B — …
…

## Non-goals / defer
## Test plan (unit / integration / manual)
## Rollout / migration notes (if any)
```

Plan rules:

- Prefer extending the slice over rewriting it
- Every phase lists **concrete files to touch**
- Every phase lists **tests to write first**
- Call out where ponytail shortcuts need hardening

**Checkpoint:** user approves the plan.

---

## Phase 5 — Full build with `ponytail` + TDD + `push-ready`

1. Load and follow **`ponytail`** ([DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail))
   at **full** unless the user sets another level. Keep it active for the whole
   build (“stop ponytail” only if the user says so).
2. For **each plan phase**, in order:
   1. Write failing tests first (from the plan’s TDD list)
   2. Implement the minimum to pass (ponytail ladder)
   3. Refactor only while tests stay green
   4. Run **`push-ready`** on that phase’s diff (slice review: AGENTS.md,
      consistency, happy + edge cases)
   5. Fix critical/high findings before the next phase
3. Do not skip phases or batch the entire feature into one unreviewed blob.

---

## Phase 6 — Final `push-ready`

1. Run **`push-ready review`** on the **complete** uncommitted/branch change.
2. Require integration pass + AGENTS.md checklist + codebase consistency.
3. End with `PUSH-READY: yes` or `PUSH-READY: no` and blockers.
4. Only suggest commit/PR language when `PUSH-READY: yes` (do not commit unless
   the user asks).

---

## Artifacts to keep in the conversation (or files if user asks)

| Artifact | When |
|----------|------|
| Grilled decision summary | after phase 1 |
| Feature spec | after phase 2 |
| Thin-slice contract + proof | after phase 3 |
| Implementation plan | after phase 4 |
| Phase push-ready notes | during phase 5 |
| Final push-ready verdict | after phase 6 |

Prefer chat unless the user asks to write `FEATURE_SPEC.md` / `IMPLEMENTATION_PLAN.md`
into the repo (and only then, following repo doc norms).

## Anti-patterns

- Coding during grill-me
- Spec that still has unresolved critical questions
- Full feature scaffold before thin slice works
- Implementation without failing tests first
- Ponytail used as an excuse to skip validation/security/tests
- One giant review only at the end (must also review mid-phase)
- Ignoring `AGENTS.md` or existing codebase patterns
