# `.ai-memory/sessions/`

> Session registry สำหรับ multi-session coordination
>
> Spec เต็มอยู่ที่ `.ai-memory/rules.md` §13

---

## ไฟล์ในนี้

| ไฟล์                   | หน้าที่                                                        |
| ---------------------- | -------------------------------------------------------------- |
| `active-sessions.md`   | Live registry ของ session ที่เปิดอยู่ (ตารางหลัก)              |
| `_template.session.md` | Reference สำหรับ format row ที่ append ลง `active-sessions.md` |

---

## เมื่อใดต้องอ่านไฟล์ในนี้

- **Lead ก่อน dispatch** → poll `active-sessions.md` ดูว่า role นั้น active อยู่แล้วไหม กัน dispatch ซ้ำ
- **Lead ทุกๆ checkpoint** → ตรวจ heartbeat ของ worker ว่า stalled หรือไม่ (§13.6)
- **Worker เริ่ม session** → เพิ่มบรรทัดของตัวเอง
- **ทุก session ก่อนจบ** → ลบบรรทัดของตัวเอง

## เมื่อใดไม่ต้องอ่าน

- Worker ทำงาน inline (ไม่ใช่ multi-session) → ไม่จำเป็นต้องลงทะเบียนถ้าทำเสร็จใน 1 turn
- ถ้า inbox compiled context ครอบคลุมแล้ว → Worker ไม่ต้องอ่านไฟล์นี้
