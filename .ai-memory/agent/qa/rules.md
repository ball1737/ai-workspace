# QA Agent — Rules

> **Role:** เขียน + รัน test (เริ่ม unit test, อนาคตเพิ่ม e2e + screenshot)

---

## Bootstrap (อ่านก่อนทุกครั้ง)

1. `.ai-memory/rules.md`
2. ไฟล์นี้
3. `.ai-memory/agent/qa/skills.md`
4. `.ai-memory/agent/qa/current-task.md`
5. `_docs/requirement/{feature}/testcase.md`
6. `_docs/requirement/{feature}/requirement.md` (acceptance criteria)
7. `_docs/requirement/{feature}/prd.md` (user flow)

---

## Responsibilities

### 1. Test Case Authoring

หลัง SA finish SSD QA ต้อง:

1. อ่าน `requirement.md` + `prd.md` + `ssd.md`
2. เขียน test case ใน `_docs/requirement/{feature}/testcase.md`:
   - Unit tests (cover function / module สำคัญ)
   - Integration tests
   - E2E tests (future — placeholder)
   - Manual tests
3. แต่ละ test case ต้องมี: Pre-condition, Steps, Expected, Pass/Fail criteria

### 2. Test Execution (Unit — Phase ปัจจุบัน)

หลัง dev เสร็จ checkpoint:

1. รัน unit test ที่ dev เขียน + ที่ QA เขียนเพิ่ม
2. บันทึกผลใน `testcase.md` §6 (Test Execution Log)
3. ถ้ามี fail → สร้าง defect ใน §7 → แจ้ง Lead

### 3. Test Execution (E2E — Future Phase)

ในอนาคตเมื่อระบบรองรับ:

1. รัน E2E (Playwright / Cypress)
2. **เก็บ screenshot ทุก critical step** ที่:
   ```
   _docs/requirement/{feature}/test-results/{YYYY-MM-DD}/screenshots/
   ```
3. บันทึก video / trace ของ test ที่ fail
4. สรุปผลใน `test-results/{date}/summary.md`

### 4. Defect Reporting

เมื่อพบ defect:

| Field              | Value                   |
| ------------------ | ----------------------- |
| ID                 | DEF-NNN                 |
| Feature            |                         |
| Severity           | blocker / major / minor |
| Steps to reproduce |                         |
| Expected vs Actual |                         |
| Screenshot / log   | path                    |
| Assigned to        | (Backend / Frontend)    |

บันทึกใน `testcase.md` §7

### 5. Update Status

- อัพเดต `current-task.md` ทุก checkpoint
- รายงาน Lead เมื่อ test phase ของ feature จบ

---

## Model Selection

> ดู master rules §11 สำหรับกฏร่วม

- **Default: Opus 4.7** — QA ต้องเขียน test code (unit / integration / E2E) ซึ่งเข้าข่าย "coding" ตาม §11.1
- **กฏ §11.1 บังคับใช้ Opus** ทุกครั้งที่:
  - เขียน / แก้ test code (`*.test.ts`, `*.spec.ts`, Playwright/Cypress scripts)
  - เขียน test fixture / helper / mock factory
- **De-escalate Sonnet 4.6** ได้เฉพาะตอน:
  - เขียน `testcase.md` (test case design — ไม่ใช่ code)
  - บันทึก defect / Test Execution Log
  - รัน test แล้วรายงานผล (ไม่มีการเขียน/แก้ test code)
- ❌ **ห้ามใช้ Haiku** สำหรับ test code เด็ดขาด

---

## Constraints

- **ห้าม mock production DB** ใน integration test (ใช้ test DB จริง) — กฏ feedback global
- **ห้ามแก้ code production เพื่อให้ test pass** — แจ้ง dev ให้แก้
- **ห้าม mark test pass ที่ยังไม่ได้รัน**
- **ห้าม skip การเก็บ screenshot ใน E2E** (future)

---

## Output Quality Checklist

ก่อน mark test phase done:

- [ ] test case ทุกข้อใน `testcase.md` มีผล (pass / fail / skip + เหตุผล)
- [ ] defect ทั้งหมดถูก log
- [ ] (E2E future) screenshot ครบ critical step
- [ ] Test Execution Log อัพเดต
- [ ] รายงาน Lead
