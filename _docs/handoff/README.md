# `_docs/handoff/`

> Handoff files สำหรับ multi-session sequential continuity (context หมด → resume ใน session ใหม่)
>
> Spec เต็มที่ `.ai-memory/rules.md` §7.3 (Handoff Protocol) + §13.9 (Resume vs Handoff)

---

## เมื่อใดต้องสร้าง handoff

- Context window > 80% เต็ม + ยังต้องทำต่อใน session ถัดไป
- ก่อนปิด session ที่มี outbox/inbox ค้าง unack
- Lead/Worker จบ phase ใหญ่ + ต้องการ checkpoint ที่อ่านง่ายสำหรับ session ถัดไป

## เมื่อใดไม่ต้องสร้าง

- Task เสร็จในรอบเดียว + ผู้ใช้ supervise อยู่ → outbox.md เพียงพอ
- Session สั้น + เปลี่ยนแปลงน้อย → update `current.md` พอ

---

## Naming Convention

`{YYYY-MM-DD}-{topic}.handoff.md`

ตัวอย่าง:

- `2026-05-05-feature-management-phase2.handoff.md`
- `2026-05-05-type-safety-cleanup-review.handoff.md`

---

## ไฟล์ในนี้

| ไฟล์                        | หน้าที่                                              |
| --------------------------- | ---------------------------------------------------- |
| `_template.handoff.md`      | Template สำหรับสร้าง handoff ใหม่                    |
| `{date}-{topic}.handoff.md` | Handoff ที่สร้างขึ้นจริง — เรียงตามวันที่ใน filename |

---

## Token efficiency rules

- **เขียนเฉพาะ delta** — สิ่งที่เปลี่ยนใน session ที่จบ ไม่ใช่ snapshot ของ workspace ทั้งก้อน
- **อ้าง file path** สำหรับ state ที่อ่านได้จากไฟล์อื่น (`current.md`, `summary.md`, SSD docs) — ห้ามลอกเนื้อ
- **Pending inbox entries เป็น link ไปยัง inbox/outbox ของแต่ละ agent** — ไม่ต้องลอกเนื้อ inbox มาทั้งก้อน
- Resume command บอกชัดว่า session ใหม่ **อ่านเฉพาะ** ไฟล์ใด — กัน bootstrap ทั้ง memory โดยไม่จำเป็น

---

## Lifecycle

1. **สร้าง handoff** → copy จาก `_template.handoff.md` → fill in delta + pending inbox + cleanup checklist
2. **Cleanup checklist** → ลบบรรทัด session ออกจาก `active-sessions.md`, release lock, log `session.end` + `handoff.created`
3. **Resume ใน session ถัดไป** → ผู้ใช้ส่ง resume command → agent อ่าน handoff + inbox/outbox ที่ระบุ + ไฟล์ของตัวเอง (rules/skills/current-task) เท่านั้น
4. **Archive** → handoff ที่เก่าเกิน 30 วันย้ายไป `_docs/handoff/_archive/` หรือลบทิ้งถ้าไม่ใช้แล้ว
