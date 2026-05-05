---
feature: { feature-name }
type: feature-checklist
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: Lead (orchestration), all agents (per-task)
status: draft
---

# {Feature Name} — Checklist

> Task list ทั้งหมด พร้อม checkbox + path ของไฟล์ — ใช้สำหรับ continuity ข้าม session
> ทุกข้อระบุ path ที่ต้องสร้าง/แก้ไขเพื่อให้ session ใหม่ทำต่อได้ทันที
> เมื่อทำเสร็จ ให้เปลี่ยน `- [ ]` เป็น `- [x]` พร้อม comment timestamp `<!-- done: YYYY-MM-DD -->`

---

## Phase 1: {Phase Name — เช่น Schema Foundation}

### Database Migrations

- [ ] **P1.1** สร้าง migration M1 — `{project}/src/database/postgresql/migrations/data/YYYYMMDD-HHMM-create-{table}.ts`
  - [ ] up: createTable + index + seed
  - [ ] down: dropTableIfExists
  - [ ] verify: รัน migrate latest → ตรวจ table exists + seed count
- [ ] **P1.2** สร้าง migration M2 — `*-create-{table2}.ts`
  - [ ] up
  - [ ] down
  - [ ] verify
- [ ] **P1.x** ... (ทำต่อให้ครบทุก migration ที่ระบุใน migration.md)
- [ ] **P1.N** Run migration บน dev — `npm run db:migrate:status` แล้ว `npm run db:migrate:latest`
  - [ ] ตรวจ all tables created
  - [ ] ตรวจ seed data
  - [ ] ตรวจ FK constraints

### Models (Objection.js)

- [ ] **P1.M1** สร้าง `{project}/src/database/postgresql/models/data/{model1}.model.ts`
- [ ] **P1.M2** สร้าง `{model2}.model.ts`
- [ ] **P1.Mx** ทุก model define `relationMappings`
- [ ] **P1.My** Type interfaces (เช่น `{Model}Type`)

---

## Phase 2: {Phase Name — เช่น Master CRUD (Backend + Frontend)}

### Backend — Constants / Middleware Update

- [ ] **P2.1** อัพเดต `{project}/src/constant/{file}.ts` — เพิ่ม helper / enum / config
- [ ] **P2.2** สร้าง / อัพเดต `{project}/src/middlewares/{name}.middleware.ts`

### Backend — Module A

- [ ] **P2.3** สร้าง `{project}/src/modules/{version}/{namespace}/{moduleA}/{moduleA}.interface.ts`
  - [ ] Zod schemas
  - [ ] TS types
- [ ] **P2.4** สร้าง `{moduleA}.repository.ts`
  - [ ] CRUD repository functions
- [ ] **P2.5** สร้าง `{moduleA}.service.ts`
  - [ ] CRUD services + validation rules
- [ ] **P2.6** สร้าง `{moduleA}.adapter.ts`
- [ ] **P2.7** สร้าง `{project}/src/api/{version}/{namespace}/{moduleA}/{moduleA}.controller.ts`
- [ ] **P2.8** สร้าง `{moduleA}.routes.ts`
- [ ] **P2.9** Register route ใน `index.routes.ts`

### Backend — Module B

- [ ] **P2.10** สร้าง interface / repository / service / adapter / controller / routes (mirror Module A)

### Backend — Tests

- [ ] **P2.X1** Unit tests: `{moduleA}.service.spec.ts`
- [ ] **P2.X2** Unit tests: `{moduleB}.service.spec.ts`
- [ ] **P2.X3** Integration test: full CRUD flow

### Frontend — Setup

- [ ] **F2.1** อัพเดต `{frontend-project}/src/utils/rbac.ts` — เพิ่ม resource ใหม่ + permissions
- [ ] **F2.2** อัพเดต `{frontend-project}/src/routes/paths.ts`
- [ ] **F2.3** อัพเดต `{frontend-project}/src/layouts/{layout}/config-navigation.tsx` — เพิ่มเมนู
- [ ] **F2.4** อัพเดต `{frontend-project}/src/locales/langs/{en,th}.json` — เพิ่ม section i18n

### Frontend — Reusable Components

- [ ] **F2.5** สร้าง `{frontend-project}/src/components/{feature-name}/{shared}.tsx`

### Frontend — Redux Slice

