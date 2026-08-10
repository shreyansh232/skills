# skills

Personal agent skills (Claude Code, Codex, Cursor, and other [Agent Skills](https://skills.sh/)–compatible tools).

| Skill | Purpose |
|-------|---------|
| `push-ready` | Iterative review/build: small slices → AGENTS.md + codebase consistency → happy/edge review → integration → verdict |

## Install

```bash
npx skills add shreyansh232/skills -g --all
```

Useful variants:

```bash
npx skills add shreyansh232/skills -l
npx skills add shreyansh232/skills -g -s push-ready -y
npx skills add shreyansh232/skills -g -a claude-code -a codex -a cursor -y --skill '*'
npx skills update -g
```

## Usage

- `push-ready` / `push-ready review` — uncommitted or existing generated code: recover intent, split into small parts, review one-by-one, then how they connect
- `push-ready build` — while implementing: smallest slice → implement → review → only then continue
