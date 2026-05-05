---
feature: type-safety-any-cleanup
type: product-requirement-document
created: 2026-04-29
updated: 2026-04-29
owner: SA
status: in-progress
---

# Product Requirement Document — Type Safety Any Cleanup

## 1. Overview (ภาพรวม)

งานนี้เป็น developer-facing refactor เพื่อให้ `mof-backend` ปลอดภัยขึ้นจากการใช้ `any` ใน production source และทำให้ model ทุกตัวมี exported type ที่ module อื่นอ้างอิงได้โดยไม่เดา field เอง

## 2. User Personas (ผู้ใช้)

| Persona | Description | Need |
|---------|------------|------|
| Backend developer | ผู้พัฒนา backend ในโปรเจค MOF | ต้องการ compiler ช่วยจับการเรียก field/params ผิด |
| Code reviewer | ผู้ตรวจ code | ต้องการ type contract ที่อ่านง่ายและ enforce ได้ |

## 3. User Stories

### US-1: Model Type Reuse
- Story: As a backend developer, I want every model to export a type, so that I can reference DB model fields consistently.
- Acceptance Criteria:
  - [ ] ทุก model class มี exported type 1 ตัว

### US-2: Safer Production Source
- Story: As a backend developer, I want production source to avoid explicit `any`, so that incorrect usage is caught earlier.
- Acceptance Criteria:
  - [ ] `mof-backend/src` ไม่มี explicit `any`

## 4. User Journey / Flow

Developer imports model type or local row type → writes service/repository with explicit params → lint/typecheck catches unsafe usage before runtime

## 5. UI / UX Requirements

ไม่มี UI changes

## 6. Business Rules (กฏทางธุรกิจ)

| Rule ID | Description | Enforced where |
|---------|------------|----------------|
| BR-1 | Runtime behavior must remain unchanged | lint/typecheck/tests/review |
| BR-2 | Function-specific params remain local to function/module | repositories/services |

## 7. Success Metrics (ตัววัดความสำเร็จ)

- `src` explicit `any` count = 0
- lint/typecheck pass
- one exported model type per model class

## 8. Out of Scope (ไม่ทำใน feature นี้)

- tests/scripts `any` cleanup
- DB migrations
- API behavior changes

## 9. Dependencies (พึ่งพา)

- Existing TypeScript strict config
- Existing Objection.js model typing

## 10. Risks (ความเสี่ยง)

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Existing test failures obscure regressions | Medium | Report baseline separately and prioritize lint/typecheck |
