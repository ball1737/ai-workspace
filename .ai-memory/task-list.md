# Project Task List — รายการ Task ระดับโปรเจค

> รวม task ที่กำลังทำ + รอทำ ระดับโปรเจค (ไม่ใช่ระดับ feature)
> **Lead agent อัพเดตไฟล์นี้** เมื่อแตก task หรือ task เปลี่ยนสถานะ

> Task ระดับ feature → `_docs/requirement/{feature}/task-list.md`
> Task ระดับ agent → `.ai-memory/agent/{role}/current-task.md`

---

## Active Tasks (กำลังทำ)

| ID             | Task                                        | Owner Agent | Status | Started    | Doc Ref                                      |
| -------------- | ------------------------------------------- | ----------- | ------ | ---------- | -------------------------------------------- |
| T-20260429-001 | Implement `mof-backend` type safety cleanup | Codex       | review | 2026-04-29 | `_docs/requirement/type-safety-any-cleanup/` |

---

## Pending Tasks (รอทำ)

_(ไม่มี task รอทำ)_

| ID  | Task | Reason | Priority | Note |
| --- | ---- | ------ | -------- | ---- |
| —   | —    | —      | —        | —    |

---

## Completed Tasks (ย้อนหลัง 10 รายการ)

_(ยังไม่มี task ที่เสร็จ)_

| ID  | Task | Owner | Completed | Note |
| --- | ---- | ----- | --------- | ---- |
| —   | —    | —     | —         | —    |

---

## Task ID Convention

รูปแบบ: `T-YYYYMMDD-NNN` เช่น `T-20260427-001`

- `YYYYMMDD` = วันที่สร้าง task
- `NNN` = ลำดับใน 1 วัน (001 ขึ้นไป)

---

## Status Values

- `pending` — รอเริ่ม
- `in-progress` — กำลังทำ
- `blocked` — ติดอยู่ รอเงื่อนไข
- `review` — เสร็จแล้วรอ review
- `done` — เสร็จสมบูรณ์
- `cancelled` — ยกเลิก
