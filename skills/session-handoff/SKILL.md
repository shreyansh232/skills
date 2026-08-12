---
name: session-handoff
description: >-
  Compact the session into a handoff doc in OS temp and stop for a fresh chat.
  Use at ~100k context, quality drop, or when the user says handoff,
  session-handoff, /handoff, context full, or continue from handoff.
argument-hint: "optional focus | continue <path>"
---

# Session Handoff

Prefer a fresh chat over a bloated one. Soft stop **~100k** tokens; hard **~120k**.
Do not wait for 150k. If the UI shows high context or quality drops, handoff now.

## `write` (default)

1. Write a short handoff to **OS temp only** (never the workspace):
   - macOS/Linux: `$TMPDIR/handoff-<slug>-<YYYYMMDD-HHMM>.md`
   - Windows: `%TEMP%\handoff-<slug>-<YYYYMMDD-HHMM>.md`
2. **Stop coding** in this chat. Print the absolute path + bootstrap prompt.
3. Redact secrets. Reference specs/plans/diffs by path — don’t paste them.
4. If the user gave a focus argument, tailor “Next session focus” to it.

### Doc shape

```markdown
# Handoff — <title>
Updated: <ISO> | Repo: <path> | Branch: <branch>
Next focus: <one line>

## Goal
## Status (done / in progress / blocked)
## Decisions (do not re-litigate)
## Artifacts (paths only)
## Working tree (`git status --short` + one-liners)
## Next actions (ordered)
## Suggested skills
## Risks

## Bootstrap (paste into new chat)
Continue from handoff: <ABS_PATH>
Read it first. Run session-handoff continue if needed. Resume Next actions. Do not redo Decisions.
```

## `continue`

Read the path (or user-attached file). Restate goal + next action #1. Follow
Suggested skills and Next actions. Handoff again if context nears ~100k.

## Anti-patterns

- Saving under the workspace
- Coding after writing the handoff
- Dumping large file bodies into the doc
- Leaving secrets in the doc
