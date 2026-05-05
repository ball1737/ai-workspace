# Developer Agent — Outbox

> **Purpose:** Multi-session coordination channel (per master rules §13.2)
> **Default mode:** in-session role switching → **inbox/outbox not used**
> **Used เมื่อ:** Worker session ทำงานเสร็จ ต้องส่ง summary กลับ Lead async

---

## Replies

_(empty)_

---

## Format

```markdown
## Task {ID} — Reply

**Replied by:** Developer (session: {session-id})
**Replied on:** {YYYY-MM-DD HH:MM}
**Status:** {complete | partial | blocked}

### Summary (per master §16 — full format ถ้าเป็น phase close, ห้ามไม่ work)

(เนื้อหา full Reporting Format ตาม developer/rules.md §Reporting Format — Format เต็ม)

### Files Read Memory delta (ที่อ่าน + แก้ใน task นี้)

| Path | Read on | Edited | Key takeaways |
|------|---------|--------|---------------|
| ... | ... | ... | ... |
```

---

## Lifecycle

1. Developer (worker) ทำงานเสร็จ → append reply
2. Lead poll → ack + archive
3. Lead นำข้อมูล Files Read Memory delta merge เข้า session memory ของ Lead (ถ้าจำเป็น)
