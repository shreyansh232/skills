---
name: push-ready
description: >-
  High-signal iterative code review that beats one-shot PR reviews. Splits
  changes into small slices, then runs separated Standards vs Spec axes (plus
  correctness/security/perf lenses). Use for uncommitted work, branch diffs,
  agent-generated code, push-ready, code review, review since main, or make
  this merge-ready.
disable-model-invocation: true
argument-hint: "review | build | since <ref> (optional)"
---

# Push-Ready

One-shot reviews miss bugs because they skim a large agent diff in a polluted
context. This skill combines:

1. **Slice-first review** (your edge) — small modules, fix before moving on
2. **Two-axis separation** (Matt Pocock–style) — **Standards** vs **Spec**, never
   merged so one cannot mask the other
3. **Multi-lens correctness** (Addy Osmani–style) — correctness, architecture,
   security, performance, readability — high-signal only
4. **Fresh-context passes** when sub-agents exist — reviewer does not inherit
   the implementer’s rationalizations

Works with Claude Code, Codex, Cursor, and other Agent Skills hosts.

Deep checklists: [reference-lenses.md](reference-lenses.md).

## Modes

| Mode | When | Behavior |
|------|------|----------|
| `review` | Existing dirty tree / branch / “since X” | Intent → slices → per-slice → integration → **two-axis** final |
| `build` | Implementing now | Smallest slice → implement → review → next → two-axis final |

Default: `review` if there is a diff; else `build`.
`push-ready since main` (or any ref) pins the fixed point explicitly.

## Hard rules

1. **Split first.** Never start with one blob review of the whole feature.
2. Do not advance past a slice with open **critical/high** findings unless the
   user explicitly accepts the risk.
3. Every slice checks **happy path** and **edge/failure** cases with concrete
   examples (not vibes).
4. Prefer **bugs and regressions** over style. Skip anything formatters/linters
   already enforce.
5. Read `AGENTS.md` (+ standards docs) **once**; keep a checklist.
6. Check **consistency with existing codebase**, not only the isolated diff.
7. Final gate always separates **`## Standards`** and **`## Spec`** — do not
   blend into one ranked list.
8. **Approval standard:** `PUSH-READY: yes` only when the change clearly
   improves code health for the stated intent, with no open critical/high on
   either axis. Imperfect-but-healthy can pass; wrong-or-dangerous cannot.
9. When the Task/sub-agent tool exists, run Standards and Spec (and optionally
   Correctness) as **parallel fresh sub-agents**. Inline fallback only if
   sub-agents are unavailable.
10. Scrutinize **tests as hard as production code** (false confidence is worse
    than missing tests).

## Step 0 — Intent (Spec seed)

State in 3–6 bullets:

- Problem / user-visible goal
- Success criteria
- Non-goals
- Boundaries (API, DB, auth, infra)

Sources: user chat, commits, PR text, `FEATURE_SPEC.md` / `specs/` /
`.scratch/`, issue refs in commits. If missing, ask **once** or list
assumptions.

This becomes the **Spec** axis input.

## Step 0b — Standards sources (once)

Collect, in order:

1. `AGENTS.md` / `CLAUDE.md` pointers
2. `CONTRIBUTING.md`, `CODING_STANDARDS.md`, ADRs, architecture docs
3. Observed neighbor patterns in the files you will touch
4. Always include the **smell baseline** in [reference-lenses.md](reference-lenses.md)

Build a short checklist you will reuse. Repo docs **override** the baseline
where they conflict.

## Step 0c — Pin the fixed point

| User said | Fixed point |
|-----------|-------------|
| `since <ref>` | that ref |
| branch / PR review | `main`/`master`/`develop` merge-base (detect) |
| uncommitted only | working tree vs `HEAD` (and staged) |
| unspecified | ask once; default `main` if it exists else `HEAD~1` for committed WIP |

Validate:

```bash
git rev-parse <fixed-point>
git status
git diff <fixed-point>...HEAD    # committed range (three-dot)
git diff                         # unstaged
git diff --cached                # staged
git log --oneline <fixed-point>..HEAD
```

Empty diff → stop (“nothing to review”).

---

## Mode: `review`

### 1. Slice the change

