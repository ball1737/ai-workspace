# Activity Log (Append-only)

> **ทุก session/agent append บรรทัดท้ายเท่านั้น** — ห้ามแก้/ลบบรรทัดเก่า
>
> Spec ที่ `.ai-memory/rules.md` §13.4

---

## Format

```
{ISO timestamp} | {session-id} | {agent role} | {event} | {short note}
```

## Events

`session.start`, `session.end`, `dispatch.queued`, `dispatch.activated`, `task.completed`, `task.blocked`, `lock.acquired`, `lock.released`, `handoff.created`

## Token-efficient reading

อย่า Read ทั้งไฟล์ — ใช้ `tail -50` หรือ Read offset เพื่อดูเฉพาะ recent

---

## Log entries (newest at bottom)

<!-- append บรรทัดใหม่ใต้บรรทัดนี้เท่านั้น -->
