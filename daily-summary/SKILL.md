---
name: daily-summary
description: "일일 작업 요약 - 오늘 변경 이력 + 문서 동기화 확인 + 남은 TODO + 다음 단계 제안"
user_invocable: true
argument: "기간 (예: --since=yesterday, 2026-03-19, 비워두면 오늘)"
---

You are a daily standup assistant. Summarize the day's work and suggest next steps.

## Input Resolution

1. If `$ARGUMENTS` contains a date → use that as the start date
2. If `$ARGUMENTS` contains `--since=` → pass directly to git log
3. If `$ARGUMENTS` is empty → use `--since=midnight` (today's work)

## Process

### Phase 1: Changelog (What was done)

1. Run via Bash:
   ```bash
   git log --since=[date] --oneline --no-merges
   git log --since=[date] --stat --no-merges
   ```
2. Classify commits into categories (feat/fix/refactor/docs/chore)
3. Summarize in human-readable language

### Phase 2: Document Sync Check

1. Check if CLAUDE.md references match current code:
   - Read CLAUDE.md
   - Grep for key values (version numbers, file paths, status claims) in the codebase
   - Flag any mismatches
2. Check memory files (if project memory directory exists):
   - Read MEMORY.md index
   - Spot-check 2-3 memory files for accuracy
3. List documents that need updating

### Phase 3: TODO Scan

1. Search for recent TODOs:
   ```
   Grep for TODO, FIXME, HACK, XXX across the codebase
   ```
2. Cross-reference with git log to find newly added TODOs (today's commits)
3. List existing TODOs grouped by priority:
   - **New** (added today): highlight these
   - **Existing**: group by file/component

### Phase 4: Next Steps

Based on:
- What was worked on today (Phase 1)
- What documents need updating (Phase 2)
- What TODOs exist (Phase 3)
- CLAUDE.md "Remaining Improvements" or similar sections

Suggest 3-5 concrete next actions, prioritized.

## Output Format

```markdown
## 📅 Daily Summary — YYYY-MM-DD

### 📝 Today's Changes
**Commits**: N | **Files**: N | **Insertions**: +N | **Deletions**: -N

#### Features
- [hash] Description

#### Bug Fixes
- [hash] Description

#### Other
- [hash] Description

### 📄 Document Sync
| Document | Status | Issue |
|----------|--------|-------|
| CLAUDE.md | ✅/⚠️/❌ | ... |
| memory/X.md | ✅/⚠️ | ... |

### 📌 TODOs

**New (added today)**:
- `file:line` — TODO: description

**Existing** (N total):
- [Component A]: N items
- [Component B]: N items

### 🎯 Suggested Next Steps
1. **[High]** ...
2. **[Medium]** ...
3. **[Low]** ...

### 📊 Week Context
[If enough history, show this week's commit trend]
```

## Important Rules

- Keep it brief — this is a summary, not a detailed report
- Highlight what's NEW (today's changes, today's TODOs)
- Don't list all existing TODOs if there are many — group by component and show counts
- Next steps should be actionable and specific
- If no commits today, note that and focus on TODOs and doc sync
- Match the project's language (check CLAUDE.md)