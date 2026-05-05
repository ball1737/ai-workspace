---
feature: {feature-name}
type: testcase
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
owner: QA
status: draft
---

# Test Cases — {Feature Name}

> รายการ test case ทั้งหมดของ feature — เขียนโดย QA หลัง SA finish SSD
> เริ่มจาก unit test, ในอนาคตจะเพิ่ม e2e + screenshot

---

## 1. Test Strategy

| Test Type | Coverage | Tool | Owner |
|-----------|---------|------|-------|
| Unit      |         |      | Backend / Frontend dev |
| Integration |       |      | QA |
| E2E       | (future) | Playwright / Cypress | QA |
| Manual    |         |      | QA |

## 2. Unit Test Cases

### UT-1: {Module / Function}
- File: `path/to/test.spec.ts`
- Description:
- Setup:
- Cases:
  - [ ] case 1: input X → expect Y
  - [ ] case 2: input Z → expect error

## 3. Integration Test Cases

### IT-1: {Flow}
- Description:
- Pre-condition:
- Steps:
- Expected:

## 4. E2E Test Cases (Future)

### E2E-1: {Happy Path}
- Description:
- User actions:
- Expected:
- Screenshot location: `_docs/requirement/{feature}/test-results/{date}/E2E-1.png`

## 5. Manual Test Cases

### MT-1: {Scenario}
- Steps:
- Expected:
- Tested by: {QA name / agent}
- Date:
- Result: pass / fail / blocked

## 6. Test Execution Log

| Run | Date | Type | Pass | Fail | Skip | Note |
|-----|------|------|------|------|------|------|
|     |      |      |      |      |      |      |

## 7. Defects Found

| ID | Test Case | Description | Severity | Status | Assigned to |
|----|-----------|------------|----------|--------|------------|
|    |           |            |          |        |            |

## 8. Test Results Folder

หลัง run test ให้เก็บผล + screenshot ที่:
```
_docs/requirement/{feature}/test-results/{YYYY-MM-DD}/
├── summary.md
├── unit-results.json
└── screenshots/
```
