---
agent: backend
updated: 2026-05-05
---

# Backend Agent — Current Task

> งานที่ Backend กำลังถืออยู่ — Lead เป็นคน assign

---

## Active Task

Status Value Migration — เปลี่ยน Zod enum + repo/service literal `'archived'` → `'inactive'` ใน Phase 2 modules (feature/packageCrud/addon)

**Status:** done (2026-05-05)
**Reason:** User สั่งให้ Phase 2 รับเฉพาะ `'active'` + `'inactive'` (ไม่ใช้ `'archived'`). Frontend ส่ง `inactive` มา. Phase 2 modules เท่านั้น — modules อื่นไม่แตะ.

**Investigation (Step 1):**
- DB column type: `status_type VARCHAR(25)` ไม่มี CHECK/ENUM constraint (อ้างอิง `src/database/postgresql/migrations/tableTemplate.ts:14-19`) → ไม่ต้อง alter schema
- DB current data: comp_features=14 active, comp_packages=7 active, comp_addons=9 active (zero archived) → ไม่ต้อง backfill data
- Zod schemas: ใช้ `z.enum(['active', 'inactive'])` อยู่แล้วใน 3 interfaces → ไม่ต้องแก้
- Repo literal: 3 ที่ใช้ `StatusTypeEnumCode.ARCHIVED` → ต้อง swap เป็น `INACTIVE`
- **Decision: ไม่สร้าง migration** — เปลี่ยนเฉพาะ code

**Approach for shared enum:** Option B (light) — ใช้ `StatusTypeEnumCode.INACTIVE` ที่มีอยู่แล้วใน `src/constant/general.ts:13` (ไม่ลบ `ARCHIVED` ออก กัน break modules อื่นที่ยังใช้)

**Files (modified, all in happywork-backend/):**
- `src/modules/v2/sale-dashboard/feature/feature.repository.ts` — `softDeleteFeatureRepository`: `ARCHIVED` → `INACTIVE` + comment
- `src/modules/v2/sale-dashboard/packageCrud/packageCrud.repository.ts` — `softDeletePackageRepository`: `ARCHIVED` → `INACTIVE` + 2 comments updated (`getPackageByUuidIncludingArchivedRepository`, `getPackageFeaturesRepository`)
- `src/modules/v2/sale-dashboard/packageCrud/packageCrud.service.ts` — 3 comment lines ใน `updatePackageUuidService` updated (archived → inactive ที่ถูก soft-delete)
- `src/modules/v2/sale-dashboard/addon/addon.repository.ts` — `softDeleteAddonRepository`: `ARCHIVED` → `INACTIVE` + comment

**Acceptance:**
- [x] DB schema check — VARCHAR(25), no constraint (no migration needed) <!-- done: 2026-05-05 -->
- [x] DB data check — zero `archived` rows in comp_features/comp_packages/comp_addons (no backfill needed) <!-- done: 2026-05-05 -->
- [x] Repo soft-delete swap (3 files) <!-- done: 2026-05-05 -->
- [x] Comments updated to use "inactive" terminology in Phase 2 modules <!-- done: 2026-05-05 -->
- [x] grep verify — zero `'archived'` / `StatusTypeEnumCode.ARCHIVED` ใน Phase 2 sale-dashboard modules <!-- done: 2026-05-05 -->
- [x] TS compile clean (`tsc --noEmit` exit 0) <!-- done: 2026-05-05 -->
- [x] Prettier formatted (4 files unchanged — already conformant) <!-- done: 2026-05-05 -->
- [x] DB re-verify after code change — still 14/7/9 active rows, no mutation <!-- done: 2026-05-05 -->
- [x] Checklist updated — added Status Value Migration section (PSV.1–PSV.10) <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

Phase 2 Refactor — Move 3 modules (`feature`, `packageCrud`, `addon`) จาก `v2/admin/` → `v2/sale-dashboard/` + middleware swap (`validateAccessToken` → `validateSaleDashboardAccessToken`)

