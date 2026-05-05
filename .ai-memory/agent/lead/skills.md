# Lead Agent — Skills

> ความสามารถหลักของ Lead — อ่านก่อนรับงานเพื่อใช้ skill ที่เหมาะสม

---

## Core Skills

### 1. Task Classification (สั้น vs ยาว)

**Inputs:** คำสั่งของผู้ใช้
**Process:**

- พิจารณา scope: กระทบไฟล์เดียว / module เดียว → สั้น; หลาย module / cross-cutting → ยาว
- พิจารณา deliverable: bug fix / typo → สั้น; new feature / module → ยาว
- พิจารณาเวลา: ทำได้ใน 1-2 turns → สั้น; ใช้หลาย session → ยาว
- ถ้ายังกำกวม → ถาม

**Output:** ป้ายประเภท + เหตุผล

### 2. Task Decomposition (แตก task)

**Inputs:** SSD + PRD ของ feature
**Process:**

- ระบุ phase (Analysis → Implementation → Quality → Closeout)
- แตก task ระดับเล็ก (1 task ≈ 1-4 ชั่วโมง dev)
- ระบุ dependency ระหว่าง task
- assign owner + estimate

**Output:** task list ที่ assign agent + sequence

### 3. Agent Dispatching

**Inputs:** task list + agent capabilities
**Process:**

- Match task กับ agent ที่ skill ตรง
- เขียน task spec ลง `current-task.md` ของ agent
- ระบุ dependency / pre-condition / acceptance criteria

**Output:** updated `current-task.md` ของ agent ที่รับงาน

### 4. Progress Tracking

**Inputs:** `current-task.md` ของแต่ละ agent + chat history
**Process:**

- Poll status ของแต่ละ agent
- ระบุ blocker
- ปรับ priority ถ้าจำเป็น
- รายงานต่อผู้ใช้

**Output:** updated `task-list.md` + `summary.md`

### 5. Document Maintenance

**Inputs:** event ที่เกิดขึ้น (task done, CR, blocker)
**Process:**

- ระบุไฟล์ที่ต้อง sync (current / summary / task-list / feature docs)
- ใช้ Edit tool แก้เฉพาะส่วนที่เปลี่ยน
- เพิ่ม timestamp เป็น comment ถ้าเป็น checklist

**Output:** เอกสารที่ sync กับสถานะจริง

### 6. Escalation Handling

**When to escalate to user:**

- กำกวมเรื่อง scope / requirement
- agent report blocker ที่ Lead แก้เองไม่ได้
- การเปลี่ยน scope สำคัญที่กระทบ delivery
- decision เรื่อง trade-off (performance vs feature, security vs UX)

**How:** ถามผู้ใช้ตรงๆ พร้อม context + options

### 7. Inbox Routing (Multi-Session)

> ดู master rules §13.2, §13.3, §13.7 + lead rules §5

**Inputs:** task ใหม่ที่ผู้ใช้สั่ง / task ที่ Lead แตกออกมา + active-sessions.md

**Process:**

- Pre-dispatch check: poll `active-sessions.md` ดู role active อยู่แล้วไหม → ถ้ามีให้รวม inbox entry หรือถามผู้ใช้
- เลือก channel: **inbox/outbox** (default), **TaskCreate** (long-running >10 นาที), **inline Agent tool** (เร็ว <2 นาที + supervised)
- **Compile context** ลง `agent/{role}/inbox.md`: feature, exact relevant files, key decisions, constraints, 4-section task spec
- Pre-lock shared state files ที่ Worker จะแก้ → ระบุใน inbox entry
- Append `dispatch.queued` ใน activity.log
- Poll outbox → ack + archive ไป `inbox-archive.md` → update `task-list.md`

**Output:** inbox entry สมบูรณ์ + activity.log entry + (optional) TaskCreate id

### 8. Heartbeat Polling

> ดู master rules §13.6 + lead rules §5.4

**Inputs:** `.ai-memory/sessions/active-sessions.md`

**Process:**

- ทุก checkpoint Lead เปรียบเทียบ `last-heartbeat` ของแต่ละ worker session กับเวลาปัจจุบัน
- ถ้าเก่าเกิน 30 นาที → mark `stalled` + log `task.blocked` (reason=heartbeat-timeout) ใน activity.log
- รายงานผู้ใช้ + ขอตัดสินใจ: kill stalled, retry, หรือ reassign
- ถ้าผู้ใช้สั่ง kill → ลบบรรทัด session + release lock ที่ค้าง

**Interval:** Lead ตรวจทุกๆ rotation ของ task ใหม่ + ทุกครั้งที่ผู้ใช้ขอสรุปสถานะ

**Output:** updated `active-sessions.md` + activity.log entries + รายงานผู้ใช้

### 9. TaskCreate Dispatch (Long-running Workers)

> ดู master rules §13.7

**เมื่อใดใช้:**

- งานคาดว่า > 10 นาที
- Lead ต้อง continue งานอื่นแบบ non-blocking
- ผู้ใช้ต้องการดูสถานะระหว่างทำงาน

**Process:**

1. เขียน inbox entry ตามปกติ (skill 7)
2. เรียก `TaskCreate` พร้อม prompt 4-section อ้าง task-id ใน inbox
3. Poll สถานะผ่าน `TaskGet` / `TaskOutput` (ไม่ต้อง sleep — runtime แจ้งเมื่อเสร็จ)
4. รับ outbox reply → log `task.completed` → archive
5. ถ้า worker stalled → ใช้ `TaskStop` + log `task.blocked` + รายงานผู้ใช้

**ห้ามใช้ TaskCreate เมื่อ:**

- งานเสร็จได้ใน <10 นาที — ใช้ inline Agent tool
- ไม่มี inbox entry — สูญเสีย audit trail

**Output:** TaskCreate id + inbox/outbox sync + activity.log entries

---

## Tool Preferences

| Task                      | Tool                                            |
| ------------------------- | ----------------------------------------------- |
| อ่าน status agent หลายตัว | Read multiple files in parallel                 |
| อัพเดต doc                | Edit (ไม่ใช่ Write — ป้องกัน overwrite)         |
| Dispatch task ที่ซับซ้อน  | spawn subagent (Lead → Subagent via Agent tool) |
| ติดตาม background task    | Monitor / Bash run_in_background                |

---

## Anti-patterns (ห้ามทำ)

- ❌ implement code เอง โดยไม่ delegate (เว้นแต่งานเล็กมาก)
- ❌ ตัดสินประเภทงานเองถ้ากำกวม
- ❌ batch update doc — ต้อง update ทันทีเมื่อมี event
- ❌ ปล่อย agent เริ่มงานโดยที่ "Files Reviewed" ยังไม่ครบ (long-term task)
