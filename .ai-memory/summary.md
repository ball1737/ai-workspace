# Project Summary — สรุปภาพรวมของ Workspace

> สรุปสถานะภาพรวมของงานปัจจุบัน + ประวัติงานที่เพิ่งจบ
> **Lead agent อัพเดตไฟล์นี้** เมื่อจบแต่ละ checkpoint สำคัญ หรือจบแต่ละ feature

---

## Workspace Purpose

AI Space — ระบบจัดระเบียบการทำงานกับ AI agent หลายตัว แบ่งงานเป็นระยะสั้น / ระยะยาว ทำงานผ่าน workflow ที่กำหนดใน `.ai-memory/rules.md`

---

## Current Status

**สถานะ:** `type-safety-any-cleanup` implemented and ready for review

**Active Feature:** type-safety-any-cleanup — review

---

## Completed Features

_(ยังไม่มี feature ที่ทำเสร็จ)_

| Feature | Type | Completed | Doc Folder | Note |
| ------- | ---- | --------- | ---------- | ---- |
| —       | —    | —         | —          | —    |

---

## Workspace Architecture (Bird's-eye view)

- **Memory:** `.ai-memory/` (rules + state ของ AI)
- **Documents:** `_docs/requirement/` (PRD/SSD/test cases ของแต่ละ feature)
- **Subagents:** `.claude/agents/` (Claude Code subagent definitions)

7 specialist agents (+ Codex adaptive runner):

- **Coordinators:** Lead
- **Analysts:** SA, BA
- **Implementers:** Developer (full-stack — backend + frontend tracks ใน agent เดียว, merged ตั้งแต่ 2026-05-05)
- **Quality:** QA, DevOps, Code Reviewer
- **Adaptive:** Codex (intake + เลือก specialist role ที่เหมาะสม)

**Default dispatch mode:** in-session role switching (Lead "สวมหมวก" specialist ใน session เดียว) — ไม่ใช้ Agent tool dispatch sub-agent ยกเว้น exception case (ดู master rules §14)

---

## Key Decisions / History

| Date       | Decision                                                                                              | Reason                                                                                |
| ---------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| 2026-04-29 | Type safety cleanup scope is `mof-backend/src` only                                                   | Tests/scripts remain out of scope for first pass                                      |
| 2026-04-29 | One exported `ModelObject<Model>` type per Objection model class                                      | User clarified no Insert/Update/Patch/Delete model aliases                            |
| 2026-05-05 | Merge backend + frontend agents → `developer` (full-stack)                                            | ผู้ใช้รายงาน context ไม่ต่อเนื่องเมื่อแยก agent — รวมแล้วสลับ track ใน session เดียว |
| 2026-05-05 | Default dispatch mode = in-session role switching (master §14)                                        | ลด overhead + token waste จาก sub-agent fresh context                                 |
| 2026-05-05 | Files Read Memory rule (master §15)                                                                   | ลด re-read ซ้ำซ้อนภายใน session เดียวกัน                                              |
| 2026-05-05 | Reporting Format trigger-based (master §16)                                                           | format เต็มเฉพาะ trigger (close phase / user ขอ / handoff) ไม่ใช่ทุก task             |

---

## Note for AI agents

ไฟล์นี้เป็นภาพรวม **ระดับ workspace** ไม่ใช่ระดับ feature

- ภาพรวมระดับ feature → `_docs/requirement/{feature}/summary.md`
- งานที่กำลังทำจริง → `.ai-memory/current.md`