**Status:** done (2026-05-05)
**Reason:** Frontend คือ `happywork-sale-cms` ที่ใช้ sale dashboard token — Phase 2 Master CRUD ต้องอยู่ใน sale-dashboard scope. URL: `/api/v2/{features,packages,addons}` → `/api/v2/sale-dashboard/{features,packages,addons}`.

**Files (moved + edited, all in happywork-backend/):**
- Folders moved (6 folders, 18 files):
  - `src/api/v2/admin/{feature,packageCrud,addon}/` → `src/api/v2/sale-dashboard/{feature,packageCrud,addon}/`
  - `src/modules/v2/admin/{feature,packageCrud,addon}/` → `src/modules/v2/sale-dashboard/{feature,packageCrud,addon}/`
- Imports updated (12 file edits): controllers (3) + routes (3) + cross-module (6: packageCrud.{i,a,s} + addon.{i,a,s})
- Middleware swap + basePath + tags updated in 3 routes files
- `src/api/v2/sale-dashboard/sale-dashboard.routes.ts` — register featureRouter, packageCrudRouter, addonRouter
- `src/api/v2/admin/index.routes.ts` — removed 3 register lines + 3 imports

**Acceptance:**
- [x] Folders moved (6 folders) <!-- done: 2026-05-05 -->
- [x] Internal imports updated to `@/modules/v2/sale-dashboard/...` (12 files) <!-- done: 2026-05-05 -->
- [x] Middleware: `validateAccessToken` → `validateSaleDashboardAccessToken` ใน 3 routes (20 endpoints รวม) <!-- done: 2026-05-05 -->
- [x] `requireSuperAdmin` คงไว้ — `validateSaleDashboardAccessToken` populate `req.user.role` from `saleDashboardUser.role` (line 76) → compatible <!-- done: 2026-05-05 -->
- [x] OpenAPI basePath updated (3 files): `/api/v2/sale-dashboard/{features|packages|addons}` <!-- done: 2026-05-05 -->
- [x] Router registration: `sale-dashboard.routes.ts` mount 3 routers ก่อน catch-all `/` mounts (no path collision detected) <!-- done: 2026-05-05 -->
- [x] Admin index cleaned (3 routes removed, `/package`, `/subscription`, `/payment` คงไว้) <!-- done: 2026-05-05 -->
- [x] grep verify — zero `@/modules/v2/admin/{feature,packageCrud,addon}` ใน src/ <!-- done: 2026-05-05 -->
- [x] TS compile clean (`tsc --noEmit` exit 0) <!-- done: 2026-05-05 -->
- [x] Prettier formatted (routes wrapped long-line; rest already conformant) <!-- done: 2026-05-05 -->
- [x] Checklist `_docs/requirement/feature-management/feature-management.checklist.md` — added "Phase 2 Refactor — Move-to-SaleDashboard" section (PMS.1–PMS.23) <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

URL Fix — Remove `/admin` segment from OpenAPI `basePath` of Phase 2 routes (feature/packageCrud/addon)

**Status:** done (2026-05-05)
**Reason:** Express mounts `v2AdminRouter` at `/api/v2` (not `/api/v2/admin`); leaf routers under it `.use('/features', ...)`, `.use('/packages', ...)`, `.use('/addons', ...)` → actual URLs are `/api/v2/features|packages|addons`. Phase 2 routes' OpenAPI `basePath` was inadvertently set to `/api/v2/admin/...` → Swagger path mismatch.
**Files (modified, all in happywork-backend/):**
- `src/api/v2/admin/feature/feature.routes.ts` — `basePath = '/api/v2/features'` + 6 comment URL refs (lines 27, 43, 53, 63, 73, 83) updated
- `src/api/v2/admin/packageCrud/packageCrud.routes.ts` — `basePath = '/api/v2/packages'` + 8 comment URL refs (lines 34, 50, 66, 83, 93, 103, 113, 123) updated
- `src/api/v2/admin/addon/addon.routes.ts` — `basePath = '/api/v2/addons'` + 6 comment URL refs (lines 29, 45, 55, 65, 75, 85) updated

