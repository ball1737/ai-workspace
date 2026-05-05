# Archived Agents

> ไฟล์ที่ deprecated แล้ว — เก็บไว้สำหรับ reference / rollback / audit เท่านั้น
> **ห้าม reference จาก rules อื่นที่ active** — ถ้าต้องดู ให้อ่านแบบ informational

---

## Archived on 2026-05-05

### `backend/` (5 files: rules.md, skills.md, current-task.md, inbox.md, outbox.md)

**เหตุผล:** Merged into `developer/` agent (full-stack). กฏและ skills ถูกย้ายไปรวมใน:
- `.ai-memory/agent/developer/rules.md`
- `.ai-memory/agent/developer/skills.md`

History ของ tasks ที่ปิดแล้วใน Phase 2 ของ feature-management ถูก migrate ไป `.ai-memory/agent/developer/current-task.md §Closed Tasks (this feature)`.

### `frontend/` (5 files)

**เหตุผล:** Merged into `developer/` agent. (เหมือน backend ด้านบน)

---

## Rollback (ถ้าต้องการ revert การ merge)

```bash
mv .ai-memory/agent/_archive/backend .ai-memory/agent/backend
mv .ai-memory/agent/_archive/frontend .ai-memory/agent/frontend
mv .claude/agents/_archive/backend.md .claude/agents/backend.md
mv .claude/agents/_archive/frontend.md .claude/agents/frontend.md

# Then revert master rules + lead rules
git checkout HEAD -- .ai-memory/rules.md .ai-memory/agent/lead/rules.md .ai-memory/agent/codex/rules.md
```

แล้วลบ `.ai-memory/agent/developer/` + `.claude/agents/developer.md` ที่ใหม่ออก

---

## Notes

- ไฟล์ `backend/current-task.md` + `frontend/current-task.md` มีรายละเอียด task ของ feature-management Phase 2 อย่างละเอียด — ใน developer agent มี summary แต่ไม่ครบทุก field. ถ้าต้องการ audit precise → อ่านไฟล์ใน archive
- ไฟล์ inbox / outbox ใน archive น่าจะว่างหรือมี leftover ของ multi-session run เก่า — ไม่จำเป็นต้องอ่าน
