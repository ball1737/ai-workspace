---
feature: type-safety-any-cleanup
type: requirement
created: 2026-04-29
updated: 2026-04-29
owner: SA
status: in-progress
---

# Requirement — Type Safety Any Cleanup

## 1. Background (ที่มา)

โปรเจค `mof-backend` เปิด TypeScript strict อยู่แล้ว แต่ production source ยังมี explicit `any` กระจายอยู่ใน shared utilities, middleware, controllers/routes, และ repository row mapping ทำให้เรียกใช้งานผิดพลาดได้ง่ายโดย compiler ไม่ช่วยเตือนเต็มที่

## 2. Goals (เป้าหมาย)

- เพิ่ม exported model type ให้ทุก Objection model class
- ลด unsafe typing ใน `mof-backend/src` จนไม่เหลือ explicit `any`
- คง runtime behavior, API response shape, DB schema, และ route paths เดิม

## 3. Non-Goals (สิ่งที่จะไม่ทำในรอบนี้)

- ไม่แก้ tests/scripts ให้ปลอด `any` ในรอบแรก
- ไม่แก้ migration หรือ DB schema
- ไม่เปลี่ยน public API contract
- ไม่ refactor business logic ที่ไม่จำเป็นต่อ type cleanup

## 4. Stakeholders

| Role | Person / Team | Responsibility |
|------|--------------|----------------|
| Owner | User | Defines type-safety expectation |
| Backend | Codex | Implement source refactor and verification |
| QA | Codex | Run lint/typecheck/targeted tests and report baseline failures |

## 5. Functional Requirements (ความต้องการเชิงฟังก์ชัน)

### FR-1: Model Type Export
- Description: ทุก Objection model class ใน `src/database/postgresql/models` ต้อง export type ของตัวเอง 1 ตัวที่แทน field ทั้งหมดของ model
- Acceptance Criteria:
  - [ ] มี `export type XxxType = ModelObject<Xxx>;` หรือ equivalent สำหรับทุก class
  - [ ] ไม่มี `Insert`, `Update`, `Patch`, `Delete` aliases ที่สร้างจาก model โดยรวม

### FR-2: Remove Production Explicit Any
- Description: ลบ explicit `any` จาก `mof-backend/src`
- Acceptance Criteria:
  - [ ] AST audit ของ `src` ไม่พบ `AnyKeyword`
  - [ ] route/controller casts `as any` ถูกแทนด้วย typed contract

### FR-3: Keep Runtime Behavior
- Description: refactor นี้ต้องไม่เปลี่ยน behavior
- Acceptance Criteria:
  - [ ] API response shape เดิม
  - [ ] DB query behavior เดิม
  - [ ] ไม่มี migration

## 6. Non-Functional Requirements (ความต้องการเชิงคุณภาพ)

| Type | Requirement | Note |
|------|------------|------|
| Type Safety | `src` ไม่มี explicit `any` | ใช้ `unknown`/specific interfaces แทน |
| Maintainability | Types อยู่ใกล้ owning module/function | CRUD params ให้ function กำหนดเอง |
| Compatibility | No public API changes | Refactor only |

## 7. Constraints (ข้อจำกัด)

- ห้ามแก้ `package.json` และ `tsconfig.json`
- ห้ามแก้ migrations/DB schema
- ต้องใช้ Prettier กับไฟล์ที่แก้

## 8. Assumptions (สมมติฐาน)

- `ModelObject<Xxx>` ครอบคลุม field ที่ประกาศใน model class รวม inherited base fields
- Tests/scripts ยังอาจมี `any` ได้ในรอบนี้

## 9. Open Questions (คำถามที่ยังไม่ได้คำตอบ)

- [x] Model type export ต้องแยก insert/update/patch หรือไม่ — User confirmed: ไม่ต้อง

## 10. References

- Related code paths: `mof-backend/src/database/postgresql/models`, `mof-backend/src/middlewares`, `mof-backend/src/utils`, `mof-backend/src/api/v1`, `mof-backend/src/modules/v1`