**Acceptance:**
- [x] All 3 `basePath` constants drop `/admin` segment <!-- done: 2026-05-05 -->
- [x] All inline comment URL refs updated to match new basePath <!-- done: 2026-05-05 -->
- [x] Express routing untouched (mount paths already correct in `src/api/v2/admin/index.routes.ts`) <!-- done: 2026-05-05 -->
- [x] grep verify — zero remaining `/api/v2/admin/{features,packages,addons}` in `src/` <!-- done: 2026-05-05 -->
- [x] TS compile clean (`NODE_OPTIONS=--max-old-space-size=8192 npx tsc --noEmit` exit 0) <!-- done: 2026-05-05 -->
- [x] Prettier formatted (3 files unchanged after run — already conformant) <!-- done: 2026-05-05 -->
- [x] Checklist `document/requirement/feature-management/checklist.md` — added "URL Fix" section (PURL.1–PURL.5) <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

Phase 2 Wave B4 — Package UUID Update endpoint `PATCH /api/v2/admin/packages/:packageUuid/identifier` (P2.15a–P2.15e)

**Status:** done — wave B4 complete (2026-05-05)
**Files (modified, all in happywork-backend/):**
- `src/modules/v2/admin/packageCrud/packageCrud.interface.ts` — เพิ่ม `updatePackageUuidSchema` (params+body), TS types `UpdatePackageUuidSchema/Params/Body`, `UpdatePackageUuidServiceInput`
- `src/modules/v2/admin/packageCrud/packageCrud.repository.ts` — เพิ่ม `getPackageByUuidIncludingArchivedRepository` + `updatePackageUuidRepository(currentUuid, newUuid, updatedBy, trx?)` + `updateCompanySelectedPackageUuidRepository(currentUuid, newUuid, updatedBy, trx?)` (denormalized cache sync)
- `src/modules/v2/admin/packageCrud/packageCrud.service.ts` — เพิ่ม `updatePackageUuidService(currentUuid, newUuid, actor)`: validate ≠ current → load existing → ensure newUuid ไม่ซ้ำ (incl archived) → trx update package+company → re-fetch return
- `src/api/v2/admin/packageCrud/packageCrud.controller.ts` — เพิ่ม `updatePackageUuidController`
- `src/api/v2/admin/packageCrud/packageCrud.routes.ts` — mount `PATCH /:packageUuid/identifier` ก่อน `PUT /:packageUuid` (กัน Express greedy param collision) + register OpenAPI

**Acceptance:**
- [x] Endpoint `PATCH /api/v2/admin/packages/:packageUuid/identifier` พร้อม middleware chain `validateAccessToken → requireSuperAdmin → validateRequest` <!-- done: 2026-05-05 -->
- [x] Transaction atomic: comp_packages.uuid + comp_companies.selected_package_uuid (denormalized cache) update ใน trx เดียวกัน <!-- done: 2026-05-05 -->
- [x] Validate newUuid ≠ currentUuid (400) + ไม่ซ้ำกับ row อื่นรวม archived (400) + currentUuid มีจริง active/inactive (404) <!-- done: 2026-05-05 -->
- [x] Verification grep — denormalized refs เจอเฉพาะ `comp_companies.selected_package_uuid` (sync ใน trx) + `comp_subscription_historys.selected_package_uuid` (audit snapshot — ไม่แตะ, SQL trigger จะ insert row ใหม่อัตโนมัติ) <!-- done: 2026-05-05 -->
- [x] TS compile clean (`tsc --noEmit` exit 0 with NODE_OPTIONS heap bump) <!-- done: 2026-05-05 -->
- [x] Prettier formatted (5 files unchanged after second run) <!-- done: 2026-05-05 -->
- [x] Checklist P2.15a–P2.15e ticked with timestamp <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

