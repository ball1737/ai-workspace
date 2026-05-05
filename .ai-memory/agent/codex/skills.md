# Codex Agent — Skills

> ความสามารถหลักของ Codex ใน workspace นี้ — ใช้เป็น adaptive runner ที่อ่านกฏกลางก่อน แล้วเลือก role ตามงานจริง

---

## Core Skills

### 1. Task Classification

**Inputs:** คำสั่งผู้ใช้ + workspace state

**Process:**

- อ่าน `.ai-memory/current.md` และ `.ai-memory/task-list.md`
- ตัดสิน short-term / long-term ตาม master rules §1
- ถ้ากำกวม ให้ถามผู้ใช้ก่อน ไม่เดา

**Output:** ประเภทงาน, role ที่ Codex จะสวม, และเหตุผล

### 2. Adaptive Role Execution

**Inputs:** ประเภทงาน + scope

**Process:**

- เลือก role ที่เหมาะสม เช่น Lead, Backend, Frontend, QA, DevOps, Code Reviewer, BA, SA
- อ่าน `.ai-memory/agent/{role}/rules.md`, `skills.md`, `current-task.md`
- ทำงานตาม constraints ของ role นั้น

**Output:** งานที่เสร็จพร้อมรายงานใน format ที่ role กำหนด

### 3. Repository Exploration

**Inputs:** path / module / feature ที่เกี่ยวข้อง

**Process:**

- ใช้ `rg` / `rg --files` ก่อนเสมอสำหรับงานสั้น
- อ่านเฉพาะ dependency chain ที่จำเป็น
- สำหรับงานยาว ให้ทำตาม SA code base reading rule และบันทึก Files Reviewed
- เคารพ `.claudeignore` และ `.codexignore`

**Output:** context ที่พอสำหรับตัดสินใจหรือแก้ไข โดยไม่อ่านเกินจำเป็น

### 4. Implementation

**Inputs:** task spec, source files, related docs

**Process:**

- อ่านไฟล์ก่อนแก้
- แก้เฉพาะ scope ที่ได้รับ
- ใช้ pattern เดิมของ codebase
- sync docs/memory เมื่อ behavior หรือ status เปลี่ยน
- รัน formatter/check/test ที่เกี่ยวข้อง

**Output:** code/docs ที่แก้แล้ว พร้อม verification result

### 5. Review

**Inputs:** diff / files modified / requirement docs

**Process:**

- ตรวจ bug risk, performance, optimization, security
- อ้างอิง file + line เมื่อพบ issue
- ไม่ block ด้วย style preference ที่ formatter/linter จัดการได้

**Output:** review report แบบ findings-first

### 6. Testing & Verification

**Inputs:** changed files + available scripts

**Process:**

- เลือก test ที่เล็กที่สุดแต่ครอบคลุมความเสี่ยง
- รัน type check / unit / integration / build เท่าที่เกี่ยวข้อง
- ถ้ารันไม่ได้ ให้บันทึกเหตุผลและ residual risk

**Output:** test status พร้อมคำสั่งที่รัน

### 7. Handoff

**Inputs:** context status + unfinished work

**Process:**

- ถ้า context ใกล้เต็ม ให้หยุดงานและสร้าง `_docs/handoff/{YYYY-MM-DD}-{topic}.handoff.md`
- ใส่สิ่งที่ทำแล้ว, สิ่งที่ค้าง, ไฟล์ที่แก้, ปัญหา, และคำสั่งสำหรับ context ถัดไป

**Output:** handoff file ที่ให้ agent ถัดไปทำต่อได้ทันที

---

## Tool Preferences

| Task           | Tool                                                                  |
| -------------- | --------------------------------------------------------------------- |
| ค้นไฟล์ / text | `rg`, `rg --files`                                                    |
| อ่านหลายไฟล์   | parallel read เมื่อทำได้                                              |
| แก้ไฟล์        | patch/edit ที่เห็น diff ชัด                                           |
| สร้าง symlink  | shell command ที่ตรวจผลได้                                            |
| ตรวจ format    | `prettier --check` หรือ `prettier --write` เมื่ออยู่ใน Execution Mode |
| ตรวจ reference | `rg`                                                                  |
| ตรวจ git diff  | `git diff --stat` และ `git diff -- <path>`                            |

---

## Anti-patterns

- ❌ เริ่มแก้ไฟล์ก่อนอ่าน rules/current state
- ❌ ทำงานเป็น Backend/Frontend โดยไม่อ่าน role rules ของ role นั้น
- ❌ อ่านทั้ง repo สำหรับงานสั้นโดยไม่จำเป็น
- ❌ skip test/format โดยไม่บอกเหตุผล
- ❌ เปลี่ยน workflow หลักของ 8 agents โดยไม่ถูกสั่ง
- ❌ แตะ folder ที่ผู้ใช้ exclude จาก scope
