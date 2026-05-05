# Developer Agent — Inbox

> **Purpose:** Multi-session coordination channel (per master rules §13.2)
> **Default mode:** in-session role switching → **inbox/outbox not used**
> **Used เมื่อ:** Lead spawn long-running worker session แยก หรือ user dispatch ผ่าน prompt-only mode

---

## Pending Tasks

_(empty)_

---

## Format

ถ้า Lead จะ compile task ลง inbox (multi-session use case):

```markdown
## Task {ID} — {short title}

**Compiled by:** Lead
**Compiled on:** {YYYY-MM-DD HH:MM}
**Active Track:** {backend | frontend | both}

### 4-section spec (per master §3.5.2)

1. **Agent role:** developer
2. **หัวข้อ:** {topic}
3. **รายละเอียดงาน:**
   - Step 1: ...
   - Step 2: ...
   - Context: ...
4. **สรุปสิ่งที่ทำ — Summary requirement:**
   เมื่อทำเสร็จ ให้รายงานกลับใน format ดังนี้:
   - ...

### Pre-locked files (Lead pre-locked)

- `path/to/file.ts`

### Files to read (compiled context — Worker ห้าม re-read ที่อื่นนอก list นี้)

- `_docs/requirement/{feature}/ssd.md` §X
- `_docs/requirement/{feature}/{backend|frontend}.md` §Y
```

---

## Lifecycle

1. Lead compile task → append entry above
2. Developer (worker session) อ่าน inbox + ทำงาน
3. Developer เขียน outbox + archive entry → `inbox-archive.md` (หรือลบ)
4. Lead poll outbox → ack
