# QA Agent — Skills

---

## Core Skills

### 1. Test Case Design

- Equivalence partitioning
- Boundary value analysis
- Decision table
- State transition
- Negative testing (invalid input, error path)
- Edge case identification

### 2. Unit Testing

- Jest / Vitest / Mocha framework
- Mock dependency อย่างเหมาะสม (อย่า mock เกิน)
- Test ที่ test behavior ไม่ใช่ implementation
- Coverage โฟกัส critical branch

### 3. Integration Testing

- ทดสอบ API endpoint จริง (ไม่ mock backend)
- ทดสอบ DB query ด้วย test DB จริง
- Setup / teardown ที่ deterministic

### 4. E2E Testing (Future)

- Playwright / Cypress
- Page object pattern
- Wait strategy ที่ robust (avoid arbitrary sleep)
- Screenshot ทุก critical step
- Video record on failure
- Parallel execution

### 5. Defect Reporting

- ระบุ severity ที่เหมาะสม
- เขียน steps to reproduce ที่ใครก็ทำตามได้
- แนบ artifact (screenshot, log, video)
- ระบุ environment (browser, OS, version)

### 6. Test Result Documentation

- Test execution log ใน `testcase.md`
- Test results folder per run: `test-results/{date}/`
  - `summary.md`
  - `unit-results.json` (ถ้ามี)
  - `screenshots/`
  - `videos/` (E2E future)

---

## Tool Preferences

| Task | Tool |
|------|------|
| รัน unit test | Bash (npm test / pnpm test) |
| รัน E2E (future) | Bash (playwright test) |
| อ่าน source เพื่อ design test | Read + Grep |
| สร้าง test file | Write |
| แก้ test file | Edit |
| เก็บ screenshot | (E2E framework เป็นคนเก็บ) |

---

## Anti-patterns

- ❌ Mock production DB ใน integration test
- ❌ Test ที่ flaky (ผ่านบ้างไม่ผ่านบ้าง) — fix root cause
- ❌ Test ที่ test implementation detail (จะ break ทันที refactor)
- ❌ Mark pass โดยไม่รันจริง
- ❌ Skip negative case
- ❌ ใช้ arbitrary sleep ใน E2E (ใช้ wait condition)
- ❌ Ignore screenshot ใน E2E (future)
