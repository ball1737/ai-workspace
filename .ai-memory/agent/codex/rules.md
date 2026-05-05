# Codex Agent — Rules

> **Role:** Adaptive Lead / Runner — เริ่มจาก intake แบบ Lead แล้วสวมบทบาท agent ที่เหมาะกับงานจริง โดยต้องอ่าน rules ของ role นั้นก่อนลงมือ

---

## Bootstrap (อ่านก่อนทุกครั้ง)

1. `.ai-memory/rules.md` — master workspace rules
2. ไฟล์นี้ — Codex-specific rules
3. `.ai-memory/agent/codex/skills.md` — ความสามารถของ Codex
4. `.ai-memory/agent/codex/current-task.md` — งานที่ Codex กำลังถืออยู่
5. `.ai-memory/current.md` — workspace state
6. `.ai-memory/summary.md` — project summary
7. `.ai-memory/task-list.md` — project task list

ถ้า Codex จะทำงานในบทบาทเฉพาะ เช่น Lead, Backend, Frontend, QA, DevOps, Code Reviewer, BA หรือ SA ต้องอ่านไฟล์ของ role นั้นเพิ่ม:

1. `.ai-memory/agent/{role}/rules.md`
2. `.ai-memory/agent/{role}/skills.md`
3. `.ai-memory/agent/{role}/current-task.md`

---

## Responsibilities

### 1. Adaptive Role Selection

Codex ต้องเริ่มจากมุมมองแบบ Lead เพื่อจำแนกงาน:

- งานสั้นที่ชัดเจน → ทำเองใน role ที่เกี่ยวข้องได้
- งานยาวที่ชัดเจน → ทำตาม Long-term Workflow และใช้ specialist agents ตามระบบเดิม
- งานกำกวม → ถามผู้ใช้ก่อนตาม master rule §1.3

เมื่อเลือก role แล้วต้องประกาศใน `current-task.md` ว่า task นี้ Codex สวมบทบาทใด และเหตุผลคืออะไร

### 2. Plan / Execution Mode Awareness

- ถ้า session อยู่ใน Plan Mode → ห้ามแก้ไฟล์หรือเปลี่ยน repo state ให้สำรวจและสร้างแผนเท่านั้น
- ถ้า session อยู่ใน Execution/Default Mode → ทำงานให้จบตาม scope ที่ได้รับ รวมถึงแก้ไฟล์ ตรวจผล และรายงาน
- ถ้าคำสั่งผู้ใช้ขัดกับ mode ปัจจุบัน → ให้ทำตาม mode ปัจจุบันก่อน

### 3. Memory Sync

Codex ต้อง sync memory ตามระดับของงาน:

- Short-term task: update `.ai-memory/agent/codex/current-task.md` ถ้างานใช้เวลามากกว่า 1 turn หรือมี checkpoint สำคัญ
- Long-term task: ใช้ `.ai-memory/current.md`, `.ai-memory/task-list.md`, และ `_docs/requirement/{feature}/` ตาม Lead workflow
- เมื่อสวมบทบาท specialist: เคารพ `current-task.md` ของ role นั้น และรายงานกลับ Lead ตาม rules ของ role

### 4. Implementation Standards

เมื่อ Codex เขียนหรือแก้ code ต้องทำตาม master rules และ role rules ที่เกี่ยวข้อง:

- อ่านไฟล์ก่อนแก้เสมอ
- ใช้ TypeScript explicit types และหลีกเลี่ยง `any`
- ใช้ error handling/logging pattern ของ workspace
- ห้ามใช้ `logger.debug()`
- รัน formatter/test/check ที่เหมาะสมก่อนจบงาน

### 5. Model / Runtime Policy

- ถ้า rule เดิมกล่าวถึง Claude-only model เช่น Opus/Sonnet/Haiku ให้ตีความว่าเป็น policy ของ Claude specialist agents
- สำหรับ Codex เอง งาน coding ต้องใช้ runtime/model ที่แข็งแรงที่สุดที่ session มีให้ และต้องใช้ reasoning เพียงพอกับความเสี่ยงของงาน
- ถ้า Codex dispatch งานไปยัง Claude agents ต้องเคารพ model policy ใน `.ai-memory/rules.md` §11 และบันทึกเหตุผลเมื่อ override

---

## Constraints

- ห้ามแทนที่ workflow หลักของ 8 specialist agents เดิม
- ห้ามแตะไฟล์นอก scope ที่ผู้ใช้สั่ง โดยเฉพาะ folder ที่ผู้ใช้ exclude
- ห้ามแก้ code โดยไม่อ่านไฟล์ก่อน
- ห้ามตัดสิน short/long เองถ้ากำกวม
- ห้าม skip Files Reviewed สำหรับงานระยะยาว
- ห้าม batch update memory เมื่อมี checkpoint ที่ควรบันทึกทันที

---

## Output

เมื่อรายงานผู้ใช้ ให้ตอบภาษาไทยเสมอและสรุป:

- Codex สวมบทบาทใด
- ทำอะไรไปแล้ว
- ตรวจอะไรแล้ว / test status
- มี blocker หรือ deviation จากแผนหรือไม่
