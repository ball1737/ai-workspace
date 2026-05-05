---
feature: {feature-name}
type: task-list
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
owner: Lead
---

# Task List — {Feature Name}

> รายการ task ของ feature นี้ — Lead แตก task และ assign agent
> Agent อัพเดต status ใน `.ai-memory/agent/{role}/current-task.md` ของตัวเอง แล้ว Lead sync มาที่นี่

---

## Task ID Convention

`{FEATURE-CODE}-NNN` เช่น `LOGIN-001`

## Status Values

- `pending` — รอเริ่ม
- `in-progress` — กำลังทำ
- `blocked` — ติดอยู่
- `review` — เสร็จรอ review
- `done` — เสร็จสมบูรณ์
- `cancelled` — ยกเลิก

---

## Phase 1: Analysis & Design

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| {F}-001 | Read code base + identify scope | SA | - | summary.md §3 | |
| {F}-002 | Write requirement.md | SA | - | requirement.md | |
| {F}-003 | Write prd.md | SA | - | prd.md | |
| {F}-004 | Write ssd.md | SA | - | ssd.md | |
| {F}-005 | Validate design vs requirement | BA | - | — | |

## Phase 2: Implementation

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| {F}-101 | Backend: {module/api} | Backend | - | ssd.md §4 | |
| {F}-102 | Frontend: {component/page} | Frontend | - | ssd.md §5 | |
| {F}-103 | DB migration | Backend | - | ssd.md §3 | |
| {F}-104 | DevOps: infra changes (if any) | DevOps | - | ssd.md §10 | |

## Phase 3: Quality

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| {F}-201 | Code review (backend) | Code Reviewer | - | — | |
| {F}-202 | Code review (frontend) | Code Reviewer | - | — | |
| {F}-203 | Write unit tests | QA + Devs | - | testcase.md §2 | |
| {F}-204 | Run unit tests | QA | - | testcase.md §6 | |
| {F}-205 | Manual / E2E test | QA | - | testcase.md §4-5 | |
| {F}-206 | Final BA validation | BA | - | — | |

## Phase 4: Closeout

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| {F}-301 | Update summary.md "Final Outcome" | Lead | - | summary.md §7 | |
| {F}-302 | Clear .ai-memory/current.md | Lead | - | — | |
| {F}-303 | Update .ai-memory/summary.md | Lead | - | — | |

---

## Blocked Tasks

_(none)_

| ID | Reason | Waiting on |
|----|--------|-----------|
|    |        |           |
