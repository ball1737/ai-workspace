# Handoff: {topic}

**Date:** {YYYY-MM-DD}
**Session:** {session-id} → next session
**Active feature:** {feature name หรือ "-" ถ้าไม่มี}

> **Token-efficient resume:** session ใหม่ **อ่านเฉพาะ** ไฟล์นี้ + outbox/inbox ที่ระบุใน "Pending inbox entries" — ไม่ต้อง bootstrap ครบ ถ้า handoff นี้ครอบคลุม

---

## Delta from previous handoff

> **เขียนเฉพาะสิ่งที่เปลี่ยนใน session นี้** — ห้ามลอก snapshot จาก `current.md` / `summary.md` ทั้งก้อน (อ้างไฟล์เอา)

### ✅ Done

- {bullet — สั้นๆ ระบุไฟล์/task ที่เสร็จ}

### 🔄 In-progress

- {bullet — ระบุ task-id + ค้างที่ขั้นตอนใด}

### ⬚ Not started (planned next)

- {bullet — task ที่รอเริ่ม}

---

## Files changed in this session

- `path/file.ts` — {what + why สั้นๆ}

---

## Open decisions / blockers

- {bullet พร้อม context สั้นๆ + ตัวเลือกที่พิจารณา (ถ้ามี)}

---

## Pending inbox entries (session ใหม่ต้องสานต่อ)

| Channel                             | Task ID        | Status | Note                              |
| ----------------------------------- | -------------- | ------ | --------------------------------- |
| `.ai-memory/agent/{role}/inbox.md`  | T-YYYYMMDD-NNN | queued | {1-line context}                  |
| `.ai-memory/agent/{role}/outbox.md` | T-YYYYMMDD-NNN | unack  | {1-line summary ของ worker reply} |

---

## Session manifest cleanup

- [ ] ลบบรรทัดของ session นี้ออกจาก `.ai-memory/sessions/active-sessions.md`
- [ ] Release lock ทุกตัวที่ค้าง (ถ้ามี) ใน `.ai-memory/locks/`
- [ ] Log `session.end` + `handoff.created` ใน `.ai-memory/activity.log.md`

---

## Resume command (สำหรับ session ถัดไป)

> "ทำต่อจาก handoff: `_docs/handoff/{filename}.handoff.md` — อ่านเฉพาะ handoff + inbox/outbox ที่ระบุ ไม่ต้อง bootstrap ครบ"