Phase 2 Wave B3 — Package CRUD module (P2.10–P2.15) + Addon module (P2.16–P2.21)

**Status:** done — wave B3 complete (2026-05-05)
**Files (created):**
- `happywork-backend/src/modules/v2/admin/packageCrud/packageCrud.interface.ts`
- `happywork-backend/src/modules/v2/admin/packageCrud/packageCrud.repository.ts`
- `happywork-backend/src/modules/v2/admin/packageCrud/packageCrud.service.ts`
- `happywork-backend/src/modules/v2/admin/packageCrud/packageCrud.adapter.ts`
- `happywork-backend/src/api/v2/admin/packageCrud/packageCrud.controller.ts`
- `happywork-backend/src/api/v2/admin/packageCrud/packageCrud.routes.ts`
- `happywork-backend/src/modules/v2/admin/addon/addon.interface.ts`
- `happywork-backend/src/modules/v2/admin/addon/addon.repository.ts`
- `happywork-backend/src/modules/v2/admin/addon/addon.service.ts`
- `happywork-backend/src/modules/v2/admin/addon/addon.adapter.ts`
- `happywork-backend/src/api/v2/admin/addon/addon.controller.ts`
- `happywork-backend/src/api/v2/admin/addon/addon.routes.ts`

**Files (modified):**
- `happywork-backend/src/api/v2/admin/index.routes.ts` — register `/packages` + `/addons`

**Acceptance:**
- [x] Package CRUD module ที่ `/api/v2/admin/packages` พร้อม replace-features + effective-menus <!-- done: 2026-05-05 -->
- [x] Addon module ที่ `/api/v2/admin/addons` พร้อม replace-features + is_quantifiable rule <!-- done: 2026-05-05 -->
- [x] Composite UNIQUE(code, billing_interval) enforce ที่ service (ทั้ง package + addon) <!-- done: 2026-05-05 -->
- [x] replaceFeatures ใช้ Objection transaction (rollback safe) <!-- done: 2026-05-05 -->
- [x] TS compile clean (`tsc --noEmit` exit 0 with NODE_OPTIONS heap bump) <!-- done: 2026-05-05 -->
- [x] Prettier formatted <!-- done: 2026-05-05 -->
- [x] Checklist P2.10-P2.21 ticked with timestamp <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

Phase 2 Wave B1 — `getAvailableMenuKeys()` (P2.1) + `requireSuperAdmin` middleware (P2.2)

**Status:** done — wave B1 complete (2026-05-05)
**Files:**
- `happywork-backend/src/constant/compPermission.ts` (added helper)
- `happywork-backend/src/middlewares/requireSuperAdmin.middleware.ts` (new)

**Acceptance:**
- [x] `getAvailableMenuKeys()` exported from `src/constant/compPermission.ts` <!-- done: 2026-05-05 -->
- [x] `requireSuperAdmin` middleware exported from `src/middlewares/requireSuperAdmin.middleware.ts` <!-- done: 2026-05-05 -->
- [x] TS compile clean (`tsc --noEmit` exit 0 with NODE_OPTIONS heap bump) <!-- done: 2026-05-05 -->
- [x] Prettier formatted <!-- done: 2026-05-05 -->
- [x] Checklist P2.1, P2.2 ticked with timestamp <!-- done: 2026-05-05 -->

---

## Previous Task (closed)

Implement Phase 1 — Schema Foundation for feature-management

**Task ID:** FEAT-101 ถึง FEAT-117 (P1.1–P1.18)
**Feature:** feature-management
**Description:** สร้าง 8 migrations + 7 Objection.js models ตาม `migration.md` + `ssd.md §3`
**Assigned by:** Lead (main thread)
**Started:** 2026-05-04
**Doc reference:** `document/requirement/feature-management/{migration.md,ssd.md,checklist.md,backend.md}`

**Acceptance criteria:**

