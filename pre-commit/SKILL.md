---
name: pre-commit
description: "커밋 전 종합 점검 - 코드 리뷰 + 영향도 분석 + 문서 동기화를 한번에 실행"
user_invocable: true
argument: "없음 (staged changes 자동 분석)"
---

You are a pre-commit gate that performs a comprehensive check before code is committed. Run all checks sequentially and produce a unified report.

You MUST complete ALL three phases before producing the final report. Do not skip any phase.

## Phase 1: Code Review

1. Run `git diff --staged` via Bash to get the staged changes
2. If no staged changes, run `git diff` for unstaged changes instead
3. Review the diff for:
   - **Bugs**: Logic errors, null access, resource leaks, error handling
   - **Security**: Injection, hardcoded secrets, buffer overflows
   - **Performance**: Unnecessary allocations, O(n²), blocking I/O
   - **Readability**: Unclear naming, dead code, inconsistent style
4. Classify each finding: Critical / Warning / Info
5. Record results for the final report

## Phase 2: Impact Analysis

1. From the staged diff, extract:
   - Modified function signatures
   - Changed struct/class definitions
   - Modified enum values
   - Changed export symbols
2. For each modified symbol:
   - Use Grep to find all references/callers
   - Check if struct sizeof changed
   - Check if vtable order changed
   - Check if export ABI is affected
3. Classify: Breaking / Behavioral / Safe
4. Record results for the final report

## Phase 3: Document Sync Check

1. Identify which files were modified in the diff
2. Search for references to those files/functions in:
   - CLAUDE.md
   - Memory files (if project memory directory exists)
   - Doc files (*.md in docs/ or similar)
3. Check if any documented values now conflict with the code changes
4. Flag documents that need updating
5. Record results for the final report

## Final Report

After ALL three phases are complete, output:

```markdown
## 🔍 Pre-commit Report

**Branch**: [current branch]
**Staged files**: N files changed
**Date**: YYYY-MM-DD HH:MM

### Results

| Phase | Result | Issues |
|-------|--------|--------|
| Code Review | ✅ PASS / ⚠️ WARN / ❌ FAIL | N critical, N warning |
| Impact Analysis | ✅ PASS / ⚠️ WARN / ❌ FAIL | N breaking, N behavioral |
| Doc Sync | ✅ PASS / ⚠️ WARN / ❌ FAIL | N mismatches |

### 📋 Detail

#### Code Review Findings
[List findings with severity, or "No issues found"]

#### Impact Analysis
[List breaking/behavioral changes, or "No ABI impact"]

#### Document Sync
[List documents needing update, or "All docs in sync"]

### 🎯 Verdict

**[COMMIT / COMMIT WITH CAUTION / DO NOT COMMIT]**

- COMMIT: No critical issues, no breaking changes, docs in sync
- COMMIT WITH CAUTION: Warnings present but no blockers
- DO NOT COMMIT: Critical bugs, ABI breaks, or security issues found

[If COMMIT WITH CAUTION or DO NOT COMMIT, list what needs to be fixed]
```

## Important Rules

- Complete ALL three phases — never skip one even if earlier phases found issues
- The verdict must be based on the WORST result across all phases
- Critical code review finding OR breaking ABI change → DO NOT COMMIT
- Be practical — don't block commits for Info-level style issues
- If there are no staged changes at all, inform the user and exit