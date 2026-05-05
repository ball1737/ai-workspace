# Current Focus — ปัจจุบันกำลังทำอะไรอยู่

> **อ่านไฟล์นี้เป็นอันดับแรกเมื่อเริ่ม session ใหม่** เพื่อให้รู้ว่าสถานะงานล่าสุดอยู่ตรงไหน
> **Lead agent อัพเดตไฟล์นี้** เมื่อเริ่ม / สลับ / จบ feature

---

## Active Feature

**Feature:** feature-management

**Type:** long-term

**Status:** Phase 1 complete (Schema Foundation deployed on dev). Phase 2 paused — waiting user instruction

**Started:** 2026-05-04

**Owner Agents:**

- Lead: Claude (main thread)
- SA: Claude (done — analysis docs ready 2026-05-04)
- Backend: Claude (Backend subagent — Opus 4.7) — about to dispatch for Phase 1
- Frontend: _(pending — Phase 2)_
- QA: _(pending — Phase 4)_

**Documents:**

- Requirement folder: `document/requirement/feature-management/`
- All decision docs ready: requirement.md, prd.md, ssd.md, summary.md, task-list.md, testcase.md, cr.md
- All engineering playbooks ready: backend.md, frontend.md, migration.md, checklist.md
- Open Questions §9: ทุกข้อ resolved 2026-05-04 (Q1=C, Q2=A, Q3=A, Q4=B, Q5=resolved by code review)

---

## Active Task (ที่กำลังทำจริงในขณะนี้)

**Phase 1 — Schema Foundation: COMPLETE 2026-05-04** — 8 migrations + 7 models deployed on dev DB (`hwv4_data4`); all 6 verification queries passed (14 features, 7 packages, 9 addons, 98 cartesian package_features, FK feature_id=RESTRICT confirmed, composite UNIQUE present, M8 orphan_count=0). 17 rows migrated from comp_companies.add_ons across 7 companies.

**Awaiting user instruction**: Phase 2 (Master CRUD Backend + Frontend) was deferred per user direction — do not start until user gives go-ahead.

---

## Paused / Waiting

| Feature                  | Status | Reason                                                                |
| ------------------------ | ------ | --------------------------------------------------------------------- |
| type-safety-any-cleanup  | review | งาน implementation เสร็จแล้ว 2026-04-29 รอ final review/closeout      |

---

## Recent Activity (ย้อนหลัง 5 รายการ)

_(จะถูกอัพเดตอัตโนมัติโดย Lead เมื่อมี activity)_

| Date       | Agent  | Action                                                                                                |
| ---------- | ------ | ----------------------------------------------------------------------------------------------------- |
| 2026-05-04 | DevOps | Phase 1 P1.9 complete: M1-M8 applied on dev (batches 31-33), all 6 verification queries passed       |
| 2026-05-04 | Backend | Hotfix M6/M7 duplicate company_id via Option A (table.foreign + raw ALTER NOT NULL); M8 verified safe |
| 2026-05-04 | DevOps | Migrate retry cycle: pre-marked 2 drift items, applied 30 unrelated migrations (batch 32), failed M6 |
| 2026-05-04 | Claude | Switched active feature to feature-management; Phase 1 dispatch incoming                              |
| 2026-05-04 | Claude | Resolved all 5 Open Questions in requirement.md §9 (Q1=C, Q2=A, Q3=A, Q4=B, Q5=code-resolved)         |
| 2026-05-04 | Claude | Updated migration.md M3+M6 + ssd.md §3.1.6+§6.2+§10 to reflect Q1-Q4 decisions                        |
| 2026-05-04 | Claude | Added rules.md §1.4 — Question Batching Protocol                                                      |
| 2026-04-29 | Codex  | Scoped implementation back to model type exports only; no-explicit-any lint set to warning            |

---

## Note for AI agents reading this

หาก section "Active Feature" ว่าง → workspace ยังไม่มีงาน active กำลังทำอยู่

- ถ้าผู้ใช้สั่งงานใหม่ → Lead ตัดสินประเภทงานก่อน (ดู `.ai-memory/rules.md` ข้อ 1)
- ถ้าผู้ใช้ถาม "ทำอะไรอยู่" → ตอบว่ายังไม่มีงาน active และพร้อมรับงานใหม่
