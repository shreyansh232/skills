---
name: codebase-explainer
description: >-
  Explain an entire codebase from first principles, starting with the most
  critical module and building outward. Uses multiple examples, real code
  snippets, and end-to-end traces (API/job paths). Use when the user says
  codebase-explainer, explain this codebase, map the system, teach me the
  architecture, or wants a foundation-up tour of the repo.
disable-model-invocation: true
argument-hint: "optional focus domain, e.g. dialing, auth, frontend"
---

# Codebase Explainer

Whole-repo teaching skill. Builds on the same pedagogy as `code-explainer`
(first principles, multiple examples, system fit) but sequences learning
across the **entire codebase**: critical core first, then layers outward.

Works with any Agent Skills–compatible tool (Claude Code, Codex, Cursor, etc.).

If the user only wants one file/module, prefer **`code-explainer`** instead.

## Hard rules

1. **Critical core first.** Identify the module/path without which the product
   does not exist. Explain that thoroughly before peripheral services.
2. **First principles.** For every layer: problem → concepts → mechanics →
   fit. Never start with a directory listing.
3. **Multiple examples** for every important behavior (≥2–3: happy, edge,
   distinctive).
4. **Real snippets** from this repo (short, cited with path). No fake
   pseudocode when the real function exists.
5. **Always show traces.** For each major capability, walk
   entry → hops → side effects → response/result, naming real symbols/files.
6. **Build a foundation.** Later sections must explicitly reuse concepts from
   earlier ones (“this queue uses the same idempotency idea as …”).
7. Read `AGENTS.md` **once** (and linked project guidelines) for boundaries;
   fold those into the map.
8. Prefer depth on the spine of the system over shallow coverage of every
   folder. Say what you are deferring.
9. Keep sessions chunked: deliver **one foundation chapter** at a time unless
   the user asks for a full dump. End each chapter with “next chapter” options
   and 2–3 check-understanding questions.

## Session workflow

```
Codebase explainer:
- [ ] 1. Scout: purpose, stack, AGENTS.md, entrypoints
- [ ] 2. Pick the critical core (+ say why)
- [ ] 3. Chapter 1: core from first principles + examples + traces
- [ ] 4. Chapter 2+: next most important dependency/capability
- [ ] 5. Continuously redraw the map as pieces lock in
- [ ] 6. Deferrals list (what remains)
```

### 1. Scout (short)

Produce a compact scout card before teaching:

- Product in one sentence
- Primary runtime paths (HTTP API, workers, agents, UI, cron/Temporal, …)
- Data stores / external systems
- Top-level layout (only as orientation, not the lesson)
- Constraints from `AGENTS.md` (boundaries you must respect while explaining)

### 2. Choose the critical core

Pick **one** starting module/subsystem using this preference order:

1. The main domain transaction (e.g. “place call”, “charge”, “ingest event”)
2. Else the primary request path (API → service → persistence)
3. Else the domain model that everything else orbits

State explicitly:

- **Core:** `path` / symbol
- **Why first:** …
- **What this unlocks next:** …

Do not start with config, CI, or pure utilities.

## Chapter structure (every chapter)

### A. Problem (first principles)

What breaks in the product if this part does not exist? One concrete example.

### B. Concepts

2–5 ideas. Each with a plain definition + example.

### C. Mechanics with snippets

Walk the real code in small steps. Quote minimal snippets:

```12:20:path/to/file.py
# only the lines that teach the point
```

### D. Traces (required)

At least one full trace table or numbered path:

| Step | Where | What happens | Example value |
|------|-------|--------------|---------------|
| 1 | `api/...` | receives request | `POST /v1/...` body … |
| 2 | `service/...` | validates / loads | … |
| 3 | `infra/...` | side effect | Redis/DB/queue … |
| 4 | response | returns | status + payload … |

Also include:

- **Edge/failure trace** (auth fail, not found, downstream timeout, retry)
- **Alternate trace** if a second entrypoint exists (worker vs API, UI vs webhook)

For APIs specifically cover:

- Who calls it
- Request shape (example JSON/params)
- Auth / tenancy checks
- Where it goes (services, DB, queues)
- What it returns (example)
- What it does *not* do (boundary)

### E. Bigger picture

Update a running mental map:

- How this chapter plugs into previous chapters
- Upstream / downstream
- Shared invariants

### F. Check understanding + next

2–3 application questions, then propose the next chapter (the next most
critical missing piece).

## Suggested chapter sequence (adapt to the repo)

Use as a default spine; rename to match the product:

1. **Critical domain transaction** (core module)
2. **Primary API (or UI) entry** that triggers it
3. **Persistence / state** the transaction needs
4. **Async / workers / orchestration** if any
5. **Auth, tenancy, safety rails**
6. **Observability / webhooks / secondary surfaces**
7. **Frontend or admin** only after backend spine is clear (or reverse if the
   repo is UI-first)

Skip chapters that do not apply. Insert repo-specific critical paths early
(e.g. dial plan, billing, ingest).

## Example density

Bad: “The API layer talks to services which talk to the database.”

Good: “`POST /calls` in `api/calls.py` builds a `DialCommand`; example body
`{...}` → `CallService.start` → Redis DNC check → Temporal workflow `...` →
returns `{call_id: ...}`. If DNC hits, trace stops at … and returns …”

If two sentences pass with no example or trace, add one.

## Depth knobs

| User says | Do this |
|-----------|---------|
| default | Chapter-by-chapter; wait for “continue” |
| `full map` | Scout + core + thin traces for top 3 paths, then offer depth |
| `focus <domain>` | Re-rank critical core toward that domain |
| `deeper` | Expand current chapter one dependency level |
| `eli5` | More analogy; keep real names + traces |

## Coordination with `code-explainer`

- Whole system / architecture foundation → **codebase-explainer**
- Single module deep-dive mid-tour → switch to **code-explainer**, then return
- After a codebase chapter, offer: “Zoom with `code-explainer` on `path`?”

## Anti-patterns

- Starting at `utils/` or listing every directory
- Architecture slideshow with no traces or snippets
- Explaining everything in one message
- Examples that are not from this repo’s real shapes
- Ignoring `AGENTS.md` boundaries (e.g. describing app code calling providers
  directly when infra boundary forbids it)
- Teaching peripheral tools before the main product path
