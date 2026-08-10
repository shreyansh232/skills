---
name: push-ready
description: >-
  Iterative code review for agent-generated work. Reviews small modules one at
  a time (not one giant pass), then checks how pieces connect. Use for
  uncommitted/local changes or while building a feature slice-by-slice. Use when
  the user says push-ready, iterative review, review as you go, make this
  push-ready, or wants tighter review than a single end-of-PR pass.
disable-model-invocation: true
argument-hint: "review | build (optional)"
---

# Push-Ready

Most one-shot reviews miss bugs in agent-generated code because they skim a
large diff after everything already exists. This skill forces **small units**:
understand intent → review (or build) one slice → only then move on → finally
check how slices connect.

Works with any Agent Skills–compatible tool (Claude Code, Codex, Cursor, etc.).

## Modes

| Mode | When | Behavior |
|------|------|----------|
| `review` | Code already exists (uncommitted, staged, or described) | Infer feature intent, split into small parts, review each, then integration |
| `build` | Implementing a new feature with an agent | Smallest next problem → implement → review hard → only then next slice |

If unspecified: use `review` when there is a local diff / dirty tree; else `build`.

## Hard rules

1. **Never** review the entire feature as one blob first. Split first.
2. Do not mark a slice done until its review findings are fixed or explicitly
   accepted by the user.
3. Every slice review must cover **happy-path / normal cases** and **edge cases**
   where the code can fail or silently mess up.
4. Prefer concrete bugs (wrong condition, missing await, race, leaky abstraction,
   broken invariant) over style nits unless style breaks correctness/clarity.
5. After all slices: do an **integration pass** — how parts compose, shared state,
   ordering, error propagation, idempotency.
6. Do not expand scope. Review/build only what the feature requires.

## Step 0 — Recover the requirement

Before reviewing or coding, state the feature intent in 3–6 bullets:

- What user/system problem this solves
- Success criteria (observable behavior)
- Non-goals / out of scope
- Relevant boundaries (API, DB, queue, auth, infra)

Sources (use what exists):

- User’s description in chat
- PR/commit messages if any
- Diff + nearby tests/docs
- Related tickets/links the user pasted

If intent is unclear, ask **one** clarifying question, then proceed with an
explicit assumption list.

## Mode: `review` (existing generated / uncommitted code)

### 1. Inventory the change

```bash
git status
git diff
git diff --cached
```

If the branch has commits ahead of base, also skim:

```bash
git log --oneline <base>..HEAD
git diff <base>...HEAD
```

Group files into **slices** (small modules / concerns), e.g.:

- slice A: data model / migration
- slice B: service/domain logic
- slice C: API/route wiring
- slice D: tests

A slice should be reviewable in isolation (~one concern, often 1–3 files). Prefer
more small slices over fewer large ones.

Present the slice plan to the user, then review **one slice at a time**.

### 2. Per-slice review loop

For each slice:

1. Summarize what this slice is supposed to do (tied to the requirement).
2. Read the code carefully (not skim).
3. Check **normal cases**: expected inputs, typical control flow, return shapes,
   status codes, persistence that should happen.
4. Check **edge cases / failure modes** (always; invent concrete ones):
   - empty/null/missing fields
   - duplicates / retries / double-submit
   - concurrency / ordering
   - partial failure (DB ok, queue fail, etc.)
   - authz / tenancy boundaries
   - timeouts, cancellation, idempotency
   - off-by-one, wrong comparison, inverted boolean
   - error swallowed or wrong exception type
5. Report findings for **this slice only** using the format below.
6. Fix critical/high issues (or get user accept) before the next slice.

### 3. Integration pass (required)

After every slice is clean enough:

- Do the pieces agree on types, names, status values, and invariants?
- Can slice B violate an assumption slice A relies on?
- End-to-end happy path: does the feature actually work as required?
- End-to-end failure path: one failure shouldn’t corrupt state or double-apply.
- Tests: do they cover the risky edges, or only the happy path?

### 4. Push-ready verdict

End with:

- `PUSH-READY: yes` — no open critical/high issues
- `PUSH-READY: no` — list remaining blockers

## Mode: `build` (generate + review iteratively)

Use while implementing a feature:

```
Build loop:
- [ ] 1. State / refine requirement (Step 0)
- [ ] 2. Pick the SMALLEST next problem that unlocks progress
- [ ] 3. Implement only that slice
- [ ] 4. Run the per-slice review (normal + edge cases)
- [ ] 5. Fix findings
- [ ] 6. Only then choose the next smallest problem
- [ ] 7. When feature complete → integration pass → verdict
```

**Smallest next problem** means the minimal coherent change (e.g. one function,
one route handler, one schema field + validation) — not “the whole service.”

Do not draft the full feature in one shot and “review later.”

## Finding format (per slice)

```markdown
### Slice: <name>
Intent: <one line>

Normal cases checked:
- ...

Edge cases checked:
- ...

Findings:
- [critical] `path:line` — <bug> — <why it fails> — <fix direction>
- [high] ...
- [medium] ...
- [low] ...

Slice status: blocked | fixed | accepted-with-risk
```

Severity:

- **critical** — wrong behavior, data loss, security, crash in likely paths
- **high** — real bug in plausible edge/normal path
- **medium** — likely issue or missing safety; should fix before push
- **low** — polish / clarity; don’t block unless easy

## Anti-patterns

- One giant review of the full diff “for speed”
- Only listing style/naming issues
- Skipping happy-path reasoning (“looks fine”)
- Skipping edge cases
- Implementing the next slice while the current one still has open high+ findings
- Rubber-stamping agent output because tests pass (tests may be shallow)

## Coordination

- Deep security pass → use a dedicated security review skill if available
- Blind “find bugs in whole PR” bots → optional *after* push-ready slices are clean
- This skill owns **iterative correctness** tied to the feature requirement
