# `.ai-memory/locks/`

> Lightweight file lock convention สำหรับ shared state files
>
> Spec เต็มอยู่ที่ `.ai-memory/rules.md` §13.5

---

## เมื่อใดต้อง lock

ก่อน Edit/Write **เฉพาะ** ไฟล์เหล่านี้:

- `.ai-memory/current.md`
- `.ai-memory/summary.md`
- `.ai-memory/task-list.md`
- `.ai-memory/sessions/active-sessions.md`
- `_docs/requirement/{feature}/summary.md`, `task-list.md`, `checklist.md`

## เมื่อใดไม่ต้อง lock

- ไฟล์ของ agent ตัวเอง (`agent/{self}/current-task.md`, `inbox.md`, `outbox.md`)
- `activity.log.md` (append-only มีกฏ atomic อยู่)
- Source code files (single agent dispatch ต่อ task)
- Worker session ที่ทำงานเสร็จใน 1 turn (lock overhead ไม่คุ้ม — ถ้าผู้ใช้ supervise อยู่)

---

## Filename format

`{escaped-path}.lock`

แทน `/` ด้วย `__`:

| Target file                        | Lock filename                              |
| ---------------------------------- | ------------------------------------------ |
| `.ai-memory/current.md`            | `_ai-memory__current.md.lock`              |
| `.ai-memory/summary.md`            | `_ai-memory__summary.md.lock`              |
| `_docs/requirement/foo/summary.md` | `_docs__requirement__foo__summary.md.lock` |

---

## เนื้อหาในไฟล์ lock

```markdown
session: {session-id}
agent: {role}
file: {target file path}
acquired: {ISO timestamp}
expires: {ISO timestamp + 30min}
purpose: {1-line reason}
```

ตัวอย่าง:

```markdown
session: backend-20260505-1435-7c2
agent: backend
file: .ai-memory/current.md
acquired: 2026-05-05T15:02:11Z
expires: 2026-05-05T15:32:11Z
purpose: update Active Feature status to Phase 2 in-progress
```

---

## Acquire / Release Workflow

1. **ก่อนแก้ shared state** → check ว่ามี `.lock` ไฟล์ค้างไหม
2. ถ้ามี **และยังไม่ expire** **และไม่ใช่ session ตัวเอง** → wait หรือ abort + รายงานผู้ใช้
3. ถ้ามี **แต่ expire แล้ว** → ถือว่า session เก่า dead, ลบ lock ของเก่า + acquire ใหม่ + log `lock.released` (reason=expired) ใน activity.log
4. ถ้าไม่มี → สร้าง lock + แก้ไฟล์ + ลบ lock + log `lock.acquired`/`lock.released`

---

## ข้อห้าม

- ❌ Lock ค้างเกิน 30 นาทีต่อครั้ง (`expires` ห้ามไกลกว่านี้)
- ❌ Acquire 5+ ไฟล์พร้อมกันแล้วทำงานยาวก่อน release
- ❌ ลบ lock ของ session อื่นที่ยังไม่ expire
- ❌ ปล่อย lock ค้างเมื่อจบ session — ต้อง release ก่อน `session.end`
