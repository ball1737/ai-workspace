---
feature: type-safety-any-cleanup
type: task-list
created: 2026-04-29
updated: 2026-04-29
owner: Lead
---

# Task List — Type Safety Any Cleanup

## Task ID Convention

`TYPE-SAFETY-NNN`

## Phase 1: Analysis & Design

| ID              | Task                              | Owner | Status | Doc Ref                                       | Updated    |
| --------------- | --------------------------------- | ----- | ------ | --------------------------------------------- | ---------- |
| TYPE-SAFETY-001 | Read code base + identify scope   | SA    | done   | summary.md §3                                 | 2026-04-29 |
| TYPE-SAFETY-002 | Write requirement/prd/ssd/summary | SA    | done   | requirement.md / prd.md / ssd.md / summary.md | 2026-04-29 |

## Phase 2: Implementation

| ID              | Task                                                          | Owner   | Status   | Doc Ref     | Updated    |
| --------------- | ------------------------------------------------------------- | ------- | -------- | ----------- | ---------- |
| TYPE-SAFETY-101 | Add one exported type per model class                         | Backend | done     | ssd.md §3.3 | 2026-04-29 |
| TYPE-SAFETY-102 | Remove explicit any from shared typings/utils/middlewares/api | Backend | deferred | ssd.md §5   | 2026-04-29 |
| TYPE-SAFETY-103 | Remove explicit any from module repositories/controllers      | Backend | deferred | ssd.md §5   | 2026-04-29 |
| TYPE-SAFETY-104 | Enable no-explicit-any ESLint gate as warning                 | Backend | done     | ssd.md §5   | 2026-04-29 |

## Phase 3: Quality

| ID              | Task                                                 | Owner | Status     | Doc Ref        | Updated    |
| --------------- | ---------------------------------------------------- | ----- | ---------- | -------------- | ---------- |
| TYPE-SAFETY-201 | Run lint                                             | QA    | done       | testcase.md §6 | 2026-04-29 |
| TYPE-SAFETY-202 | Run typecheck                                        | QA    | done       | testcase.md §6 | 2026-04-29 |
| TYPE-SAFETY-203 | Run targeted/full tests and report baseline failures | QA    | superseded | testcase.md §6 | 2026-04-29 |

## Blocked Tasks

_(none)_
