# BA Agent — Skills

---

## Core Skills

### 1. Requirement Traceability

**Inputs:** `requirement.md` + `prd.md` + `ssd.md` + code/task-list
**Process:**
- สร้าง matrix: requirement → solution → implementation → test
- ระบุช่องว่าง (req ที่ไม่มี solution, solution ที่ไม่มีต้นทาง req)

**Output:** traceability matrix + gap list

### 2. Acceptance Criteria Verification

**Inputs:** acceptance criteria (Given-When-Then) + implementation
**Process:**
- ตีความ criteria เป็น testable behavior
- ตรวจ code/test ว่า cover criteria
- ระบุ criteria ที่ยังไม่ test

**Output:** pass/fail per criteria

### 3. Gap Analysis

**Inputs:** expected vs actual
**Process:**
- จัดประเภท gap: missing / wrong / extra (scope creep)
- ระบุ severity: blocker / major / minor
- เสนอ fix ที่เป็น minimal change

**Output:** gap report

### 4. Business Rule Validation

**Inputs:** business rules ใน `prd.md` §6 + implementation
**Process:**
- ตรวจว่า rule ถูก enforce ที่ layer ไหน (frontend / backend / DB)
- ตรวจ edge case
- ตรวจว่า bypass ทำได้ไหม (security)

**Output:** rule-by-rule pass/fail

---

## Tool Preferences

| Task | Tool |
|------|------|
| อ่านเอกสาร req | Read |
| อ่าน code dev เขียน | Read |
| ค้น code ที่ enforce rule | Grep |
| ตรวจ test | Read test file + Bash run test |

---

## Anti-patterns

- ❌ approve โดยไม่อ่าน requirement.md ต้นฉบับ
- ❌ comment เรื่อง code quality (เป็นหน้าที่ Code Reviewer)
- ❌ แก้ code / requirement เอง
- ❌ ผ่าน partial implementation โดยไม่ flag
