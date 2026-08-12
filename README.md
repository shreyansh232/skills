# skills

Personal agent skills (Claude Code, Codex, Cursor, and other [Agent Skills](https://skills.sh/)–compatible tools).

| Skill | Purpose |
|-------|---------|
| `implement-feature` | Full pipeline: grill-me → spec → thin-slice (TDD) → plan → ponytail build + push-ready reviews |
| `push-ready` | High-signal review: slice-first + Standards/Spec axes + correctness/security/AI-slop lenses |
| `thin-slice` | Before a big feature: ship the smallest end-to-end working slice, then plan the rest from that reference |
| `session-handoff` | At ~100k context (or on demand): write a handoff doc, stop, resume in a fresh chat via continue |
| `code-explainer` | First-principles walkthrough of a module with multiple examples and how it fits the whole codebase |
| `codebase-explainer` | Whole-repo tour from the critical core outward: examples, snippets, and end-to-end API/job traces |

## Install

```bash
npx skills add shreyansh232/skills -g --all
```

### Dependencies for `implement-feature`

```bash
npx skills add mattpocock/skills -g -s grill-me -s grilling -y
npx skills add https://github.com/DietrichGebert/ponytail -g -s ponytail -y
```

Useful variants:

```bash
npx skills add shreyansh232/skills -l
npx skills add shreyansh232/skills -g -s implement-feature -y
npx skills add shreyansh232/skills -g -s push-ready -y
npx skills add shreyansh232/skills -g -s thin-slice -y
npx skills add shreyansh232/skills -g -s session-handoff -y
npx skills add shreyansh232/skills -g -s code-explainer -y
npx skills add shreyansh232/skills -g -s codebase-explainer -y
npx skills add shreyansh232/skills -g -a claude-code -a codex -a cursor -y --skill '*'
npx skills update -g
```

## Usage

- `implement-feature <feature>` — clarify (grill-me) → spec → thin-slice TDD → plan with files-to-touch → ponytail implement + push-ready mid/final reviews
- `push-ready` / `push-ready review` / `push-ready since main` — slice-first review, then Standards vs Spec final gate
- `push-ready build` — while implementing: smallest slice → implement → review → only then continue
- `thin-slice <feature>` — prove the smallest working end-to-end path first, then write the full implementation plan from it
- `session-handoff` / `/handoff` — when context ~100k (or on demand): write handoff doc and open a **new** chat
- `session-handoff continue <path>` — resume from a handoff document in a fresh session
- `code-explainer <path-or-topic>` — break down a module from first principles with multiple examples and system fit
- `codebase-explainer` — foundation-up tour of the whole repo (critical module first, chapter by chapter)
- `codebase-explainer focus <domain>` — same, but re-rank the core toward a domain