- [x] 8 migration files created (M1–M8)
- [x] 7 Objection.js models created
- [x] All models define relationMappings + Type interfaces
- [x] Q1=C (Cartesian seed in M3) enforced
- [x] Q4=B (FK RESTRICT for feature_id in M6) enforced
- [x] UUIDs in M2/M4 seed preserve `packageMasterData` / `addonsMasterData` exactly
- [x] All files prettier-formatted
- [x] TypeScript compile clean (no errors in new files)
- [ ] User to run `npm run db:migrate:latest` on dev (P1.9 — outside agent scope)

## Pre-implementation Checklist

- [x] อ่าน rules.md / agent/backend/rules.md / skills.md / current-task.md <!-- done: 2026-05-04 -->
- [x] อ่าน migration.md (source of truth สำหรับ migration code) <!-- done: 2026-05-04 -->
- [x] อ่าน ssd.md §3 (schema definition + Q1-Q5 decisions) <!-- done: 2026-05-04 -->
- [x] อ่าน backend.md (overall structure) <!-- done: 2026-05-04 -->
- [x] อ่าน checklist.md (P1.1–P1.18) <!-- done: 2026-05-04 -->
- [x] ตรวจ Files Reviewed ใน summary.md ครอบคลุมไฟล์ที่จะแก้ (✅ pass) <!-- done: 2026-05-04 -->
- [x] อ่าน reference patterns: tableTemplate, alterTableOnUpdate, packageMasterData/addonsMasterData ตัวจริง <!-- done: 2026-05-04 -->
- [x] อ่าน existing models pattern (compAnnualHolidays, compCompanies, aprvFlowsFormMapping) <!-- done: 2026-05-04 -->

## Implementation Sub-checklist

- [x] M1 — `20260504-1000-create-comp_features.ts` (+ GIN index + 14 seed features)
- [x] M2 — `20260504-1001-create-comp_packages.ts` (+ GIN index + 7 seed packages, UUID preserved)
- [x] M3 — `20260504-1002-create-comp_package_features.ts` (+ Cartesian seed Q1=C)
- [x] M4 — `20260504-1003-create-comp_addons.ts` (+ GIN index + 9 seed addons, HappyBeacon quantifiable)
- [x] M5 — `20260504-1004-create-comp_addon_features.ts` (link table)
- [x] M6 — `20260504-1005-create-comp_company_features.ts` (Q4=B: feature_id RESTRICT)
- [x] M7 — `20260504-1006-create-comp_company_addons.ts` (+ migrate from comp_companies.add_ons)
- [x] M8 — `20260504-1007-alter-comp_companies-package-id-fk.ts` (HIGH RISK — backfill + FK swap)
- [x] Models 1-7 with full relationMappings + JSON schemas + Type interfaces

## Files Created

| Path                                                                                                                | Purpose                                                                |
| ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1000-create-comp_features.ts`                   | M1: master features table + 14 seed features                           |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1001-create-comp_packages.ts`                   | M2: master packages table + 7 seed packages (UUIDs preserved)          |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1002-create-comp_package_features.ts`           | M3: link table + Cartesian seed (Q1=C)                                 |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1003-create-comp_addons.ts`                     | M4: master addons table + 9 seed addons (HappyBeacon quantifiable)     |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1004-create-comp_addon_features.ts`             | M5: link table                                                         |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1005-create-comp_company_features.ts`           | M6: per-company override (Q4=B: feature_id FK RESTRICT)                |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1006-create-comp_company_addons.ts`             | M7: per-company addons + migrate from `comp_companies.add_ons` JSONB   |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1007-alter-comp_companies-package-id-fk.ts`     | M8: backfill + FK swap (HIGH RISK)                                     |
| `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts`                                       | Model + relationMappings (packages, addons, companyOverrides)          |
| `happywork-backend/src/database/postgresql/models/data/compPackages.model.ts`                                       | Model + relationMappings (features, packageFeatures)                   |
| `happywork-backend/src/database/postgresql/models/data/compPackageFeatures.model.ts`                                | Link model + relationMappings (package, feature)                       |
| `happywork-backend/src/database/postgresql/models/data/compAddons.model.ts`                                         | Model + relationMappings (features, addonFeatures, companyAddons)      |
| `happywork-backend/src/database/postgresql/models/data/compAddonFeatures.model.ts`                                  | Link model + relationMappings (addon, feature)                         |
| `happywork-backend/src/database/postgresql/models/data/compCompanyFeatures.model.ts`                                | Override model + relationMappings (company, feature)                   |
| `happywork-backend/src/database/postgresql/models/data/compCompanyAddons.model.ts`                                  | Per-company addon model + relationMappings (company, addon)            |