Group into small concerns (often 1–3 files each): model, domain, API, worker,
tests, etc. Prefer more small slices.

Show the slice plan, then review **one slice at a time**.

### 2. Per-slice loop (high-signal)

For each slice:

1. Intent of this slice vs overall Spec
2. Read new code **and** neighbors (callers/callees/similar modules)
3. Apply lenses (details in reference file):
   - **Correctness** — happy + edge; races; wrong branch; state corruption
   - **Spec fidelity** — missing / partial / wrong / scope creep *in this slice*
   - **Standards / consistency** — AGENTS.md + patterns + smells
   - **Security** — trust boundaries, authz, injection, secrets in logs
   - **Performance** — N+1, unbounded, sync-on-async path
   - **Tests** — do they fail if the bug exists? Only happy path?
   - **AI-slop traps** — hallucinated APIs, dead code, swallowed errors,
     duplicate helpers, speculative abstractions
4. Report with the finding format below
5. Fix critical/high (or user accepts) before next slice

### 3. Integration pass

- Cross-slice contracts (types, statuses, invariants)
- End-to-end happy + failure traces
- Shared state / ordering / idempotency / partial failure
- Diff vs rest of codebase still coherent
- AGENTS.md final sweep

### 4. Two-axis final gate (required)

Spawn **parallel** sub-agents when possible (fresh context). Paste into each:

**Shared context:** fixed-point diff commands, commit list, intent bullets,
AGENTS/standards paths, slice summary.

**Standards agent** — follow Standards brief in [reference-lenses.md](reference-lenses.md)
(include full smell baseline). Output under 500 words.

**Spec agent** — follow Spec brief in reference file. If no spec/intent,
report `no spec available` and list assumption gaps. Under 500 words.

**Optional Correctness agent** (large or risky diffs) — security + concurrency +
test-quality brief from reference file.

Aggregate **verbatim under separate headings**:

```markdown
## Standards
...

## Spec
...

## Correctness (optional)
...

## Summary
- Standards: N findings (worst: …)
- Spec: N findings (worst: …)
- PUSH-READY: yes | no
- Blockers: …
```

Do **not** merge axes into one priority list.

---

## Mode: `build`

```
- [ ] Intent + standards once
- [ ] Smallest next problem
- [ ] Implement only that slice (match codebase)
- [ ] Per-slice review (all lenses)
- [ ] Fix critical/high
- [ ] Next slice
- [ ] Integration + two-axis final gate
```

Never “generate the whole feature then review once.”

---

## Finding format

```markdown
### Slice: <name>
Intent: <one line>

Checked: happy | edge | standards | spec | security | perf | tests | ai-traps

Findings:
- [critical] `path:line` — <bug> — <why it fails / evidence> — <fix>
- [high] ...
- [medium] ...
- [low] ...

Slice status: blocked | fixed | accepted-with-risk
```

| Severity | Meaning |
|----------|---------|
| critical | wrong behavior, data loss, security, likely crash |
| high | real bug on plausible path; spec miss; broken invariant |
| medium | should fix before push; missing important edge/test |
| low | polish; never block alone |

**Noise control:** Cap low findings at ~5 per slice. If you only have lows and
nits, say so and prefer `PUSH-READY: yes`.

---

## What “better than Codex defaults” means here

Codex-quality reviews usually win by (a) separating **did we build the right
thing?** from **did we build it in-house-correctly?**, (b) using a fixed diff
base, and (c) hunting real failure modes. This skill keeps those and adds
**slice-first iteration** + **AI-slop + test false-confidence** checks so agent
output cannot rubber-stamp itself.

## Anti-patterns

- One giant review “for speed”
- Blending Standards and Spec into one score
- Style-only commentary
- Approving because tests are green without reading them
- Reviewing only new files, ignoring callers/callees
- Re-reviewing in the same context that just wrote the code when a sub-agent
  is available
- Expanding scope while reviewing

## Coordination

- `implement-feature` / `thin-slice` — call push-ready mid-phase and at the end
- `ponytail` — minimize code; push-ready still verifies correctness/spec
- Dedicated security audit tools — optional extra after `PUSH-READY: yes` for
  sensitive surfaces
