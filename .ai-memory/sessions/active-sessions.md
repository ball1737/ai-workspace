# Active Sessions

> **Live registry** ของ session ที่เปิดอยู่ — ทุก Lead/Worker session **ต้องลงทะเบียน** ที่นี่ก่อนเริ่มงาน และลบบรรทัดของตัวเองเมื่อจบ
>
> ดู `.ai-memory/rules.md` §13.1 (Session Identity) + §13.6 (Heartbeat) สำหรับ protocol

---

## Currently Active

| session-id | role | started | last-heartbeat | task | working-files | status |
| ---------- | ---- | ------- | -------------- | ---- | ------------- | ------ |
| _(empty)_  | —    | —       | —              | —    | —             | —      |

---

## Status Values

- `active` — กำลังทำงาน (heartbeat ภายใน 30 นาที)
- `idle` — เปิดอยู่แต่รอผู้ใช้สั่ง (Lead รอคำสั่ง)
- `stalled` — heartbeat เก่าเกิน 30 นาที (Lead mark + log `task.blocked`)
- `closing` — กำลังเขียน handoff / outbox สุดท้าย ก่อนจบ session

## Session ID Format

`{role}-{YYYYMMDD}-{HHMM}-{short-token}`

- `role` = lead, sa, ba, backend, frontend, qa, code-reviewer, devops, codex
- `YYYYMMDD-HHMM` = เวลาที่เปิด session (24h)
- `short-token` = 3-char random hex เช่น `a3f`, `7c2`

ตัวอย่าง: `lead-20260505-1430-a3f`, `backend-20260505-1435-7c2`

## Working-files Convention

- ใส่ count + scope สั้นๆ เช่น `3 files (mof-backend/src/modules/feature/)` — ไม่ต้อง list ครบ
- ถ้าแก้ shared state file (`current.md`, `summary.md`, ฯลฯ) → ต้อง acquire lock ก่อน (§13.5)

---

## เมื่อจบ session

1. Append outbox / handoff ตาม use case
2. Log `session.end` ใน `activity.log.md`
3. **ลบบรรทัดของตัวเองออกจากตารางนี้**

> ห้ามทิ้งบรรทัด session ตัวเองไว้หลังปิด — ทำให้ Lead เข้าใจผิดว่ายัง active
