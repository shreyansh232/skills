---
name: session-handoff
description: >-
  Compact the live session into a handoff document and stop so work continues
  in a fresh chat. Use when context is large (~100k+ tokens), the host shows
  high context usage, quality is degrading, the user says handoff,
  session-handoff, /handoff, context full, new chat, or continue from handoff.
  Also use to resume: session-handoff continue <path>.
argument-hint: "optional: focus for next session | continue <path>"
---

# Session Handoff

Long threads get dumber before the context window is actually full. Prefer a
**fresh chat + handoff doc** over stuffing more into a bloated session.

## Threshold (default)

| Signal | Action |
|--------|--------|
| **~100k tokens used** (soft) | Stop productive work; run this skill now |
| **~120k+** (hard) | Emergency handoff — do not add more exploration |
| Host UI “context nearly full” | Treat as soft trigger |
| Noticeable quality drop / loops / forgotten decisions | Treat as soft trigger even below 100k |

**Do not wait for 150k.** For many coding models the “smart zone” fades around
100–120k even when the marketed window is 200k–1M. 100k is the right default
budget; only raise it if you have measured that *your* model stays sharp longer.

Agents often lack an exact token counter. If the UI shows ≥100k, or the user
says context is high, or the session has been very long with many tool rounds —
**handoff proactively**. Prefer an early handoff over a late one.

## Modes

| Mode | Trigger | Behavior |
|------|---------|----------|
| `write` (default) | handoff / context high | Write handoff doc, stop, give bootstrap for next chat |
| `continue` | `session-handoff continue [path]` or user pastes handoff | Load doc, resume exactly where left off |

---

## Mode: `write`

### Hard rules

1. **Stop** implementing/reviewing after the doc is written — end the turn with
   clear “open a new chat” instructions. Do not keep coding in this thread.
2. **Reference, don’t dump.** Point to specs, plans, ADRs, commits, diffs, tests
   by path/URL. Do not paste large file bodies already on disk.
3. **Redact secrets** (API keys, tokens, passwords, PII, private URLs with
   credentials).
4. Tailor the doc to the next focus if the user gave an argument.
5. Include **Suggested skills** the next agent must invoke (e.g. `push-ready`,
   `implement-feature`, `thin-slice`, `ponytail`).

### Where to save

Prefer (first writable path wins):

1. Workspace: `.handoff/HANDOFF-<YYYYMMDD-HHMM>.md` (create `.handoff/`; add to
   `.gitignore` if missing — do not force-add ignored files)
2. Else OS temp: `$TMPDIR/handoff-<slug>-<YYYYMMDD-HHMM>.md` (macOS/Linux) or
   `%TEMP%\handoff-...md` (Windows)

Print the **absolute path** prominently.

Optional: if `.gitignore` exists and `.handoff/` is not ignored, append
`.handoff/` in a minimal edit (only if clearly desired / already a personal
scratch pattern). If unsure, use OS temp instead of polluting git status.

### Handoff document template

```markdown
# Handoff — <short title>
Updated: <ISO timestamp>
Repo: <path or remote>
Branch: <branch>
Fixed point / base (if any): <ref>
Next session focus: <one sentence>

## Goal
<what we are trying to achieve>

## Status
- Done:
- In progress (exact):
- Blocked:

## Decisions (do not re-litigate)
- …

## Artifacts (read these; do not recreate)
- Spec: `<path>`
- Plan: `<path>`
- Slice / proof: `<path or test command>`
- Diff / commits: `<git status summary + key SHAs>`
- Other: …

## Working tree
```
<git status --short>
```
Key uncommitted paths and what each is for (one line each).

## Next actions (ordered)
1. …
2. …
3. …

## Suggested skills
- `skill-name` — why
- …

## Risks / landmines
- …

## Bootstrap prompt (paste into new chat)
```
Continue from handoff: <ABSOLUTE_PATH_TO_THIS_FILE>

Read that file first. Then run session-handoff continue (or follow its Next actions).
Do not re-discover settled decisions. Resume implementation/review from Next actions.
```
```

### End of turn (required)

Tell the user:

1. Absolute path to the handoff file  
2. **Stop using this chat** for the feature  
3. Open a **new chat** and paste the Bootstrap prompt (or `@` the file if the
   host supports it)  
4. Optional one-liner: `session-handoff continue <path>`

True auto-open of a new chat is host-dependent; the skill always provides a
copy-paste bootstrap so continuation is one step.

---

## Mode: `continue`

1. Resolve path (argument, or `.handoff/` latest, or user-attached file).
2. Read the full handoff.
3. Briefly restate Goal + Status + Next action #1 (≤10 lines).
4. Invoke **Suggested skills** as appropriate.
5. Execute Next actions in order without re-asking settled Decisions.
6. If context again approaches ~100k, run **write** mode again (chain of
   handoffs is expected on large features).

---

## Anti-patterns

- Continuing to code in the bloated session after writing the handoff
- Pasting entire specs/diffs into the handoff
- Leaving secrets in the doc
- Vague “continue the work” with no Next actions
- Re-grilling decisions already listed under Decisions
- Waiting for 150k+ tokens before handing off

## Coordination

| Skill | With handoff |
|-------|----------------|
| `implement-feature` | Handoff between phases if context is high |
| `push-ready` / `thin-slice` | Finish current slice checkpoint, then handoff |
| Matt Pocock `handoff` | Compatible idea; this skill adds **100k threshold**, workspace `.handoff/`, and **continue** bootstrap |
