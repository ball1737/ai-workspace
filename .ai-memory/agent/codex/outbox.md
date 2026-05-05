# codex Agent — Outbox

> **Worker เขียนผลลัพธ์ + status. Lead อ่าน + ack + archive.**
> **Lead ห้ามแก้ outbox** — แก้ inbox/current.md/summary.md แทน
>
> Spec: `.ai-memory/rules.md` §13.3

---

## Format ของแต่ละ reply (ตอน Worker append)

```markdown
## [{TASK-ID}] — completed: {YYYY-MM-DD HH:MM}

**Worker session:** {session-id}
**Result:** done | partial | blocked

### Summary (ตาม format ที่ Lead ขอใน inbox)

{fill in}

### Files changed

- `path/a.ts` — {what}

### Blockers / questions

{ถ้ามี — Lead จะหยิบไปถามผู้ใช้}

### Next steps suggested

{ถ้ามี}
```

---

## Replies (newest at bottom)

<!-- Worker append reply ใต้บรรทัดนี้ -->

_(ยังไม่มี outbound)_
