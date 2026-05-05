# devops Agent — Inbox

> **Lead เขียนเข้ามา. Worker อ่าน + ทำตาม.**
> **Worker ห้ามแก้ inbox** — ตอบกลับใน `outbox.md`
>
> Spec: `.ai-memory/rules.md` §13.3

---

## Format ของแต่ละ task entry (ตอน Lead append)

```markdown
## [{TASK-ID}] {หัวข้องาน} — assigned: {YYYY-MM-DD HH:MM}

**Status:** queued | in-progress | done | rejected
**Lead session:** {session-id}
**Priority:** P1 | P2 | P3

### Compiled Context (Lead รวบมาให้ — Worker ไม่ต้องอ่าน workspace ซ้ำ)

- Feature: {ชื่อ feature}
- Relevant files: `path/a.ts`, `path/b.ts` (อ่านแค่นี้พอ)
- Key decisions: {bullet 1-3 ข้อ}
- Constraints: {bullet}

### Task (4-section ตาม §3.5.2)

1. Agent role: devops
2. หัวข้อ: {topic}
3. รายละเอียด: {steps}
4. Summary requirement: {format ที่ต้องรายงานกลับใน outbox}

### Files Locked by Lead for this task

- `path/x.md` (Lead pre-locked → Worker ไม่ต้อง lock ซ้ำ)
```

---

## Active entries (newest at bottom)

<!-- Lead append task entry ใต้บรรทัดนี้ -->

_(ยังไม่มี task)_
