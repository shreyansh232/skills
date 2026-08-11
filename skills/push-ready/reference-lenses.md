# Push-Ready review lenses

Loaded by `push-ready` when doing deep checks. Keep briefs tight when pasting
into sub-agents.

## Smell baseline (Fowler)

Repo standards override these. Smells are **judgement calls**, never hard
violations by themselves. Skip anything tooling already enforces.

| Smell | What it is | Typical fix |
|-------|------------|-------------|
| Mysterious Name | Name hides purpose | Rename; if unnamed, redesign |
| Duplicated Code | Same logic shape repeated in the change | Extract shared helper |
| Feature Envy | Method uses another object’s data more than its own | Move method to the data |
| Data Clumps | Same field group travels together | Introduce a small type |
| Primitive Obsession | String/int stands for a domain concept | Small domain type |
| Repeated Switches | Same type cascade repeated | Polymorphism or one shared map |
| Shotgun Surgery | One change edits many scattered sites | Gather into one module |
| Divergent Change | One module changes for unrelated reasons | Split by reason |
| Speculative Generality | Abstractions for unneeded futures | Delete until real need |
| Message Chains | `a.b().c().d()` coupling | Hide behind one method |
| Middle Man | Mostly delegates | Remove; call target |
| Refused Bequest | Ignores most of inherited API | Prefer composition |

## AI-generated code traps

Hunt these explicitly — common in agent diffs:

- Hallucinated methods, flags, env vars, or config keys
- Imports / helpers added but unused or duplicated nearby
- `try/except` / `catch` that swallows and continues
- Copy-paste with one branch not updated (renames half-done)
- Tests that mock the system under test until assertions are vacuous
- Comments that describe intended behavior the code does not do
- “Temporary” flags / dead feature branches left enabled
- New abstraction with a single caller (speculative generality)
- Logging PII/secrets or huge payloads
- Async functions never awaited; fire-and-forget without supervision
- Permission checks on UI only, missing on API/worker

## Correctness lens

- Matches stated intent / spec for this slice?
- Null/empty/boundary inputs?
- Error paths real (not only happy path)?
- Off-by-one, wrong comparison, inverted boolean?
- Concurrency: double-submit, lost update, stale read?
- Idempotency / retries / partial failure?
- State consistency after failure mid-way?

## Spec lens (brief for Spec sub-agent)

Report only:

1. Requirements missing or partial (quote intent/spec)
2. Behavior in the diff that was **not** asked (scope creep)
3. Requirements that look done but implementation looks **wrong**

Cite evidence (path/hunk). Under 500 words. No style nits.

## Standards lens (brief for Standards sub-agent)

Report only:

1. Violations of repo docs (`AGENTS.md`, CONTRIBUTING, ADRs) — cite rule
2. Baseline smells from the table above — name smell + hunk
3. Inconsistency with neighboring modules (patterns, layering, duplicated helper)

Distinguish **hard** (documented rule) vs **judgement** (smell/consistency).
Skip formatter/linter territory. Under 500 words.

## Security lens

- Untrusted input validated at boundary?
- Authn/authz on every sensitive path (not only UI)?
- Injection (SQL/command/template)? Parameterized?
- Secrets out of code, logs, fixtures?
- Tenancy / org scoping preserved?
- Unsafe deserialization / path traversal?

## Performance lens

- N+1 queries or per-item remote calls in a loop?
- Unbounded list/fetch without limit?
- Blocking sync I/O on request path?
- Hot-path allocations / repeated heavy work?
- Missing pagination / backpressure?

## Test quality lens

- Would this test fail if the production bug existed?
- Assertions on behavior, not only “no throw”?
- Edge/failure cases covered, or only happy path?
- Over-mocking hiding integration mistakes?
- Regression test for the bug fix (when reviewing a fix)?
- Snapshot/assert everything anti-pattern?

## Structural remedies (propose the move)

When flagging structure, name the fix:

- Replace conditional chains with a model/dispatcher
- Collapse duplicate branches
- Separate orchestration from domain logic
- Move feature logic out of shared modules
- Reuse the canonical helper instead of a near-duplicate
- Make type boundaries explicit
- Delete pass-through wrappers
- Extract helper / split oversized file

Prefer remedies that **remove** moving pieces.

## Change sizing signals

| Diff size | Guidance |
|-----------|----------|
| ~≤100 LOC | Ideal slice |
| ~300 LOC | OK if one concern |
| ~1000+ LOC | Split; push-ready should insist on slices |

Refactor-only vs feature behavior → prefer separate commits when mixed.

## Optional Correctness sub-agent brief

Using the diff + intent: list critical/high bugs only across correctness,
security, performance, and test false-confidence. Concrete `path:line`,
failure scenario, fix direction. Under 500 words. No style.