- [ ] **F2.6** สร้าง `src/types/store/{slice}.ts`
- [ ] **F2.7** สร้าง `src/store/services/{feature}-request.ts`
- [ ] **F2.8** สร้าง `src/store/actions/{slice}.ts`
- [ ] **F2.9** สร้าง `src/store/reducers/{slice}.ts` + register ใน combineReducers
- [ ] **F2.10** สร้าง `src/store/sagas/{slice}.ts` + register ใน rootSaga
- [ ] **F2.11** สร้าง hook `use{Feature}` (อยู่ใน reducer หรือแยกไฟล์)

### Frontend — Page A: {Page Name}

- [ ] **F2.12** สร้าง `src/sections/{feature-name}/{feature}.enum.ts`
- [ ] **F2.13** สร้าง `src/sections/{feature-name}/utils/{feature}-form.ts` (Yup schema)
- [ ] **F2.14** สร้าง `src/sections/{feature-name}/components/{feature}-table.tsx`
- [ ] **F2.15** สร้าง `src/sections/{feature-name}/components/{feature}-form.tsx` (shared)
- [ ] **F2.16** สร้าง `{feature}-delete-dialog.tsx` และ component อื่นๆ
- [ ] **F2.17** สร้าง views: `{feature}-list-view.tsx`, `-create-view.tsx`, `-detail-view.tsx`, `-edit-view.tsx`
- [ ] **F2.18** สร้าง pages:
  - [ ] `src/app/{routeRoot}/{feature-name}/page.tsx`
  - [ ] `src/app/{routeRoot}/{feature-name}/create/page.tsx`
  - [ ] `src/app/{routeRoot}/{feature-name}/[id]/page.tsx`
  - [ ] `src/app/{routeRoot}/{feature-name}/[id]/edit/page.tsx`

### Frontend — Page B / Page C (ถ้ามี)

- [ ] **F2.19** Mirror Page A pattern

### Frontend — Test

- [ ] **F2.X1** Manual UAT — golden path Page A/B/C

---

## Phase 3: {Phase Name — เช่น Per-X feature, integration กับโมดูลอื่น}

> ใส่ task ตามที่ระบุใน requirement / backend.md / frontend.md

### Backend

- [ ] **P3.1** ...

### Frontend

- [ ] **F3.1** ...

### Tests

- [ ] **P3.X1** Unit + Integration tests

---

## Phase 4: {Phase Name — เช่น Resolver Integration / Existing API Update / Regression}

### Backend — Update Existing Endpoints

- [ ] **P4.1** อัพเดต `path/to/existingFile.ts`
- [ ] **P4.2** ...

### Frontend — Update existing UI (ถ้ามี)

- [ ] **F4.1** ...

### Tests & Regression

- [ ] **P4.X1** Integration test: end-to-end
- [ ] **P4.X2** Regression test: existing flows
- [ ] **F4.X1** UAT

---

## Phase 5: Cleanup (LATER — แยก scope ถ้ามี)

> ทำหลังจาก Phase 1-4 deploy stable

- [ ] **P5.1** ...
- [ ] **P5.2** ...

---

## Unit Test Tracker

> ทุก test case ที่ระบุใน testcase.md / backend.md / frontend.md ต้อง track ที่นี่

| Test Case   | ผลที่คาดหวัง | ผลการ Test | สถานะ      |
| ----------- | ------------ | ---------- | ---------- |
| {testCase1} | -            | -          | - [ ] ผ่าน |
| {testCase2} | -            | -          | - [ ] ผ่าน |

---

## Cross-Session Continuity Tips

ถ้า session ใหม่อ่าน checklist นี้:

1. เช็ค checkbox ที่ติ๊กแล้ว → ทำต่อจากข้อถัดไป
2. ถ้า Phase 1 ยังไม่เสร็จ → focus migrations + models ก่อน
3. ถ้าจะเริ่ม code module ใหม่ → ดู `backend.md` หรือ `frontend.md` ประกอบ
4. ถ้าเจอ regression ระหว่างทำ Phase 4 → กลับไปดู logic ใน `backend.md`
5. หลังทำเสร็จแต่ละข้อ — อัพเดต checkbox ทันที (ห้ามรอจนเสร็จหลายข้อ)

---

## Reference Patterns

> Copy จาก backend.md / frontend.md เพื่อให้ session ใหม่ดูได้เร็ว

- Backend: `{path-to-backend-reference}`
- Backend Routes: `{path-to-routes-reference}`
- Frontend: `{path-to-frontend-reference}`
