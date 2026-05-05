# Session Template

> ใช้เป็น reference เมื่อจะสร้างบรรทัดของตัวเองใน `active-sessions.md`
>
> ดู `.ai-memory/rules.md` §13.1 + §13.6

---

## Row format ที่ append เข้า `active-sessions.md`

```markdown
| {session-id} | {role} | {HH:MM} | {HH:MM} | {task-id หรือ "-"} | {short scope} | active |
```

ตัวอย่าง:

```markdown
| lead-20260505-1430-a3f | lead | 14:30 | 14:30 | - | - | idle |
| backend-20260505-1435-7c2 | backend | 14:35 | 15:02 | T-20260505-001 | 3 files (mof-backend/src/feature/) | active |
| frontend-20260505-1500-b91 | frontend| 15:00 | 15:00 | T-20260505-002 | 1 file (mof-frontend/.../page.tsx) | active |
```

---

## Lifecycle

1. **เปิด session** → append บรรทัดใหม่ใน `active-sessions.md` + log `session.start` ใน `activity.log.md`
2. **ทุก checkpoint** (ทำงานเกิน 10 นาที) → update `last-heartbeat` + `working-files` ของบรรทัดตัวเอง
3. **เริ่ม task ใหม่ / สลับ task** → update `task` column
4. **เจอ blocker** → เปลี่ยน `status` เป็น `stalled` แล้ว log `task.blocked`
5. **ปิด session** → ลบบรรทัดของตัวเอง + log `session.end`

---

## ข้อห้าม

- ❌ ห้ามแก้บรรทัดของ session อื่น (ยกเว้น Lead mark `stalled` ตาม §13.6)
- ❌ ห้ามทิ้งบรรทัดตัวเองค้างหลังปิด session
- ❌ ห้ามใช้ session-id ซ้ำ — ถ้าเปิดใหม่ ต้องสร้าง id ใหม่