## Deviations from SSD

| Description                                                                                                                                              | Reason                                                                                                                                          | Reported to Lead?            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `comp_packages` seed = **7 records** (not 8 as stated in SSD §3.2 / checklist P1.2)                                                                       | `packageMasterData` ใน `package.interface.ts` มีจริง 7 records: Seed + LITE×2 + CORE×2 + PRO×2 = 7. SSD/checklist เขียน 8 น่าจะ count ผิด        | Yes — flagged in this report |
| `comp_addons` seed = **9 records** (HappyBeacon + E-Slip×2 + Evaluation×2 + HappyPM×2 + E-Signature×2)                                                    | จำนวน addons ที่อยู่ใน `addonsMasterData` จริง                                                                                                  | Yes — informational          |
| `tableTemplate` adds `company_id BIGINT` to all master tables (comp_features, comp_packages, comp_addons)                                                | ตามคำแนะนำของ Lead — keep tableTemplate consistent + column nullable; conform to existing pattern (`const_packages` ก็มี structure คล้ายกัน) | No deviation — by design     |

## Blockers

ไม่มี blocker ตอนนี้ — Phase 1 ทำเสร็จครบ 18/18 (เว้น P1.9 ที่ user ต้อง execute เอง)

## Pre-execution Notes for User (Before Running P1.9)

ก่อนรัน `npm run db:migrate:latest` บน dev:

1. **Pre-validation query** (ตามใน `migration.md` Pre-Migration Checklist) — ตรวจว่า `comp_companies.selected_package_uuid` ทุกค่า match กับ 7 UUID ใน packageMasterData. ถ้ามี orphan UUID ต้องแก้ก่อน mig M8 จะ rollback
2. **Backup DB** ก่อน apply M8 (HIGH RISK)
3. หลังรัน latest:
   - `SELECT COUNT(*) FROM comp_features;` ต้อง = 14
   - `SELECT COUNT(*) FROM comp_packages;` ต้อง = 7
   - `SELECT COUNT(*) FROM comp_addons;` ต้อง = 9
   - `SELECT COUNT(*) FROM comp_package_features;` ต้อง = 14×7 = 98 (Cartesian)
   - `SELECT COUNT(*) FROM comp_companies cc LEFT JOIN comp_packages cp ON cc.package_id=cp.id WHERE cp.id IS NULL;` ต้อง = 0

## Recent updates (latest 5)

| Date       | Update                                                                                       |
| ---------- | -------------------------------------------------------------------------------------------- |
| 2026-05-04 | **HOTFIX** — M6 + M7 duplicate `company_id` (PG 42701) แก้แล้ว: ใช้ `table.foreign()` แทน redeclare + raw `ALTER COLUMN ... SET NOT NULL` (Option A). M8 review แล้วไม่มี bug. Prettier + tsc clean. รอ DevOps resume `npm run db:migrate:latest` |
| 2026-05-04 | Phase 1 complete — 8 migrations + 7 models created, formatted, type-checked clean             |
| 2026-05-04 | Models include relationMappings + Type interfaces + jsonAttributes for JSONB fields           |
| 2026-05-04 | Q1=C Cartesian seed in M3, Q4=B FK RESTRICT in M6, M8 backfill via selected_package_uuid       |
| 2026-04-29 | Previous task: type-safety-any-cleanup — closed                                              |
