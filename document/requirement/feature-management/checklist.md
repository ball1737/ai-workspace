---
feature: feature-management
type: feature-checklist
created: 2026-05-04
updated: 2026-05-06
owner: Lead (orchestration), all agents (per-task)
status: draft
---

# Feature Management — Checklist

> Task list ทั้งหมด พร้อม checkbox + path ของไฟล์ — ใช้สำหรับ continuity ข้าม session
> ทุกข้อระบุ path ที่ต้องสร้าง/แก้ไขเพื่อให้ session ใหม่ทำต่อได้ทันที

---

## Phase 1: Schema Foundation (Backend Only)

### Database Migrations

- [x] **P1.1** สร้าง migration M1 — `happywork-backend/src/database/postgresql/migrations/data/20260504-1000-create-comp_features.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + GIN index + seed feature เริ่มต้น (จาก FEATURES_NAME_MAP, menu_keys = []) <!-- done: 2026-05-04 -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->
  - [x] verify: รัน migrate latest → ตรวจ table exists + seed count <!-- done: 2026-05-04 — comp_features=14 rows, table exists -->

- [x] **P1.2** สร้าง migration M2 — `20260504-1001-create-comp_packages.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + GIN index + seed 7 packages จาก `packageMasterData` (preserve UUID เดิม) <!-- done: 2026-05-04. Schema follow-up 2026-05-04: UNIQUE(code) → UNIQUE(code, billing_interval) composite; seed code = PackageEnumCode raw values (seed/lite/core/pro), reused across mo+yr -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->
  - [x] verify: 7 rows seeded, UUID match `packageMasterData`, composite UNIQUE (code, billing_interval) enforced <!-- done: 2026-05-04 — comp_packages=7 rows; uq_comp_packages_code_billing_interval index exists -->

- [x] **P1.3** สร้าง migration M3 — `20260504-1002-create-comp_package_features.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable พร้อม FK + composite PK + Cartesian seed (Q1=C) <!-- done: 2026-05-04 -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->

- [x] **P1.4** สร้าง migration M4 — `20260504-1003-create-comp_addons.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + seed 9 addons จาก `addonsMasterData` (HappyBeacon set is_quantifiable=true, max_quantity=100) <!-- done: 2026-05-04. Schema follow-up 2026-05-04: UNIQUE(code) → UNIQUE(code, billing_interval) composite; seed code = PackageAddonsEnumCode raw values (happybeacon/e_slip/evaluation/happy_pm/e_signature), reused across mo+yr -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->

- [x] **P1.5** สร้าง migration M5 — `20260504-1004-create-comp_addon_features.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + FK + composite PK <!-- done: 2026-05-04 -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->

- [x] **P1.6** สร้าง migration M6 — `20260504-1005-create-comp_company_features.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + UNIQUE(company_id, feature_id) + indexes (Q4=B: feature_id RESTRICT) <!-- done: 2026-05-04 -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->
  <!-- fix: 2026-05-04 — duplicate company_id resolved via table.foreign() + raw ALTER NOT NULL (Option A); tableTemplate ที่ตำแหน่ง column auto-add ทำให้ก่อนหน้านี้ redeclare ซ้ำเป็น 42701 -->

- [x] **P1.7** สร้าง migration M7 — `20260504-1006-create-comp_company_addons.ts` <!-- done: 2026-05-04 -->
  - [x] up: createTable + migrate data จาก `comp_companies.add_ons` (jsonb) ผ่าน `INSERT...SELECT` <!-- done: 2026-05-04 -->
  - [x] down: dropTableIfExists <!-- done: 2026-05-04 -->
  - [x] verify: count match กับ `comp_companies.add_ons` array length <!-- done: 2026-05-04 — old_with_addons=7, new_distinct_companies=7 (match), total rows=17 -->
  <!-- fix: 2026-05-04 — duplicate company_id resolved via table.foreign() + raw ALTER NOT NULL (Option A); tableTemplate ที่ตำแหน่ง column auto-add ทำให้ก่อนหน้านี้ redeclare ซ้ำเป็น 42701 -->

- [x] **P1.8** สร้าง migration M8 — `20260504-1007-alter-comp_companies-package-id-fk.ts` <!-- done: 2026-05-04 -->
  - [x] **Pre-validation**: รัน query ตรวจ `selected_package_uuid` ทุกตัว match `packageMasterData` <!-- done: 2026-05-04 — orphan_count=0 (57 companies all valid) -->
  - [x] up step (a): backfill จาก `selected_package_uuid` → `comp_packages.id` <!-- done: 2026-05-04 -->
  - [x] up step (b): set default Seed สำหรับ legacy row <!-- done: 2026-05-04 -->
  - [x] up step (c): drop FK เก่า + recreate FK ใหม่ → `comp_packages.id` <!-- done: 2026-05-04 -->
  - [x] down: revert FK กลับ const_packages <!-- done: 2026-05-04 -->
  - [x] **CRITICAL** verify: `SELECT COUNT(*) FROM comp_companies cc LEFT JOIN comp_packages cp ON cc.package_id = cp.id WHERE cp.id IS NULL` = 0 <!-- done: 2026-05-04 — orphan_count=0 confirmed post-M8 -->

- [x] **P1.9** Run migration บน dev — `npm run db:migrate:status` แล้ว `npm run db:migrate:latest` <!-- done: 2026-05-04 — batch 33 applied 3 migrations (M6+M7+M8); all M1..M8 confirmed -->
  - [x] ตรวจ all tables created <!-- done: 2026-05-04 — C1: 7/7 feature-management tables exist -->
  - [x] ตรวจ seed data <!-- done: 2026-05-04 — C2: features=14, packages=7, addons=9, package_features=98 (14x7), addon_features=0, company_features=0, company_addons=17 -->
  - [x] ตรวจ FK constraints <!-- done: 2026-05-04 — C6: feature_id=RESTRICT('r'), company_id=CASCADE('c') -->

### Models (Objection.js)

- [x] **P1.10** สร้าง `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.11** สร้าง `compPackages.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.12** สร้าง `compPackageFeatures.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.13** สร้าง `compAddons.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.14** สร้าง `compAddonFeatures.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.15** สร้าง `compCompanyFeatures.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.16** สร้าง `compCompanyAddons.model.ts` <!-- done: 2026-05-04 -->
- [x] **P1.17** ทุก model define `relationMappings` (BelongsToOneRelation, ManyToManyRelation, HasManyRelation) <!-- done: 2026-05-04 -->
- [x] **P1.18** Type interfaces (เช่น `CompFeaturesType`, `CompPackagesType`, `CompPackageFeaturesType`, `CompAddonsType`, `CompAddonFeaturesType`, `CompCompanyFeaturesType`, `CompCompanyAddonsType`) <!-- done: 2026-05-04 -->

---

## Phase 2: Master CRUD (Backend + Frontend)

### Backend — Constants Update

- [x] **P2.1** อัพเดต `happywork-backend/src/constant/compPermission.ts` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม helper `getAvailableMenuKeys(): string[]` — recursive walk ของ PermissionDefault, return leaf paths <!-- done: 2026-05-05 -->

### Backend — Middleware

- [x] **P2.2** สร้าง `happywork-backend/src/middlewares/requireSuperAdmin.middleware.ts` (ถ้ายังไม่มี) <!-- done: 2026-05-05 -->
  - [x] เช็ค `req.user.role === 'super_admin'` → ผ่าน, ไม่ใช่ → throw 403 <!-- done: 2026-05-05 -->

### Backend — Feature Module

- [x] **P2.3** สร้าง `happywork-backend/src/modules/v2/sale-dashboard/feature/feature.interface.ts` <!-- done: 2026-05-05 -->
  - [x] Zod schemas: `featureBaseSchema`, `createFeatureSchema`, `updateFeatureSchema`, `listFeaturesSchema` <!-- done: 2026-05-05 -->
  - [x] TS types: `Feature`, `FeatureWithUsage` <!-- done: 2026-05-05 -->
- [x] **P2.4** สร้าง `feature.repository.ts` <!-- done: 2026-05-05 -->
  - [x] `getFeatureByIdRepository`, `getFeatureByUuidRepository`, `getFeatureByCodeRepository` <!-- done: 2026-05-05 -->
  - [x] `listFeaturesRepository(filter, pagination)` <!-- done: 2026-05-05 -->
  - [x] `createFeatureRepository`, `updateFeatureRepository`, `softDeleteFeatureRepository` <!-- done: 2026-05-05 -->
  - [x] `countFeatureUsageRepository(featureId)` (count packages + addons) <!-- done: 2026-05-05 -->
- [x] **P2.5** สร้าง `feature.service.ts` <!-- done: 2026-05-05 -->
  - [x] `getFeatureListService`, `getFeatureByUuidService` <!-- done: 2026-05-05 -->
  - [x] `createFeatureService` (validate code unique, validate menu_keys exist) <!-- done: 2026-05-05 -->
  - [x] `updateFeatureService`, `deleteFeatureService` (block ถ้ามี usage) <!-- done: 2026-05-05 -->
  - [x] `getAvailableMenuKeysService` <!-- done: 2026-05-05 -->
- [x] **P2.6** สร้าง `feature.adapter.ts` — convert/normalize for response <!-- done: 2026-05-05 -->
- [x] **P2.7** สร้าง `happywork-backend/src/api/v2/sale-dashboard/feature/feature.controller.ts` <!-- done: 2026-05-05 -->
  - [x] `getFeatureListController`, `getFeatureByUuidController` <!-- done: 2026-05-05 -->
  - [x] `createFeatureController`, `updateFeatureController`, `deleteFeatureController` <!-- done: 2026-05-05 -->
  - [x] `getAvailableMenuKeysController` <!-- done: 2026-05-05 -->
- [x] **P2.8** สร้าง `feature.routes.ts` — Express router with middleware chain <!-- done: 2026-05-05 -->
- [x] **P2.9** Register route ใน `src/api/v2/sale-dashboard/index.routes.ts` <!-- done: 2026-05-05 -->

### Backend — Feature Module — Wave B2 Notes (2026-05-05)

- Routes mounted at `/api/v2/sale-dashboard/features` (with leaf path `/menu-keys/available` ลงทะเบียนก่อน `/:featureUuid` ใน Express router เพื่อกัน path collision)
- Middleware chain ทุก route: `validateAccessToken` → `requireSuperAdmin` → `validateRequest`
- Zod schemas register OpenAPI ผ่าน `registerPathSchema` (tags: `Feature Management - V2 Admin`)
- Adapter strips `id` ภายใน — response มี `uuid` เท่านั้น (ตาม master rules §10.2)
- Service ใช้ `errorBadRequest` (400) สำหรับ `CODE_EXISTS`, `INVALID_MENU_KEY`, `FEATURE_IN_USE` (Q3=A: block delete) และ `errorNotFound` (404) สำหรับ uuid ที่หาไม่เจอ
- `countFeatureUsageBatchRepository` เพิ่มเข้ามา (extra) เพื่อเลี่ยง N+1 ตอน list features หลายตัวพร้อมกัน — comment ใน service ระบุเหตุผล (Pre-fetch dueto 1:many Cartesian risk)

### Backend — Package CRUD Module

- [x] **P2.10** สร้าง `happywork-backend/src/modules/v2/sale-dashboard/packageCrud/packageCrud.interface.ts` <!-- done: 2026-05-05 -->
- [x] **P2.11** สร้าง `packageCrud.repository.ts` <!-- done: 2026-05-05 -->
  - [x] CRUD บน comp_packages <!-- done: 2026-05-05 -->
  - [x] `getPackageFeaturesRepository(packageId)`, `replacePackageFeaturesRepository(packageId, featureIds[])` (transaction) <!-- done: 2026-05-05 -->
  - [x] `countPackageUsageRepository` (count comp_companies) <!-- done: 2026-05-05 -->
- [x] **P2.12** สร้าง `packageCrud.service.ts` <!-- done: 2026-05-05 -->
  - [x] CRUD services <!-- done: 2026-05-05 -->
  - [x] `replacePackageFeaturesService` <!-- done: 2026-05-05 -->
  - [x] `getPackageEffectiveMenusService` (รวม menu_keys ของ features, dedupe) <!-- done: 2026-05-05 -->
- [x] **P2.13** สร้าง `packageCrud.adapter.ts` <!-- done: 2026-05-05 -->
- [x] **P2.14** สร้าง `src/api/v2/sale-dashboard/packageCrud/packageCrud.controller.ts` <!-- done: 2026-05-05 -->
  - [x] CRUD controllers <!-- done: 2026-05-05 -->
  - [x] `replacePackageFeaturesController`, `getPackageEffectiveMenusController` <!-- done: 2026-05-05 -->
- [x] **P2.15** สร้าง `packageCrud.routes.ts` + register <!-- done: 2026-05-05 -->

### Backend — Package CRUD Module — Wave B3 Notes (2026-05-05)

- Routes mounted at `/api/v2/sale-dashboard/packages` (specific leaf paths `/:uuid/features` + `/:uuid/effective-menus` ลงทะเบียนก่อน `/:packageUuid` ใน Express router เพื่อกัน path collision เหมือน B2)
- Composite UNIQUE(code, billing_interval) — ตรวจ uniqueness ที่ service ผ่าน `getPackageByCodeAndBillingIntervalRepository` (ไม่ตรวจ code อย่างเดียว) + handle `null` billing_interval ด้วย `whereNull`
- `replacePackageFeaturesRepository` ใช้ `CompPackageFeatures.transaction(async (trx) => ...)` — delete + insert ใน 1 trx, rollback อัตโนมัติถ้า fail
- `replacePackageFeaturesService` validate ทุก feature uuid ผ่าน `getActiveFeaturesByUuidsRepository` (batch WHERE IN) ก่อนแก้ trx → defense in depth นอกจาก FK RESTRICT (Q4=B)
- `getPackageEffectiveMenusService` รวม `menu_keys` ของทุก feature → dedupe ผ่าน `Set<string>` → sort
- List N+1 prevention: `countPackageFeaturesBatchRepository` + `countPackageUsageBatchRepository` ใช้ WHERE IN + GROUP BY (Pre-fetch — comment ใน service)
- `deletePackageService` block ถ้า `comp_companies.package_id` ยังมี reference (Q3=A pattern เดียวกับ feature)

### Backend — Package UUID Update (Added 2026-05-05 — Stripe sync)

> **Why:** uuid ของ package ต้อง match Stripe product ID. ถ้า Stripe ID เปลี่ยน (เช่น re-create product) admin ต้องปรับ uuid ฝั่ง DB ตาม. Endpoint dedicated `PATCH /packages/:packageUuid/identifier` แทนการรวมใน update ปกติ (REST anti-pattern: ห้ามแก้ resource id ผ่าน body ของ resource เดียวกัน)

- [x] **P2.15a** อัพเดต `packageCrud.interface.ts` — Zod schema `updatePackageUuidSchema` (params + body `{newUuid: z.string().uuid()}`); register OpenAPI <!-- done: 2026-05-05 -->
- [x] **P2.15b** อัพเดต `packageCrud.repository.ts` — `updatePackageUuidRepository(currentUuid, newUuid, updatedBy)` ใช้ transaction; UPDATE single row; throw ถ้า affected rows = 0 <!-- done: 2026-05-05 -->
- [x] **P2.15c** อัพเดต `packageCrud.service.ts` — `updatePackageUuidService(currentUuid, newUuid, actor)` validate format + ≠ current + unique; **grep หา denormalized `package_uuid` references** (เช่น `comp_companies.selected_package_uuid` legacy หรือ field อื่นที่ cache uuid) และถ้าเจอ → update ใน trx เดียวกัน หรือรายงาน Lead ถ้าไม่ชัด <!-- done: 2026-05-05 -->
- [x] **P2.15d** อัพเดต `packageCrud.controller.ts` + `packageCrud.routes.ts` — `updatePackageUuidController`; mount `PATCH /:packageUuid/identifier` ก่อน `PUT /:packageUuid` (path collision avoidance — เนื่องจาก `:packageUuid` greedy) <!-- done: 2026-05-05 -->
- [x] **P2.15e** ตรวจ TS pass + prettier + ทำ verification grep ว่าไม่มี code path อื่นแอบเก็บ `package.uuid` แบบ denormalized + ติ๊ก checklist <!-- done: 2026-05-05 -->

### Backend — Addon Module

- [x] **P2.16** สร้าง `happywork-backend/src/modules/v2/sale-dashboard/addon/addon.interface.ts` <!-- done: 2026-05-05 -->
  - [x] Validation: is_quantifiable=true → max_quantity required (Zod refine + service defense in depth) <!-- done: 2026-05-05 -->
- [x] **P2.17** สร้าง `addon.repository.ts` <!-- done: 2026-05-05 -->
  - [x] CRUD + `getAddonFeaturesRepository`, `replaceAddonFeaturesRepository` (transaction) <!-- done: 2026-05-05 -->
- [x] **P2.18** สร้าง `addon.service.ts` <!-- done: 2026-05-05 -->
  - [x] CRUD + replaceAddonFeaturesService + delete validation (block ถ้ามี company purchase) <!-- done: 2026-05-05 -->
- [x] **P2.19** สร้าง `addon.adapter.ts` <!-- done: 2026-05-05 -->
- [x] **P2.20** สร้าง `src/api/v2/sale-dashboard/addon/addon.controller.ts` + `addon.routes.ts` <!-- done: 2026-05-05 -->
- [x] **P2.21** Register route <!-- done: 2026-05-05 -->

### Backend — Addon Module — Wave B3 Notes (2026-05-05)

- Routes mounted at `/api/v2/sale-dashboard/addons` (leaf path `/:uuid/features` ลงทะเบียนก่อน `/:addonUuid`)
- Composite UNIQUE(code, billing_interval) — pattern เดียวกับ package
- `is_quantifiable=true → maxQuantity > 0`: enforce 2 ชั้น
  1. Zod schema (create) ใช้ `.refine()` ครอบทั้ง object
  2. Service `validateQuantifiableRule` ครอบทั้ง create + update (รับ partial input คำนวณ next state)
  - update schema ไม่ refine แล้ว (เพราะ `partial()` + refine ใน Zod ทำงานกับทั้งคู่ไม่ครบ field) — ใช้ service เป็นแหล่ง truth
- `replaceAddonFeaturesRepository` ใช้ `CompAddonFeatures.transaction(...)` — delete + insert ใน 1 trx
- `deleteAddonService` block ถ้ามี `comp_company_addons` referencing (count ทุก row, ไม่ filter expires_at — เพราะถ้า addon ถูก delete แล้ว record purchase ก็ orphan)

### Backend — Tests

- [x] **P2.22** Unit tests: `feature.service.spec.ts` — 16 tests PASS (getAvailableMenuKeys, getByUuid, create, update, delete — error paths + happy path ครบ) <!-- done: 2026-05-05 -->
  - File: `tests/unit/modules/v2/sale-dashboard/feature/feature.service.test.ts`
- [x] **P2.23** Unit tests: `packageCrud.service.spec.ts` — 20 tests PASS (create, update, delete, replaceFeatures, updateUuid — composite uniqueness + 403 gating ครบ) <!-- done: 2026-05-05 -->
  - File: `tests/unit/modules/v2/sale-dashboard/packageCrud/packageCrud.service.test.ts`
- [x] **P2.24** Unit tests: `addon.service.spec.ts` — 20 tests PASS (create, update, delete, replaceFeatures — isQuantifiable rule, null billingInterval ครบ) <!-- done: 2026-05-05 -->
  - File: `tests/unit/modules/v2/sale-dashboard/addon/addon.service.test.ts`
- [x] **P2.25** Integration test: ใช้ super_admin token CRUD ครบ flow — test file เขียนครบ (19 test cases: 401/200/201/400 gating ทั้ง 3 modules) <!-- done: 2026-05-05 -->
  - File: `tests/integration/sale-dashboard/feature-management.integration.spec.ts`
  - Blocker: OOM เกิดขึ้นเมื่อรัน local (app module graph ใหญ่ เกิน default Node heap ใน test process) — ต้องรัน CI pipeline หรือเพิ่ม `--max-old-space-size=4096` ใน jest config สำหรับ integration tests

### Frontend — Setup

- [x] **F2.1** อัพเดต `happywork-sale-cms/src/utils/rbac.ts` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม Resource: `features`, `addons`, `company_features` (deviation: `packages` already existed; left existing PERMISSIONS unchanged to avoid breaking `package-config-feature-view.tsx`) <!-- done: 2026-05-05 -->
  - [x] อัพเดต PERMISSIONS matrix (super_admin only — admin/manager/viewer = `[]` for the 3 new resources) <!-- done: 2026-05-05 -->
- [x] **F2.2** อัพเดต `happywork-sale-cms/src/routes/paths.ts` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม `paths.dashboard.featureManagement.{root,create,detail,edit}` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม `paths.dashboard.packageManagement.*` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม `paths.dashboard.addonManagement.*` <!-- done: 2026-05-05 -->
- [x] **F2.3** อัพเดต `happywork-sale-cms/src/layouts/dashboard/config-navigation.tsx` <!-- done: 2026-05-05 -->
  - [x] เพิ่ม section "Master Data" (super_admin conditional) มี 3 menu items <!-- done: 2026-05-05 -->
- [x] **F2.4** อัพเดต `happywork-sale-cms/src/locales/langs/{en,th}.json` <!-- done: 2026-05-05 -->
  - [x] section `featureManagement`, `packageManagement`, `addonManagement`, `companyFeatures`, `menu.master_data` (parity verified en + th) <!-- done: 2026-05-05 -->

### Frontend — Reusable Components

- [x] **F2.5** สร้าง `happywork-sale-cms/src/components/feature-management/multilingual-text-field.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.6** สร้าง `menu-keys-select.tsx` — multi-select tree (group by main category) <!-- done: 2026-05-05 -->
- [x] **F2.7** สร้าง `menu-tree-preview.tsx` — read-only tree preview <!-- done: 2026-05-05 -->
- [x] **F2.8** สร้าง `feature-source-badge.tsx` <!-- done: 2026-05-05 -->

### Frontend — Redux Slice (Feature Management)

> **Note (Wave F3, 2026-05-05):** ไฟล์เดิม `actions/sale-dashboard-feature-management.ts` (และ reducer/saga คู่กัน) ถูกใช้โดย legacy per-company feature-toggle slice อยู่แล้ว — มี consumer คือ `sections/package-config-feature/*`, `leads/lead-detail/components/lead-information-tab.tsx`, `customer/customer-detail/components/customer-information-tab.tsx`, และ `package-config-feature/components/edit-package-dialog.tsx`. เพื่อกัน break Wave F3 จึงสร้างไฟล์ Master Data CRUD slice ใหม่ด้วย suffix `-master`: `sale-dashboard-feature-master.ts`, reducer key `saleDashboardFeatureMaster`, hook `useSaleDashboardFeatureMaster`, action prefix `sdGetFeatureList* / sdGetFeatureByUuid* / sdCreateFeature* / sdUpdateFeature* / sdDeleteFeature* / sdGetAvailableMenuKeys*`. การ rename refactor legacy slice ภายหลัง defer ไว้ก่อน — ใช้แนวเดียวกันกับ F2.15 (`-package-master`) และ F2.16 (`-addon-master`).

- [x] **F2.9** สร้าง `happywork-sale-cms/src/types/store/sale-dashboard-feature-master.ts` (renamed `-master` to avoid collision; registered ใน `types/store/index.ts`) <!-- done: 2026-05-05 -->
- [x] **F2.10** สร้าง `src/store/services/feature-management-request.ts` (typed wrappers สำหรับ `getList / getByUuid / create / update / delete / getAvailableMenuKeys`) + URL groups `adminFeatures*` ใน `services/sale-dashboard.ts` <!-- done: 2026-05-05 -->
- [x] **F2.11** สร้าง `src/store/actions/sale-dashboard-feature-master.ts` + registered ใน `actions/index.ts` <!-- done: 2026-05-05 -->
- [x] **F2.12** สร้าง `src/store/reducers/sale-dashboard-feature-master.ts` + register ใน combineReducers (slot `saleDashboardFeatureMaster`) <!-- done: 2026-05-05 -->
- [x] **F2.13** สร้าง `src/store/sagas/sale-dashboard-feature-master.ts` + register ใน rootSaga (`watchSaleDashboardFeatureMaster`) <!-- done: 2026-05-05 -->
- [x] **F2.14** สร้าง hook `useSaleDashboardFeatureMaster` (export จาก reducer file) <!-- done: 2026-05-05 -->

### Frontend — Redux Slice (Package Management)

- [x] **F2.15** Mirror P2.9–P2.14 สำหรับ package-management — ไฟล์: `types/store/sale-dashboard-package-master.ts`, `services/package-management-request.ts` (รวม `replacePackageFeatures` + `getPackageEffectiveMenus`), `actions/sale-dashboard-package-master.ts`, `reducers/sale-dashboard-package-master.ts` (slot `saleDashboardPackageMaster`, hook `useSaleDashboardPackageMaster`), `sagas/sale-dashboard-package-master.ts` (`watchSaleDashboardPackageMaster`) — ทุก index.ts registered <!-- done: 2026-05-05 -->

### Frontend — Redux Slice (Addon Management)

- [x] **F2.16** Mirror P2.9–P2.14 สำหรับ addon-management — ไฟล์: `types/store/sale-dashboard-addon-master.ts`, `services/addon-management-request.ts` (รวม `replaceAddonFeatures`), `actions/sale-dashboard-addon-master.ts`, `reducers/sale-dashboard-addon-master.ts` (slot `saleDashboardAddonMaster`, hook `useSaleDashboardAddonMaster`), `sagas/sale-dashboard-addon-master.ts` (`watchSaleDashboardAddonMaster`) — ทุก index.ts registered <!-- done: 2026-05-05 -->

### Wave F3 Notes — Q7 Rename (Decision B, 2026-05-05)

> **Q7 Resolution (Decision B — rename-only, no logic change):** Legacy per-company slice `sale-dashboard-feature-management` ถูก rename เป็น `sale-dashboard-package-config-feature` (ตรง section folder semantic) เพื่อปลดล็อค slot key `saleDashboardFeatureManagement` ให้ Master Data slice ของ Wave F3 ใช้ตามชื่อ spec เดิม. F3 slices ทั้ง 3 ตัวจึงถอด `-master` suffix ออก. **Pure file/symbol rename — ไม่มีการเปลี่ยน logic, action types, reducer behavior, หรือ saga effect.**

- [x] **Q7.1** Rename legacy slice `sale-dashboard-feature-management` → `sale-dashboard-package-config-feature` (types + actions + reducers + sagas) — slot key `saleDashboardFeatureManagement` → `saleDashboardPackageConfigFeature`, namespace `SaleDashboardFeatureManagement` → `SaleDashboardPackageConfigFeature`, watcher `watchSaleDashboardFeatureManagement` → `watchSaleDashboardPackageConfigFeature`. Action types (`sdGetFeaturesRequest`, `sdToggleFeatureRequest`, `sdGetPackageInfoRequest`, `sdGetPackageOptionsRequest`, `sdPreviewChangePackage*`, `sdChangePackage*`) ปล่อยไว้เหมือนเดิมเพราะ 4 production views ยังใช้อยู่ <!-- done: 2026-05-05 -->
- [x] **Q7.2** Update imports ใน 4 production views ที่ใช้ legacy slot: `sections/package-config-feature/package-config-feature-view.tsx`, `sections/package-config-feature/components/edit-package-dialog.tsx` (รวม namespace ref `TStore.SaleDashboardPackageConfigFeature.PackageOption|PackageInfo`), `sections/leads/lead-detail/components/lead-information-tab.tsx`, `sections/customer/customer-detail/components/customer-information-tab.tsx` <!-- done: 2026-05-05 -->
- [x] **Q7.3** Drop `-master` suffix จาก F3 slices: `sale-dashboard-{feature,package,addon}-master.ts` → `sale-dashboard-{feature,package,addon}-management.ts` (types + actions + reducers + sagas). Slot keys: `saleDashboardFeatureMaster` → `saleDashboardFeatureManagement`, `saleDashboardPackageMaster` → `saleDashboardPackageManagement`, `saleDashboardAddonMaster` → `saleDashboardAddonManagement`. Hooks: `useSaleDashboardFeatureMaster` → `useSaleDashboardFeatureManagement`, `useSaleDashboardPackageMaster` → `useSaleDashboardPackageManagement`, `useSaleDashboardAddonMaster` → `useSaleDashboardAddonManagement`. Watchers: `watchSaleDashboardFeatureMaster` → `watchSaleDashboardFeatureManagement` (และ Package + Addon เช่นกัน). Namespaces ใน `types/store/index.ts` rename ตาม. Action types ของ F3 ปล่อยไว้เหมือนเดิม (ไม่มี `-master` infix อยู่แล้ว) <!-- done: 2026-05-05 -->
- [x] **Q7.4** Update register imports ทุก index file: `types/store/index.ts`, `store/actions/index.ts`, `store/reducers/index.ts`, `store/sagas/index.ts` <!-- done: 2026-05-05 -->
- [x] **Q7.5** Verify TS pass: `rm tsconfig.tsbuildinfo && npx tsc --noEmit` exit 0 หลัง rename ครบ <!-- done: 2026-05-05 -->
- [x] **Q7.6** Run `npx prettier --write` กับไฟล์ที่ rename + ไฟล์ importer (3 ไฟล์ reformatted, 24 ไฟล์ already conformant) <!-- done: 2026-05-05 -->

### Frontend — Page A: Feature Management

- [x] **F2.17** สร้าง `happywork-sale-cms/src/sections/feature-management/feature.enum.ts` <!-- done: 2026-05-05 -->
- [x] **F2.18** สร้าง `src/sections/feature-management/utils/feature-form.ts` (Yup schema) <!-- done: 2026-05-05 -->
- [x] **F2.19** สร้าง `src/sections/feature-management/components/feature-table.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.20** สร้าง `src/sections/feature-management/components/feature-form.tsx` (shared create+edit) <!-- done: 2026-05-05 -->
- [x] **F2.21** สร้าง `feature-delete-dialog.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.22** สร้าง `feature-status-option.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.23** สร้าง `feature-usage-info.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.24** สร้าง `feature-list-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.25** สร้าง `feature-create-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.26** สร้าง `feature-detail-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.27** สร้าง `feature-edit-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.28** สร้าง pages: <!-- done: 2026-05-05 -->
  - [x] `src/app/dashboard/feature-management/page.tsx` <!-- done: 2026-05-05 -->
  - [x] `src/app/dashboard/feature-management/create/page.tsx` <!-- done: 2026-05-05 -->
  - [x] `src/app/dashboard/feature-management/[id]/page.tsx` <!-- done: 2026-05-05 -->
  - [x] `src/app/dashboard/feature-management/[id]/edit/page.tsx` <!-- done: 2026-05-05 -->

### Wave F4 Notes (2026-05-05)

- All section files + components + views + pages created under `src/sections/feature-management/` + `src/app/dashboard/feature-management/`
- Reusable components leveraged: `MultilingualTextField`, `MenuKeysSelect`, `MenuTreePreview` (from F2 wave)
- Redux slice: uses existing `useSaleDashboardFeatureManagement` hook (F3 wave) — no slice changes
- RBAC: `<Can resource="features" action="...">` gating on create/edit/delete buttons + page-level guards
- Form library: react-hook-form + Yup resolver via `@hookform/resolvers/yup` (matches survey + admin-users pattern)
- Delete dialog: re-fetches usage from detail endpoint on open → blocks confirm button if `packageCount + addonCount > 0`
- i18n parity: 53 `featureManagement.*` keys in en + th (no drift). New keys added: `subtitle`, `section.*`, `form.{name,description,create_action}`, `table.*`, `filter.*`, `menu_keys.empty`, `usage.*`, `create.success`, `edit.success`, `delete.{success,usage_error}`, `validation.{sort_order_number,sort_order_min}`
- TypeScript: `npx tsc --noEmit` reports zero errors in feature-management scope (only pre-existing `addon-management/utils/addon-form.ts` error which is in F6 parallel wave's scope)
- Prettier: ran `npx prettier --write` on all 11 new files + updated locales — all conformant
- Tests: deferred per task instruction (Wave QA scope)

### Frontend — Page B: Package Management

- [x] **F2.29** สร้าง section files (Mirror Page A pattern) <!-- done: 2026-05-05 -->
  - [x] `package.enum.ts`, `utils/package-form.ts` <!-- done: 2026-05-05 -->
  - [x] components: `package-table.tsx`, `package-form.tsx`, `package-delete-dialog.tsx` <!-- done: 2026-05-05 -->
  - [x] `package-features-section.tsx` (อยู่ใน detail view) <!-- done: 2026-05-05 -->
  - [x] `package-features-edit-dialog.tsx` <!-- done: 2026-05-05 -->
  - [x] `package-effective-menus-preview.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.30** สร้าง views: `package-list-view.tsx`, `package-create-view.tsx`, `package-detail-view.tsx`, `package-edit-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.31** สร้าง pages: list, create, [id], [id]/edit <!-- done: 2026-05-05 -->

### Wave F5 Notes (2026-05-05)

- All section files + components + views + pages created under `src/sections/package-management/` + `src/app/dashboard/package-management/`
- **Pre-step F5.0:** Added `master_packages` resource to `src/utils/rbac.ts` (Q6) — separate from legacy `packages` (untouched). super_admin gets `view/create/edit/delete/manage`; admin/manager/viewer get `[]`.
- **B4-FE integration:** `<PackageUuidEditDialog>` mounted in `package-detail-view.tsx` toolbar (gated `<Can resource="master_packages" action="manage">`). On success → `router.replace(paths.dashboard.packageManagement.detail(updated.uuid))` since URL slug = uuid.
- Reusable components leveraged: `MultilingualTextField` (F2 wave) + `MenuTreePreview` (F2 wave) for `<PackageEffectiveMenusPreview>`
- Redux: uses existing `useSaleDashboardPackageManagement` hook (F3 + B4-FE waves) — no slice changes besides type extension (added `priceAmount`, `currency`, `userLimitMin/Max`, `isContactUs`, `shortName: string` to `Package` type to match backend B3 contract; `billingInterval` aligned to `'month'|'year'` per backend zod schema)
- RBAC: `<Can resource="master_packages" action="...">` gating on create/edit/delete/manage buttons + page-level guards via `<Can fallback={...}>`
- Form library: react-hook-form + Yup resolver via `@hookform/resolvers/yup` (matches survey + admin-users + Page A pattern)
- Form schema: `code` regex `^[a-z0-9_]+$`, `priceAmount ≥ 0`, `currency` 3-letter, `userLimitMax ≥ userLimitMin` cross-field validation
- Delete dialog: relies on backend `deletePackageService` rejection when companies reference the package (error bubbled via `submitError`); shows inline warning before confirm
- Features management: `<PackageFeaturesEditDialog>` loads feature list via `useSaleDashboardFeatureManagement.getFeatureList({statusType:'active', limit:200})`, dispatches `replacePackageFeatures` on save
- i18n parity: ~80 `packageManagement.*` keys in en + th (no drift, verified via flatten script). Added: `list_subtitle`, `empty`, `not_found`, `create_success`, `update_success`, `delete_success`, `section.{identification,pricing,user_limits,display}`, `form.{uuid,short_name,name,description,billing_interval,price_amount,currency,user_limit_*,is_recommend,is_contact_us,sort_order_*}`, `billing.{month,year,one_time}`, `status.{all,active,archived}`, `flag.{recommend,contact_us}`, `table.*`, `features.*`, `effective_menus.*`, `delete_warning_inline`, `validation.{code_max,short_name_max,price_*,currency_*,user_limit_*,sort_order_*}`, `uuidEdit.button`. Also added `common.permission_denied_{title,message}` for page-level fallback.
- TypeScript: `npx tsc --noEmit` reports zero errors in package-management scope (only pre-existing 3 errors in addon-management/Page C — out of scope)
- Prettier: ran `npx prettier --write` on all 20 modified/new files — all conformant
- Tests: deferred per task instruction

### Frontend — Page C: Addon Management

- [x] **F2.32** สร้าง section files <!-- done: 2026-05-05 -->
  - [x] `addon.enum.ts`, `utils/addon-form.ts` <!-- done: 2026-05-05 -->
  - [x] components: `addon-table.tsx`, `addon-form.tsx`, `addon-delete-dialog.tsx` <!-- done: 2026-05-05 -->
  - [x] `addon-quantifiable-fields.tsx` (conditional max_quantity) <!-- done: 2026-05-05 -->
  - [x] `addon-features-section.tsx`, `addon-features-edit-dialog.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.33** สร้าง views: `addon-list-view.tsx`, `addon-create-view.tsx`, `addon-detail-view.tsx`, `addon-edit-view.tsx` <!-- done: 2026-05-05 -->
- [x] **F2.34** สร้าง pages <!-- done: 2026-05-05 -->

### Wave F6 Notes (2026-05-05)

- Section: `src/sections/addon-management/` + reusable components in `src/sections/addon-management/components/`
- Views + App Router pages wired with `<Can resource="addons">` guards (view / create / edit) and i18n via `addonManagement.*` namespace
- Yup schema (`createAddonSchema`) enforces `is_quantifiable=true → max_quantity ≥ 1` with conditional `Yup.when("isQuantifiable", …)` — defense in depth (backend also enforces)
- `addon-quantifiable-fields.tsx`: switch toggles `isQuantifiable` and resets `maxQuantity` to `null` when off so payload builders ship the correct shape
- `addon-delete-dialog.tsx`: refetches addon detail to get `companyCount` (or `usage.companies`) and blocks confirmation while `companyCount > 0`; surfaces backend-side block via toast / error banner if FE check is bypassed
- `addon-features-edit-dialog.tsx`: lazy-loads master features via `useSaleDashboardFeatureManagement().getFeatureList({ statusType: "active", limit: 200 })`; multi-select with search; dispatches `replaceAddonFeatures(uuid, featureUuids[])` and detects success transition via `wasReplacingRef`
- Type extension (deviation): `src/types/store/sale-dashboard-addon-management.ts` extended with `priceAmount?: number`, `currency?: string`, `companyCount?: number` fields on `Addon` + corresponding optional fields on Create/Update payloads. The Q7 slice originally stopped after `billingInterval` (with `// ... etc`) — these additions are type-only, no logic change in actions/reducer/saga/hook. Re-exported `Multilingual` / `Pagination` / `SortOrder` so addon section files don't reach into the feature-management types module
- Verification: `npx tsc --noEmit` passes (clean exit 0); `npx prettier --write` applied to all 19 created/modified files (12 reformatted, 7 already conformant); i18n parity confirmed (89 `addonManagement.*` keys in en + th, no drift)

### Q8 Fix — Addon `billingInterval` value rename (2026-05-05)

> **Q8 Resolution (Pure value rename, no logic change):** Frontend `AddonBillingInterval` was wrongly typed as `'monthly' | 'yearly' | 'one_time'` during F3 draft (Wave F3 stopped at `billingInterval` with `// ... etc`). Backend B3 Zod schema (`happywork-backend/src/modules/v2/admin/addon/addon.interface.ts` lines 34, 81, 113) is `z.enum(['month', 'year']).nullable()` — submitting the form would have hit a 400 from Zod. Fixed to match backend: literals `'month' | 'year'` + `null` for one-time billing (drops the `'one_time'` literal since `null` carries the semantic). Mirrors the Package pattern (F5).

- [x] **Q8.1** Rename type `AddonBillingInterval` in `src/types/store/sale-dashboard-addon-management.ts` from `'monthly' | 'yearly' | 'one_time'` → `'month' | 'year'`. `null` continues to mean one-time (already `nullable` on the wire). <!-- done: 2026-05-05 -->
- [x] **Q8.2** Replace `ADDON_BILLING_INTERVALS` array + `ADDON_BILLING_INTERVAL_TRANSLATION_KEY` map in `src/sections/addon-management/addon.enum.ts` with `ADDON_BILLING_INTERVAL_OPTIONS` (3 options: `value:""` one-time, `value:"month"`, `value:"year"`) and `AddonBillingFormValue = '' | 'month' | 'year'`. Default form value flipped to `""` (one-time). Mirrors `PACKAGE_BILLING_INTERVAL_OPTIONS` shape in `package.enum.ts`. <!-- done: 2026-05-05 -->
- [x] **Q8.3** Update Yup schema in `src/sections/addon-management/utils/addon-form.ts` — `billingInterval` accepts `["", "month", "year"]` via `Yup.mixed<AddonBillingFormValue>().oneOf(...)`. Update `mapAddonToFormValues` to map backend `null` → form `""`. Update `buildCreateAddonPayload` + `buildUpdateAddonPayload` to map form `""` → wire `null` (Backend `nullable()` semantics). <!-- done: 2026-05-05 -->
- [x] **Q8.4** Update `src/sections/addon-management/components/addon-form.tsx` Select to iterate `ADDON_BILLING_INTERVAL_OPTIONS` (3 items: One-time / Monthly / Yearly). <!-- done: 2026-05-05 -->
- [x] **Q8.5** Update display logic in `src/sections/addon-management/components/addon-table.tsx` and `src/sections/addon-management/addon-detail-view.tsx`: when `billingInterval` is null/undefined → show `addonManagement.billing.one_time`; otherwise look up by month/year. <!-- done: 2026-05-05 -->
- [x] **Q8.6** Rename i18n keys under `addonManagement.billing` in `src/locales/langs/{en,th}.json`: `monthly` → `month`, `yearly` → `year`. Keep `one_time` (used as label for null). 89-key parity preserved (en + th equal). Did not touch `companies.monthly/yearly` (line ~462), `leads.monthly/yearly` (line ~506), or `packageManagement.billing.*` — those are separate scopes. <!-- done: 2026-05-05 -->
- [x] **Q8.7** Verify `npx tsc --noEmit` exits 0; run `npx prettier --write` on the 8 modified files (all already conformant). i18n parity verified via flatten script — 89 `addonManagement.*` keys in en + th, no drift. <!-- done: 2026-05-05 -->

### Bug Fix — Runtime errors on Master Data pages (2026-05-05)

> **Bug #1 (RSC):** `Error: This function is not supported in React Server Components ... @ useDispatch` — 3 of the 12 new pages under `src/app/dashboard/feature-management/**/page.tsx` were authored as Server Components (no `"use client"`) but rendered `<Can>` which calls `usePermissions()` → `useSaleDashboardAuth()` → `useDispatch`. The Package and Addon pages already had `"use client"`; the Feature pages diverged because they exported `metadata` (which forced server component shape).
>
> **Bug #2 (Maximum update depth):** F4/F5/F6 hooks (`useSaleDashboardFeatureManagement`, `useSaleDashboardPackageManagement`, `useSaleDashboardAddonManagement` in `src/store/reducers/`) returned a fresh action-handler object every render instead of `useCallback`-wrapped refs. Consumers like `feature-list-view`, `feature-create-view`, `feature-edit-view`, etc. listed those handlers (e.g. `getFeatureList`, `getAvailableMenuKeys`, `clearError`) in `useEffect` dep arrays — every render produced new refs → effect fired → `dispatch` updated state → re-render → new refs → infinite loop. The MUI `setFilled / checkDirty` trace was just where React detected the loop. Existing repo pattern (`src/hooks/use-sale-dashboard.ts` for Auth/Leads) wraps every dispatch in `useCallback([dispatch])`; F4/F5/F6 hooks did not follow this pattern. Fix: wrap every action in `useCallback`.

- [x] **BugFix.1** Add `"use client"` to 3 Feature pages so `<Can>` (Redux hook) can mount: `src/app/dashboard/feature-management/page.tsx`, `src/app/dashboard/feature-management/create/page.tsx`, `src/app/dashboard/feature-management/[id]/edit/page.tsx`. Removed `export const metadata` (incompatible with `"use client"`); for the edit page replaced `params: { id }` server prop with `useParams()` (matches the existing `feature-management/[id]/page.tsx` pattern). Package and Addon pages already had `"use client"` — left untouched. <!-- done: 2026-05-05 -->
- [x] **BugFix.2** Wrap every dispatch in `useCallback([dispatch])` inside the 3 hooks `useSaleDashboardFeatureManagement` / `useSaleDashboardPackageManagement` / `useSaleDashboardAddonManagement` (`src/store/reducers/sale-dashboard-{feature,package,addon}-management.ts`). Stable refs prevent useEffect dep churn → kills the FormControl/InputBase `setFilled` re-render loop. Mirrors the pattern in `src/hooks/use-sale-dashboard.ts`. <!-- done: 2026-05-05 -->
- [x] **BugFix.3** Verify `npx tsc --noEmit` exits 0; run `npx prettier --write` on 6 modified files (all already conformant). Dev server (`next dev -p 4001`) returns HTTP 200 on `/dashboard/{feature,package,addon}-management` and `…/create` (Bug #1 RSC error gone — page no longer throws during SSR). Bug #2 fix is structural (stable hook refs); browser-side runtime verification deferred to user since Playwright is not installed in this environment. <!-- done: 2026-05-05 -->

### URL Fix — Drop `/admin` from Phase 2 Master CRUD URL builders (2026-05-05)

> **Bug:** Frontend URL builders in `src/store/services/sale-dashboard.ts` for the Phase 2 Master Data CRUD (`adminFeatures`, `adminFeatureDetail`, `adminFeatureAvailableMenuKeys`, `adminPackages`, `adminPackageDetail`, `adminPackageFeatures`, `adminPackageEffectiveMenus`, `adminPackageIdentifier`, `adminAddons`, `adminAddonDetail`, `adminAddonFeatures`) targeted `/api/v2/admin/{features|packages|addons}/...` → 404 on every Phase 2 master CRUD call. Backend Express routing (verified by Lead) mounts `v2AdminRouter` at `/api/v2` (not `/api/v2/admin`); actual paths are `/api/v2/features`, `/api/v2/packages`, `/api/v2/addons` (and nested children). Pre-existing subscription URLs at lines 72-76 (`/api/v2/admin/subscription/*`) intentionally untouched — out of Phase 2 scope.
>
> **Builder symbols are unchanged** (still named `adminFeatures` etc.) so all consumers via `feature-management-request.ts`, `package-management-request.ts`, `addon-management-request.ts` continue to type-check. Only the URL string values changed.

- [x] **URLFix.1** Edit `src/store/services/sale-dashboard.ts` lines 100-125: drop `/admin` segment from all 11 Phase 2 URL builders (3 feature + 5 package + 3 addon) → `/api/v2/{features|packages|addons}/...`. Update inline section comments to note backend mounts `v2AdminRouter` at `/api/v2`. <!-- done: 2026-05-05 -->
- [x] **URLFix.2** Grep verify no other inline URL strings under `src/store/services/`, `src/sections/{feature,package,addon}-management/`, `src/components/{feature,package}-management/` — confirmed all hits outside `sale-dashboard.ts` are JSDoc/comments only (request files use `saleDashboardService().adminFeatures` etc.), so the single-file edit propagates everywhere. <!-- done: 2026-05-05 -->
- [x] **URLFix.3** `npx tsc --noEmit` exits 0; `npx prettier --write src/store/services/sale-dashboard.ts` — already conformant. Dev server on port 4001 returns HTTP 200 on `/dashboard/feature-management` (page renders; full network smoke test for the new Phase 2 URLs deferred to user — backend OpenAPI basePath fix is a parallel scope). <!-- done: 2026-05-05 -->

### Move-to-SaleDashboard URL update — Add `/sale-dashboard/` segment (2026-05-05)

> **Why:** User instructed Backend to move 3 Phase 2 modules (`feature` / `packageCrud` / `addon`) from `v2/admin/...` → `v2/sale-dashboard/...` and switch middleware to `validateSaleDashboardAccessToken`. New backend mount is `saleDashboardV2Router` at `/api/v2/sale-dashboard`, so URL paths become `/api/v2/sale-dashboard/{features|packages|addons}/...`. Frontend must re-add the `/sale-dashboard/` segment that was dropped in the previous URL Fix (the previous fix was correct against the older `v2AdminRouter @ /api/v2` mount; this one tracks the new sale-dashboard mount).
>
> **Builder symbols still unchanged** (`adminFeatures`, `adminPackageIdentifier`, etc.) — pure URL string update. Token handling: `sd_method_*` axios wrappers in `src/store/services/sale-dashboard-request.ts` already attach `saleDashboardAccessToken` (Bearer) on every call, which is exactly what the new `validateSaleDashboardAccessToken` middleware expects, so no code change needed for auth.

- [x] **MoveSD.1** Edit `src/store/services/sale-dashboard.ts` lines 100-128: insert `/sale-dashboard/` segment in all 11 Phase 2 URL builders (3 feature + 5 package + 3 addon) → `/api/v2/sale-dashboard/{features|packages|addons}/...`. Update 3 inline section comments to read "API v2 sale-dashboard (super_admin only via sale dashboard token) — Backend mounts saleDashboardV2Router at `/api/v2/sale-dashboard`". Pre-existing subscription URLs at lines 72-76 (`/api/v2/admin/subscription/*`) intentionally left untouched — out of scope. <!-- done: 2026-05-05 -->
- [x] **MoveSD.2** Grep `api/v2/features\|api/v2/packages\|api/v2/addons` across `src/` — zero hits remaining (consumers go through `saleDashboardService().adminFeatures` etc., so the single-file edit propagates). <!-- done: 2026-05-05 -->
- [x] **MoveSD.3** Verified token handling: `sd_method_GET/POST/PUT/PATCH/DELETE` in `src/store/services/sale-dashboard-request.ts` reads `saleDashboardAccessToken` from session/localStorage and attaches `authorization: Bearer ...`. Phase 2 request files (`feature-management-request.ts`, `package-management-request.ts`, `addon-management-request.ts`) all use these wrappers — token automatically forwarded to the new sale-dashboard mount. No auth client change needed. <!-- done: 2026-05-05 -->
- [x] **MoveSD.4** `npx tsc --noEmit` exits 0; `npx prettier --write src/store/services/sale-dashboard.ts` — already conformant. Browser smoke test deferred to user (HMR will pick up automatically since backend agent is moving routes in parallel; user can verify Network tab shows `/api/v2/sale-dashboard/features` once both sides land). <!-- done: 2026-05-05 -->

### Status Filter Label Fix — "Archived" → "Inactive" / "ไม่ใช้งาน" (2026-05-05)

> **Why:** User feedback — filter dropdowns ของทั้ง 3 หน้า (Feature/Package/Addon Management) ต้องการให้แสดง "Active" / "Inactive" (en) และ "ใช้งาน" / "ไม่ใช้งาน" (th) แทน "Active" / "Archived". เป็น display-only change. **Wire value `archived` คงเดิมทั้งหมด** (backend `statusType` enum ห้ามแตะ).
>
> **Approach (Option B — cleaner):** rename TS enum identifier `ARCHIVED` → `INACTIVE` (string literal value `"archived"` คงเดิม) + rename i18n key `packageManagement.status.archived` → `packageManagement.status.inactive`. Feature/Addon ใช้ shared `common.active` / `common.inactive` keys อยู่แล้ว — แค่ rename TS identifier เพื่อ semantic consistency.

- [x] **SFL.1** `src/sections/feature-management/feature.enum.ts` — `FeatureStatus.ARCHIVED` → `FeatureStatus.INACTIVE` (wire value `"archived"` คงเดิม). อัพเดต `FEATURE_STATUS_COLOR` + `FEATURE_STATUS_TRANSLATION_KEY` ให้ใช้ key ใหม่. <!-- done: 2026-05-05 -->
- [x] **SFL.2** `src/sections/feature-management/feature-list-view.tsx` + `components/feature-form.tsx` + `components/feature-status-option.tsx` + `utils/feature-form.ts` — refs ทั้งหมดเปลี่ยนเป็น `FeatureStatus.INACTIVE`. Filter dropdown ยังใช้ `t("common.inactive")` (display label = "Inactive"). <!-- done: 2026-05-05 -->
- [x] **SFL.3** `src/sections/addon-management/addon.enum.ts` — `AddonStatus.ARCHIVED` → `AddonStatus.INACTIVE` (wire value คงเดิม). อัพเดต `ADDON_STATUS_COLOR` + `ADDON_STATUS_TRANSLATION_KEY` ให้ใช้ key ใหม่. <!-- done: 2026-05-05 -->
- [x] **SFL.4** `src/sections/addon-management/addon-list-view.tsx` + `components/addon-form.tsx` + `utils/addon-form.ts` — refs ทั้งหมดเปลี่ยนเป็น `AddonStatus.INACTIVE`. Filter dropdown แสดง label จาก `ADDON_STATUS_TRANSLATION_KEY[AddonStatus.INACTIVE]` = `common.inactive` = "Inactive". <!-- done: 2026-05-05 -->
- [x] **SFL.5** `src/sections/package-management/package.enum.ts` — `PackageStatusTypeEnum.ARCHIVED` → `PackageStatusTypeEnum.INACTIVE` (wire value คงเดิม). `PACKAGE_STATUS_OPTIONS` labelKey เปลี่ยนเป็น `packageManagement.status.inactive`. <!-- done: 2026-05-05 -->
- [x] **SFL.6** `src/sections/package-management/package-detail-view.tsx` + `components/package-table.tsx` — i18n key `packageManagement.status.archived` → `packageManagement.status.inactive` ใน status chip labels (display-only). <!-- done: 2026-05-05 -->
- [x] **SFL.7** `src/locales/langs/en.json` — `packageManagement.status.archived` ("Archived") → `packageManagement.status.inactive` ("Inactive"). <!-- done: 2026-05-05 -->
- [x] **SFL.8** `src/locales/langs/th.json` — `packageManagement.status.archived` ("เก็บถาวร") → `packageManagement.status.inactive` ("ไม่ใช้งาน"). <!-- done: 2026-05-05 -->
- [x] **SFL.9** Verify: grep `FeatureStatus.ARCHIVED|PackageStatusTypeEnum.ARCHIVED|AddonStatus.ARCHIVED|packageManagement.status.archived` returns zero hits. `npx tsc --noEmit` exits 0. Prettier ran on 14 modified files (all conformant). i18n parity confirmed for 3 management namespaces. **Wire value verified unchanged** — TS enum string literal value still `"archived"` ในทุก 3 enums (Feature/Package/Addon), payload ที่ส่ง backend ยังคง `statusType=archived`. <!-- done: 2026-05-05 -->

### Wire Value Migration — `archived` → `inactive` (2026-05-05)

> **Why:** Phase 2 next step — backend Zod schema migration ขนานกัน (DB + Zod ใช้ wire value `'inactive'` แทน `'archived'`). Frontend ต้องเปลี่ยน wire literal ของ TS enums + Redux types ให้ตรงกัน. Display label "Inactive" / "ไม่ใช้งาน" คงเดิม (i18n keys ไม่ต้องแก้ — wave SFL ก่อนหน้าใช้ `common.inactive` / `packageManagement.status.inactive` อยู่แล้ว).
>
> **Approach:** pure value change — flip 3 enum literals + 3 Redux type aliases + 3 JSDoc comments. ไม่มี logic change. **Out of scope:** legacy slice `sale-dashboard-package-config-feature` (Q7 rename), subscription / payment URLs, lead-source `status_archived`.

- [x] **WVM.1** `src/sections/feature-management/feature.enum.ts` — `FeatureStatus.INACTIVE = "archived"` → `"inactive"`. Comment updated to reflect new wire value. <!-- done: 2026-05-05 -->
- [x] **WVM.2** `src/sections/package-management/package.enum.ts` — `PackageStatusTypeEnum.INACTIVE = "archived"` → `"inactive"`. Comment updated. <!-- done: 2026-05-05 -->
- [x] **WVM.3** `src/sections/addon-management/addon.enum.ts` — `AddonStatus.INACTIVE = "archived"` → `"inactive"`. Comment updated. <!-- done: 2026-05-05 -->
- [x] **WVM.4** `src/types/store/sale-dashboard-feature-management.ts` — `FeatureStatusType = "active" | "archived"` → `"active" | "inactive"`. <!-- done: 2026-05-05 -->
- [x] **WVM.5** `src/types/store/sale-dashboard-package-management.ts` — `PackageStatusType = "active" | "archived"` → `"active" | "inactive"`. <!-- done: 2026-05-05 -->
- [x] **WVM.6** `src/types/store/sale-dashboard-addon-management.ts` — `AddonStatusType = "active" | "archived"` → `"active" | "inactive"`. <!-- done: 2026-05-05 -->
- [x] **WVM.7** `src/store/services/{feature,package,addon}-management-request.ts` — JSDoc `Soft-delete (status_type = 'archived')` → `'inactive'` ใน 3 ไฟล์ (documentation sync). <!-- done: 2026-05-05 -->
- [x] **WVM.8** Verify: grep `'archived'|"archived"` ใน Phase 2 scope (sections + components + store/{actions,reducers,sagas,services,types}) returns zero hits. `npx tsc --noEmit` exits 0. Prettier ran on 9 modified files (all conformant). i18n parity confirmed (no orphan `archived` keys; no new keys needed). Dev server (port 4001) HTTP 200 — full Network tab smoke (`statusType=inactive` ใน list query string) deferred to user, ขึ้นกับ backend Zod migration parallel completion. <!-- done: 2026-05-05 -->

### Frontend — Package UUID Update (Added 2026-05-05 — Stripe sync, Wave B4 counterpart)

> **Why:** ฝั่ง Backend Wave B4 เพิ่ม `PATCH /api/v2/admin/packages/:packageUuid/identifier` — Frontend ต้องเตรียม redux slice + reusable dialog component ที่ F5 (Page B) จะนำไป mount ใน package detail/edit view. ทำตอนนี้เพราะ scope ชัดและ component ใช้ซ้ำได้

- [x] **F2.36** ขยาย type ใน `src/types/store/sale-dashboard-package-management.ts` — เพิ่ม state slot `updateUuidLoading`, `updateUuidError` + action types `sdUpdatePackageUuidRequest/Success/Failure` <!-- done: 2026-05-05 -->
- [x] **F2.37** เพิ่ม service method ใน `src/store/services/package-management-request.ts` — `updatePackageUuidRequest(packageUuid, newUuid)` → call `PATCH /api/v2/admin/packages/:packageUuid/identifier` body `{newUuid}` <!-- done: 2026-05-05 -->
- [x] **F2.38** เพิ่ม actions/reducer/saga ใน `src/store/{actions,reducers,sagas}/sale-dashboard-package-management.ts` — saga worker เรียก service + dispatch success/failure + ปรับ state.current ให้ uuid ใหม่หลัง success (หรือ refetch package) <!-- done: 2026-05-05 -->
- [x] **F2.39** สร้าง reusable component `src/components/package-management/package-uuid-edit-dialog.tsx` (หรือ path ที่ตรง pattern repo) — Dialog props: `open`, `onClose`, `currentUuid`, `packageUuid`, `onSuccess?`; form input `newUuid` (Yup `string().uuid().required().notOneOf([currentUuid])`); warning message อธิบายว่าใช้สำหรับ Stripe sync เท่านั้น + ผลกระทบ (jump URL ของ detail page); loading + error state <!-- done: 2026-05-05 -->
- [x] **F2.40** i18n keys (en + th) ใน `src/locales/langs/{en,th}.json` — namespace `packageManagement.uuidEdit.*`: title, description, warning_stripe, current_uuid_label, new_uuid_label, validation.\* (uuid_format / same_as_current / required), buttons.confirm / cancel, success / error messages <!-- done: 2026-05-05 -->

> Implementation note (2026-05-05): UUID v4 regex used (Yup `.matches(...)`) instead of generic `Yup.string().uuid()` — generic validator accepts any uuid version, but the backend (Stripe sync context) implies v4. Same logical strictness as spec, with clearer error message. Saga refetches package list (not detail) since reducer merges the updated package object into `state.current` inline — avoids one extra GET round-trip when the dialog is hosted inside a detail view.

### Frontend — Test

- [ ] **F2.35** Manual UAT — golden path Page A/B/C (รวม UUID edit dialog ใน Page B detail view)

### Edit Page Back Navigation (Added 2026-05-05)

> **Why:** UX requirement — เมื่อ user navigate มาหน้า edit จาก list view ควร back กลับ list; ถ้ามาจาก detail view ควร back กลับ detail. ใช้ query param `?from=list` เป็น signal โดยไม่กระทบ detail view เดิมที่ไม่ส่ง param นี้.
>
> **Approach:** List view append `?from=list` ตอน `handleEdit` navigate; Edit view อ่าน `useSearchParams()` → ถ้า `from === 'list'` → navigate ไป root path; ไม่งั้น navigate ไป detail path (default safe fallback).

- [x] **EBN.1** `src/sections/feature-management/feature-list-view.tsx` — `handleEdit` เปลี่ยน `router.push(paths.featureManagement.edit(uuid))` → append `?from=list` <!-- done: 2026-05-05 -->
- [x] **EBN.2** `src/sections/package-management/package-list-view.tsx` — `handleEdit` append `?from=list` <!-- done: 2026-05-05 -->
- [x] **EBN.3** `src/sections/addon-management/addon-list-view.tsx` — `handleEdit` append `?from=list` <!-- done: 2026-05-05 -->
- [x] **EBN.4** `src/sections/feature-management/feature-edit-view.tsx` — `useSearchParams()` อ่าน `from`; `handleCancel`/`handleBack` branch: `from=list` → `router.push(paths.featureManagement.root)`; else → `router.push(paths.featureManagement.detail(uuid))` <!-- done: 2026-05-05 -->
- [x] **EBN.5** `src/sections/package-management/package-edit-view.tsx` — เดียวกับ EBN.4, path ใช้ `paths.packageManagement.*` <!-- done: 2026-05-05 -->
- [x] **EBN.6** `src/sections/addon-management/addon-edit-view.tsx` — เดียวกับ EBN.4, path ใช้ `paths.addonManagement.*` <!-- done: 2026-05-05 -->
- [x] **EBN.7** Detail views ไม่แก้ — `handleEdit` ที่มีอยู่ไม่ส่ง `?from` = default fallback ทำงานถูกต้อง (back → detail) <!-- done: 2026-05-05 -->
- [x] **EBN.8** `npx tsc --noEmit` exits 0. Prettier conformant on all 6 modified files. <!-- done: 2026-05-05 -->

---

## Phase 3: Per-Company Toggle (Backend + Frontend)

### Backend — Company Feature Module

> Path sync (Q9=A, 2026-05-06): ใช้ `src/modules/v2/sale-dashboard/companyFeature/` + `src/api/v2/sale-dashboard/companyFeature/` (ตาม Move-to-SaleDashboard precedent ของ Phase 2). URL = `/api/v2/sale-dashboard/companies/:companyUuid/features[/:featureUuid]`. SSD §1.4 + §4.4 ที่ยังเขียน `/v2/companies/...` = stale ที่ Lead จะ batch update ทีหลัง

- [x] **P3.1** สร้าง `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.interface.ts` — Zod schemas (get/set/remove) + types `CompanyFeatureItem`, `OverrideRowDb`, `OverrideStatusType`, `CompanyFeatureSource` <!-- done: 2026-05-06 -->
- [x] **P3.2** สร้าง `companyFeature.repository.ts` <!-- done: 2026-05-06 -->
  - [x] `getCompanyByUuidRepository(companyUuid)` — lookup id + packageId <!-- done: 2026-05-06 -->
  - [x] `listActiveFeaturesRepository()` — Q4=B base list (status_type='active' only, ordered) <!-- done: 2026-05-06 -->
  - [x] `getFeatureByUuidForOverrideRepository(featureUuid)` — return id + uuid + statusType, service block on inactive <!-- done: 2026-05-06 -->
  - [x] `getOverridesByCompanyRepository(companyId)` — JOIN comp_features filter status_type='active' (Q4=B) <!-- done: 2026-05-06 -->
  - [x] `upsertOverrideRepository(companyId, featureId, status, reason, actorBy)` — PostgreSQL ON CONFLICT (company_id, feature_id) DO UPDATE; status_type→'active' <!-- done: 2026-05-06 -->
  - [x] `removeOverrideRepository(companyId, featureId)` — hard delete + return {removed} <!-- done: 2026-05-06 -->
  - [x] `getCompanyAddonsRepository(companyId)` — **N/A** (reuse `getActiveCompanyAddonIdsRepository` + `getAddonFeatureIdsRepository` จาก permissionResolver — ห้าม duplicate logic) <!-- done: 2026-05-06 -->
- [x] **P3.3** สร้าง `companyFeature.service.ts` <!-- done: 2026-05-06 -->
  - [x] `getCompanyFeaturesService(companyUuid)` — list active features with effective + source; pre-fetch + Promise.all 4 batch queries (resolver repos + listActiveFeatures + getOverrides) <!-- done: 2026-05-06 -->
  - [x] `setCompanyFeatureOverrideService(companyUuid, featureUuid, enabled, reason, actor)` — UPSERT + race handling (PG 23505/40001/40P01 → 409 CONCURRENT_OVERRIDE per SSD §7); blocks Q4=B inactive feature <!-- done: 2026-05-06 -->
  - [x] `removeCompanyFeatureOverrideService(companyUuid, featureUuid, actor)` — hard delete; idempotent (no-op log if not found); blocks Q4=B inactive feature <!-- done: 2026-05-06 -->
  - [x] Reuse permissionResolver: `getCompanyPackageIdRepository`, `getSeedPackageIdRepository`, `getPackageFeatureIdsRepository`, `getActiveCompanyAddonIdsRepository`, `getAddonFeatureIdsRepository` (no duplicate) <!-- done: 2026-05-06 -->
- [x] **P3.4** สร้าง `companyFeature.adapter.ts` — pure `computeCompanyFeatureItem` + `toCompanyFeatureItems` (source priority: override > package > addon > default-disabled per backend.md §2.4) <!-- done: 2026-05-06 -->
- [x] **P3.5** สร้าง `src/api/v2/sale-dashboard/companyFeature/companyFeature.controller.ts` — 3 controllers wrap services; res.success / handleError pattern; resolveActorUuid from req.user <!-- done: 2026-05-06 -->
- [x] **P3.6** สร้าง `companyFeature.routes.ts` + register <!-- done: 2026-05-06 -->
  - [x] basePath `/api/v2/sale-dashboard/companies`; tags `Company Features - V2 Sale Dashboard` <!-- done: 2026-05-06 -->
  - [x] Middleware chain: `validateSaleDashboardAccessToken` → `requireSuperAdmin` → `validateRequest` <!-- done: 2026-05-06 -->
  - [x] Routes: GET `/:companyUuid/features`, PUT/DELETE `/:companyUuid/features/:featureUuid` (specific paths registered first per Express greedy match) <!-- done: 2026-05-06 -->
  - [x] Mounted ใน `sale-dashboard.routes.ts` ที่ `/companies` segment <!-- done: 2026-05-06 -->
  - [x] TS pass + prettier conformant <!-- done: 2026-05-06 -->
  - [x] Frontend equivalent path: `/api/v2/sale-dashboard/companies/:companyUuid/features/...` (Wave 3 จะใช้) <!-- done: 2026-05-06 -->

### Backend — Permission Resolver Module

> Path sync (Q9=A, 2026-05-06): ใช้ `src/modules/v2/sale-dashboard/permissionResolver/` แทน `v2/admin/...` (ตาม Move-to-SaleDashboard precedent ของ Phase 2). SSD §1.4 + §7 ที่ยังเขียน `/v2/admin/...` = stale ที่ Lead จะ batch update ทีหลัง

- [x] **P3.7** สร้าง `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.interface.ts` <!-- done: 2026-05-06 -->
- [x] **P3.8** สร้าง `permissionResolver.repository.ts` <!-- done: 2026-05-06 -->
  - [x] `getPackageFeatureIdsRepository(packageId)` — JOIN comp_package_features ↔ comp_features filter status_type='active' <!-- done: 2026-05-06 -->
  - [x] `getActiveCompanyAddonIdsRepository(companyId)` — Q3=A runtime expiry filter (`expires_at IS NULL OR expires_at > NOW()`) <!-- done: 2026-05-06 -->
  - [x] `getAddonFeatureIdsRepository(addonIds[])` — batch WHERE IN + DISTINCT + filter feature.status_type='active' <!-- done: 2026-05-06 -->
  - [x] `getCompanyOverridesRepository(companyId)` — JOIN comp_company_features ↔ comp_features (Q4=B: filter feature.status_type='active') <!-- done: 2026-05-06 -->
  - [x] `getFeatureMenuKeysRepository(featureIds[])` — batch WHERE IN + LATERAL jsonb_array_elements_text (DB-level unnest) <!-- done: 2026-05-06 -->
  - [x] `getCompanyPackageIdRepository(companyId)` + `getSeedPackageIdRepository()` — Seed fallback support <!-- done: 2026-05-06 -->
  - [x] `getUserPermissionJsonbRepository(userUuid)` — employee → comp_permission_mapping → comp_permission (Phase 4 prep) <!-- done: 2026-05-06 -->
- [x] **P3.9** สร้าง `permissionResolver.service.ts` <!-- done: 2026-05-06 -->
  - [x] `resolveCompanyEffectiveFeatureIdsService(companyId, cache?)` — Seed fallback + log "warn" (via `logger.error` + `level:'warn'` since `logger.warn` not exposed) <!-- done: 2026-05-06 -->
  - [x] `resolveCompanyEffectiveMenuKeysService(companyId, cache?)` — reuse feature ids resolver via cache <!-- done: 2026-05-06 -->
  - [x] `filterPermissionByMenuKeysService(jsonb, allowedKeys)` — pure recursive walker, leaf detection via `hasActionKey` ตาม PermissionDefault shape <!-- done: 2026-05-06 -->
  - [x] `resolveUserEffectivePermissionService(userUuid, cache?)` — throws `PERMISSION_DATA_NOT_FOUND` (404) / `PERMISSION_DATA_CORRUPTED` (500) ตาม SSD §7 <!-- done: 2026-05-06 -->
  - [x] In-request memoization — Option C: explicit `cache?: Map<string, unknown>` param (no Express coupling, testable) <!-- done: 2026-05-06 -->

### Backend — Tests

- [x] **P3.10** Unit test: companyFeature.service.test.ts <!-- done: 2026-05-06 -->
  - File: `happywork-backend/tests/unit/modules/v2/sale-dashboard/companyFeature/companyFeature.service.test.ts`
  - 26 cases: 11 (getCompanyFeaturesService TC-1..TC-11) + 11 (setCompanyFeatureOverrideService TC-12..TC-19b) + 4 (removeCompanyFeatureOverrideService TC-20..TC-23). PASS 26/26.
- [x] **P3.11** Unit test: permissionResolver.service.test.ts (delta logic, addon union, override) <!-- done: 2026-05-06 -->
  - File: `happywork-backend/tests/unit/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.test.ts`
  - 32 cases: resolveEffectivePackageId (5) + resolveCompanyEffectiveFeatureIds TC-1..TC-9+1 (10) + resolveCompanyEffectiveMenuKeys TC-10..TC-13+1 (5) + filterPermissionByMenuKeys TC-14..TC-20+1 (8) + resolveUserEffectivePermission TC-21..TC-23+1 (4). PASS 32/32.
- [x] **P3.12** Integration test: end-to-end resolver flow <!-- done: 2026-05-06 -->
  - File: `happywork-backend/tests/integration/sale-dashboard/companyFeature.integration.spec.ts`
  - 17 cases: GET (4) + PUT (8 — IT-2/5/6/7/8 + propagation) + DELETE (3) + IT-4 round-trip + sanity (1). PASS 17/17 บน local (Mac) ด้วย `NODE_OPTIONS="--max-old-space-size=8192"`.
  - Total Phase 3 unit + integration = **75/75 PASS** (Time 13.4s).
  - Note: errorHandler limitation surfaced — `errorMethods.errorXxx()` (CustomError with `.status` field) ไม่ตรง `isHttpException` (`.status` check vs AppError `.statusCode`). ใน prod path, errors จาก service จะ fallback เป็น HTTP 500 ทุกตัว (response.code ยัง surface ผ่าน `error.code` แต่ HTTP status 500). Integration test จึง throw `AppError` จาก mocked services เพื่อสะท้อนสิ่งที่ middleware รองรับ. **Recommendation**: SA/Lead ตัดสินใจ — ถ้าต้องการ HTTP status ตรงกับ errorMethods status ต้องแก้ `errorHandlers.middleware.ts` `isHttpException` ให้ check `'status' in err` หรือ migrate ทุก service throw → AppError. (ตอนนี้ระบบ controller test = pass แต่ user-facing error UX ยังเป็น 500 ทุก domain error → ต้องตามต่อใน follow-up CR)

### Frontend — Redux Slice (Company Features)

> Slice naming Q11=A: `sale-dashboard-company-features` (kebab plural, not `-management` suffix). Reasons: (1) frontend.md §3 lines 112+125 already spec this name; (2) Phase 2 used `-management` because those slices back master-data CRUD pages, but Wave 3 backs a tab inside an existing client detail page — not a management page; (3) URL itself is plural (`/companies/:uuid/features`).

- [x] **F3.1** สร้าง `src/types/store/sale-dashboard-company-features.ts` <!-- done: 2026-05-06 -->
  - [x] `CompanyFeatureSource` union ตรง SSD §3.3 (5 variants); `CompanyFeatureItem` (featureUuid, featureCode, name multilingual, enabled, source, overrideReason?) <!-- done: 2026-05-06 -->
  - [x] Request payloads: `GetCompanyFeaturesParams`, `SetCompanyFeatureOverridePayload`, `RemoveCompanyFeatureOverridePayload` <!-- done: 2026-05-06 -->
  - [x] Response payloads: `GetCompanyFeaturesResponse = {list}`, set/remove return single `CompanyFeatureItem` <!-- done: 2026-05-06 -->
  - [x] `Root` state: list + isListLoading + listError + togglingFeatureUuid + toggleError (per-row toggling lock) <!-- done: 2026-05-06 -->
  - [x] Registered ใน `src/types/store/index.ts` (slot key `saleDashboardCompanyFeatures` — verified no collision via grep) <!-- done: 2026-05-06 -->
- [x] **F3.2** สร้าง `src/store/services/company-features-request.ts` <!-- done: 2026-05-06 -->
  - [x] `sdGetCompanyFeaturesRequest(companyUuid)` → GET `/api/v2/sale-dashboard/companies/:uuid/features` <!-- done: 2026-05-06 -->
  - [x] `sdSetCompanyFeatureOverrideRequest(companyUuid, featureUuid, {enabled, reason?})` → PUT <!-- done: 2026-05-06 -->
  - [x] `sdRemoveCompanyFeatureOverrideRequest(companyUuid, featureUuid)` → DELETE <!-- done: 2026-05-06 -->
  - [x] เพิ่ม URL builders ใน `sale-dashboard.ts`: `saleDashboardCompanyFeatures(uuid)` + `saleDashboardCompanyFeatureDetail(uuid, featureUuid)` (prefix `saleDashboard*` เพื่อกัน collision กับ legacy `companyFeatures(id)` ที่ชี้ไป `/company-settings/...`) <!-- done: 2026-05-06 -->
- [x] **F3.3** สร้าง action/reducer/saga + register <!-- done: 2026-05-06 -->
  - [x] Actions (`sale-dashboard-company-features.ts`): 3 RSF triplets + 2 cleanup helpers (clearError, clearList) <!-- done: 2026-05-06 -->
  - [x] Reducer with `replaceRowByFeatureUuid` immutable map (single-row update, ไม่ refetch list) <!-- done: 2026-05-06 -->
  - [x] Saga: `takeLatest` สำหรับ GET, `takeEvery` สำหรับ set/remove (ป้องกัน rapid toggle หลายแถวถูก cancel) <!-- done: 2026-05-06 -->
  - [x] Register: actions/index.ts + reducers/index.ts (slot `saleDashboardCompanyFeatures`) + sagas/index.ts (`watchSaleDashboardCompanyFeatures`) <!-- done: 2026-05-06 -->
- [x] **F3.4** สร้าง hook `useSaleDashboardCompanyFeatures` (อยู่ในไฟล์ reducer ตาม pattern Phase 2) <!-- done: 2026-05-06 -->
  - [x] Returns: `{...state, getCompanyFeatures, setCompanyFeatureOverride, removeCompanyFeatureOverride, clearError, clearList}` <!-- done: 2026-05-06 -->
  - [x] ทุก dispatcher wrap `useCallback([dispatch])` (Phase 2 max-depth bug fix pattern) <!-- done: 2026-05-06 -->
  - [x] TS pass + prettier conformant <!-- done: 2026-05-06 -->

### Frontend — Page D: Features Tab

- [x] **F3.5** สร้าง `happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx` <!-- done: 2026-05-06 -->
  - [x] Orchestrator: load list on mount via `getCompanyFeatures(companyUuid)` + cleanup `clearList()` on unmount <!-- done: 2026-05-06 -->
  - [x] Local state: `sourceFilter` (default `'all'`) + dialog open + `targetItem` <!-- done: 2026-05-06 -->
  - [x] Derived via `useMemo`: `counts` (6 buckets — all + 5 source variants) + `filteredList` (no-op for 'all', .filter otherwise) <!-- done: 2026-05-06 -->
  - [x] Toast for `listError` + `toggleError` (auto-clear via `clearError()` after emission) <!-- done: 2026-05-06 -->
  - [x] All handlers wrapped in `useCallback` for stable child memoization <!-- done: 2026-05-06 -->
- [x] **F3.6** สร้าง `feature-tab/components/company-feature-list.tsx` <!-- done: 2026-05-06 -->
  - [x] Pure presentational; renders `<Skeleton/>` × 4 while loading; "no_features" empty state <!-- done: 2026-05-06 -->
- [x] **F3.7** สร้าง `feature-tab/components/company-feature-row.tsx` <!-- done: 2026-05-06 -->
  - [x] Reuses Phase 2 `<FeatureSourceBadge source={...}/>` + bilingual name via `getBilingualText` + currentLang <!-- done: 2026-05-06 -->
  - [x] Per-row `disabled = togglingFeatureUuid === item.featureUuid` (lock only the flipping row) <!-- done: 2026-05-06 -->
  - [x] `Switch onClick` (not onChange) intercepts user gesture → calls `onToggle(item)` to open dialog (no optimistic flip) <!-- done: 2026-05-06 -->
  - [x] Override reason rendered under name when present <!-- done: 2026-05-06 -->
- [x] **F3.8** สร้าง `feature-tab/components/company-feature-toggle-confirm-dialog.tsx` <!-- done: 2026-05-06 -->
  - [x] Dialog mode logic: `hasOverride` (source startsWith `override-`) → 3 buttons (Cancel + Reset to Default + Set); else 2 buttons (Cancel + Set) <!-- done: 2026-05-06 -->
  - [x] Reason TextField (max 500 chars + char counter) with reset on open <!-- done: 2026-05-06 -->
  - [x] Confirm Set button color reflects target state (primary for enable, error for disable) <!-- done: 2026-05-06 -->
- [x] **F3.9** สร้าง `feature-tab/components/company-feature-source-filter.tsx` <!-- done: 2026-05-06 -->
  - [x] MUI `<Select>` with 6 options (`all` + 5 sources) + count suffix per option <!-- done: 2026-05-06 -->
  - [x] Type-safe `CompanyFeatureSourceFilterValue` + `CompanyFeatureSourceCounts` exported for parent use <!-- done: 2026-05-06 -->
- [x] **F3.10** เพิ่ม tab "Features" ใน existing client detail view <!-- done: 2026-05-06 -->
  - [x] File: `src/sections/client-management/company/list-view.tsx` (existing tabs render at line ~350) <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `<Tab value="features"...>` + conditional render `{currentTab === "features" && param?.uuid && <CompanyFeaturesTabView companyUuid={String(param.uuid)} />}` <!-- done: 2026-05-06 -->
  - [x] No `<Can>` wrapper (backend `requireSuperAdmin` enforces) — left as-is per spec <!-- done: 2026-05-06 -->
- [ ] **F3.11** Manual UAT — toggle/reset/source badge ถูกต้อง (deferred to QA wave)

---

## Phase 4: Resolver Integration (Backend + Regression Test)

### Backend — Update Existing Endpoints

- [x] **P4.1** อัพเดต `happywork-backend/src/api/external/auth/permissions.controller.ts` <!-- done: 2026-05-06 -->
  - [x] `getPermissionsController` — **left untouched** (Phase 2 layering: resolver call lives in service per `Service orchestrate repository` rule; controller stays thin with try/catch + `handleError` + `res.success`). Response shape extends automatically via `PermissionsResponse`. <!-- done: 2026-05-06 -->
- [x] **P4.2** อัพเดต `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts` <!-- done: 2026-05-06 -->
  - [x] `getPermissionsService` — added `resolveUserEffectivePermissionService` call with per-request `ResolverCache` (`new Map()`); `PermissionsResponse` interface extended with `permission: Record<string, unknown>` field <!-- done: 2026-05-06 -->
  - [x] Edge case `PERMISSION_DATA_NOT_FOUND` → caught + return `permission: {}` (graceful fallback, preserves prior un-assigned-user behavior); other resolver errors (`PERMISSION_DATA_CORRUPTED`, `COMPANY_NOT_FOUND`, `SEED_PACKAGE_NOT_FOUND`) propagate <!-- done: 2026-05-06 -->
- [x] **P4.3** อัพเดต `happywork-backend/src/modules/v1/admin/compPermission/*` <!-- done: 2026-05-06 -->
  - [x] `getCompPermissionByUuidService` — filter `permission` + `permissionMobile` JSONB ผ่าน `filterPermissionByMenuKeysService` ตาม company effective menus (resolve menu keys ผ่าน `resolveCompanyEffectiveMenuKeysService` พร้อม per-call `ResolverCache`) <!-- done: 2026-05-06 -->
  - [x] `getCompPermissionListService` + `getCompPermissionByIdService` — **ไม่ต้องแก้** (existing repo `select(...)` ไม่ return `permission`/`permissionMobile` JSONB; list response shape ใช้แค่ `id, uuid, createdAt, updatedAt, statusType, name, description`) <!-- done: 2026-05-06 -->
  - [x] Single-company assumption confirmed via `GetCompPermissionListInterface.filter.companyId` (controller derives จาก `req.user.companyId` หรือ `companyUuid` query) — ไม่ต้อง group-by <!-- done: 2026-05-06 -->
- [x] **P4.4** อัพเดต `happywork-backend/src/modules/v1/admin/compPermission/*` (path correction — Wave 2 spec) <!-- done: 2026-05-06 -->
  - [x] **Path correction:** P4.4 ทำที่ `compPermission` create/update services (ที่ save JSONB จริง) ไม่ใช่ `compPermissionMapping` (ซึ่งแค่ map employee→permissionId ไม่มี JSONB) <!-- done: 2026-05-06 -->
  - [x] `createCompPermissionService` — pre-validate JSONB ผ่าน `assertPermissionKeysWithinCompanyMenusService` (collect leaf paths + intersect กับ company effective menus); reject 400 `INVALID_MENU_KEY_FOR_COMPANY` พร้อม `data.invalidKeys` list <!-- done: 2026-05-06 -->
  - [x] `updateCompPermissionByUuidService` — same validation; companyId lookup via lightweight `getCompPermissionByUuid(uuid, ['id', 'companyId'])` (avoid recursive self-call to filter-applied `getCompPermissionByUuidService`); skip validation เมื่อ payload ไม่มี `permission` หรือ `permissionMobile` <!-- done: 2026-05-06 -->
  - [x] Helper `extractMenuKeysFromPermissionJsonbService` ใหม่ใน `permissionResolver.service.ts` (co-located กับ `filterPermissionByMenuKeysService` — ใช้ leaf-detection logic ตัวเดียวกัน, ACTION_KEYS / NON_CHILD_KEYS / `hasActionKey`) <!-- done: 2026-05-06 -->
  - [x] Empty `{}` JSONB → no-op (skip resolver call); both web + mobile JSONB validated; invalid keys deduped (Set) before throwing <!-- done: 2026-05-06 -->
  - [x] Error preservation: AppError + CustomError (`INVALID_MENU_KEY_FOR_COMPANY` code) propagate ผ่าน try/catch โดยไม่ถูก wrap เป็น 500 <!-- done: 2026-05-06 -->

## Phase 4 W3.5 — Audit Fix Wave (2026-05-06)

> F4.1 audit (`document/requirement/feature-management/f4-1-frontend-audit.md`) เจอช่องว่างที่ Phase 4 W1+W2 ยังไม่ครอบคลุม:
> Lead-confirmed decisions: **Q-A=STRICT** (owner ของ company ที่ feature ปิด → ไม่เห็น menu), **Q-B** (`/comp-permission/default` รับ `companyUuid` query param), **Q-C=No data migration** (per-read filter เท่านั้น).
> Frontend ไม่ต้องแก้ (UI render จาก JSONB ตรงๆ — เมื่อ backend filter pre-merge แล้ว menu ปิดหายไปทันที).

- [x] **W3.5.1 — Fix 1: `/auth/user-info` hot path** (`compPermissionMapping.service.ts:106` `getEmployeePermissionService`) <!-- done: 2026-05-06 -->
  - [x] Resolve `companyId` จาก permission row (JOIN ของ `getCompPermissionMappingByEmployeeId` select `compPermission.*` ซึ่งมี `companyId`); pick first non-null <!-- done: 2026-05-06 -->
  - [x] เรียก `resolveCompanyEffectiveMenuKeysService(companyId, cache)` (1 cache ต่อ call) → filter `PermissionDefault` + `PermissionMobileDefault` baseline ก่อน deepMerge <!-- done: 2026-05-06 -->
  - [x] Post-deepMerge re-filter: `deepMerge` อาจ inject keys นอก allowed set กลับเข้ามาจาก role JSONB ที่ Q-C=no migration → second filter pass ใช้ `allowedKeys` ที่ resolved แล้ว (ไม่เรียก resolver ซ้ำ) <!-- done: 2026-05-06 -->
  - [x] Q-A=STRICT enforced: `isOwner=true` ก็ใช้ baseline ที่ filter แล้ว (ไม่ bypass) — owner ไม่เห็น menu ของ feature ปิด <!-- done: 2026-05-06 -->
  - [x] Edge case: `companyId` หาไม่เจอ (no permission row) → fallback ใช้ baseline เต็ม (ไม่ throw); resolver error (COMPANY_NOT_FOUND / SEED_PACKAGE_NOT_FOUND) → propagate per SSD §7 <!-- done: 2026-05-06 -->
- [x] **W3.5.2 — Fix 2: `getCompPermissionByUuidController` deepMerge baseline filter** (`compPermission.controller.ts:147-148`) <!-- done: 2026-05-06 -->
  - [x] Approach: filter `PermissionDefault` baseline ก่อน deepMerge ใน controller (touch ≤ ย้าย deepMerge เข้า service); cache สำหรับ web + mobile baseline ใช้ resolver call เดียว <!-- done: 2026-05-06 -->
  - [x] Wave 2 service-level filter ของ saved JSONB ยังคงทำงาน — ผลลัพธ์รวมเป็น `deepMerge(filteredDefault, filteredPermission, isOwner)` ไม่มี leftover keys <!-- done: 2026-05-06 -->
  - [x] Owner case: `deepMerge isOwner=true` bypass override per-leaf แต่ baseline filter แล้ว → owner = subset ของ company-enabled menus (parity กับ Fix 1) <!-- done: 2026-05-06 -->
- [x] **W3.5.3 — Fix 3: `/comp-permission/default` filter by company** (`compPermission.controller.ts:111-127` `getCompPermissionDefaultListController`) <!-- done: 2026-05-06 -->
  - [x] Zod schema update: `getCompPermissionDefaultListSchema.query` รับ `companyUuid` optional UUID (`schemas/compPermission.ts:26`) <!-- done: 2026-05-06 -->
  - [x] Controller: ถ้ามี `companyUuid` → `getCompanyByUuidService(companyUuid)` (existing v1 service ที่ throw 400 ถ้าไม่พบ) → `resolveCompanyEffectiveMenuKeysService(company.id, cache)` → filter PermissionDefault + PermissionMobileDefault <!-- done: 2026-05-06 -->
  - [x] Backward compat: ไม่มี `companyUuid` → คืน full baseline เหมือนเดิม (super-admin role-create UX) <!-- done: 2026-05-06 -->
- [x] **W3.5.4 — Fix 4: Owner case STRICT enforcement** <!-- done: 2026-05-06 -->
  - [x] No extra code — Fix 1 + Fix 2 baseline filter already enforces Q-A=STRICT (owner deepMerge bypass override per-leaf, but baseline เป็น filtered → owner = company-enabled subset) <!-- done: 2026-05-06 -->
  - [x] Verify scenario "owner of company that disabled feature X" → menu X หายจาก runtime nav + role-edit table — รอ QA Wave 4 (P4.5+P4.6) <!-- done: 2026-05-06 -->
- [x] **W3.5.5 — Verification** <!-- done: 2026-05-06 -->
  - [x] TS compile clean: `NODE_OPTIONS="--max-old-space-size=8192" npx tsc --noEmit` exit 0 <!-- done: 2026-05-06 -->
  - [x] Prettier conformant: 3 ไฟล์ unchanged after `prettier --write` (compPermissionMapping.service.ts, compPermission.controller.ts, schemas/compPermission.ts) <!-- done: 2026-05-06 -->
  - [x] No `any` type introduced; explicit types via `PermissionJsonb` + `ResolverCache` <!-- done: 2026-05-06 -->

### Frontend — Permission Management UI

- [x] **F4.1** ตรวจสอบหน้า permission management เดิม (CMS) <!-- done: 2026-05-06 -->
  - [x] Audit complete: `document/requirement/feature-management/f4-1-frontend-audit.md` <!-- done: 2026-05-06 -->
  - [x] Verdict: Frontend ไม่ต้องแก้ — UI render จาก JSONB ตรงๆ; gap อยู่ที่ backend (4 endpoints ไม่ pre-filter ก่อน deepMerge) → fix ใน W3.5 below <!-- done: 2026-05-06 -->
  - [x] `MENU_REGISTRY` (frontend) เป็น pure metadata (icon + i18n + route); iterate ตาม `userInfo.permission` keys ที่ backend ส่งมา → safe ทันทีเมื่อ backend filter pre-merge แล้ว <!-- done: 2026-05-06 -->
  - [x] `mergePermissions(default, specific)` ใน `step-permission.tsx` safe เมื่อ `/comp-permission/default` filter ตาม company (W3.5 Fix 3) — ไม่ต้องแก้ frontend <!-- done: 2026-05-06 -->

### Tests & Regression

- [x] **P4.5** Integration test: end-to-end <!-- done: 2026-05-06 -->
  - [x] Toggle feature → re-fetch permission API → ผล filter ตรง (IT-A — `permission-resolver.integration.spec.ts` `getEmployeePermissionService` group, asserts `payroll.runPayroll` filtered out when override disables it) <!-- done: 2026-05-06 -->
  - [x] Add addon → addon features ปรากฏใน effective (IT-B — resolver returns expanded set incl. addon menu; assertion via `collectLeavesFromFiltered`) <!-- done: 2026-05-06 -->
  - [x] Override disabled → feature ที่ package เปิด → ไม่ปรากฏ (IT-C — explicit override-disabled case) <!-- done: 2026-05-06 -->
  - [x] Remove override → กลับไปใช้ default จาก package (IT-D — resolver returns full package menus → menu visible again) <!-- done: 2026-05-06 -->
- [x] **P4.6** Regression test: existing permission users <!-- done: 2026-05-06 -->
  - [x] User ของ company ที่ไม่มี override → permission เหมือนเดิม (IT-E — baseline allowed-keys path, `expect.arrayContaining(['dashboard','myHappywork.attendance','report.leaveReport'])`) <!-- done: 2026-05-06 -->
  - [x] รัน permission API ของ user ทั้งหมดในระบบ test → ไม่มี error (IT-F shape sanity + W1 error propagation; PERMISSION_DATA_NOT_FOUND graceful, COMPANY_NOT_FOUND propagates) <!-- done: 2026-05-06 -->
- [x] **F4.2** UAT: เปิด HappyWork app เป็น user ของ company ที่ feature ถูกปิด → เมนูที่เกี่ยวข้องไม่แสดง <!-- done: 2026-05-06 -->
  - [x] **API-level UAT smoke** via integration test แทน UI (UI test รอ Playwright wave): IT-G owner Q-A=STRICT + IT-H non-owner parity ใน `getEmployeePermissionService`, IT-K admin role-edit (owner + non-owner parity) ใน `getCompPermissionByUuidService`, IT-I/J `/comp-permission/default` companyUuid filter <!-- done: 2026-05-06 -->
  - [x] Additional W3.5 cases: IT-L P4.4 save-validation INVALID_MENU_KEY_FOR_COMPANY 400 reject + 4 supplementary cases (accept-when-valid, empty `{}` no-op, update reject, update without payload skip) <!-- done: 2026-05-06 -->

### Wave 4 Results (2026-05-06 — QA)

- **File:** `happywork-backend/tests/integration/sale-dashboard/permission-resolver.integration.spec.ts` (25 TCs)
- **Run:** `NODE_OPTIONS="--max-old-space-size=8192" npx jest --testPathPattern="permission-resolver" --no-coverage` → 25 / 25 PASS in 9.58 s
- **TS check:** `tsc --noEmit` exit 0 (full project)
- **Prettier:** spec file conformant after `prettier --write`
- **Sibling regression:** `companyFeature.integration.spec.ts` re-run 17 / 17 PASS (no cross-spec mock leakage)
- **Defects found:** 1 minor (DEF-001 — pre-existing stale unit `tests/unit/externalAuth/permissions.service.test.ts` doesn't include `permission` field added in P4.2; not a Wave 4 regression, see `testcase.md` §7)

---

## Phase 4 CR Fix Wave (2026-05-06)

> Source: `document/requirement/feature-management/cr-phase4.md` — verdict PASS-with-fixes (0 critical / 2 high / 2 medium / 3 low). BLOCKED on H1+H2.
> Approach: ทำ 6 จาก 7 findings; skip L3 (test code style — 25/25 ยัง pass). หลัง fix ครบ → ready for BA.

### CR Fixes (6 ticked + 1 deferred)

- [x] **CR-H1** Double resolver call ใน `getCompPermissionByUuidController` <!-- done: 2026-05-06 -->
  - [x] เพิ่ม optional `cache?: ResolverCache` parameter ที่ `getCompPermissionByUuidService` (`happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts`) — ถ้า caller ไม่ส่ง → service สร้างใหม่ (backward compat ของ caller อื่น)
  - [x] Controller (`happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts:getCompPermissionByUuidController`) สร้าง `cache` ตัวเดียว ส่งให้ service + reuse กับ `resolveCompanyEffectiveMenuKeysService` ที่ baseline filter ก่อน deepMerge
  - [x] ผลลัพธ์: resolver call เหลือ 1 ครั้งต่อ request (cache hit ครั้งที่ 2 ผ่าน `menuKeys:{companyId}` key)
  - [x] Verify other callers (`compPermissionMapping.controller.ts:64,178`) ไม่กระทบ — เรียก `getCompPermissionByUuidService(uuid)` ไม่ส่ง cache → fallback path ทำงานเหมือนเดิม
- [x] **CR-H2** DEF-001 — stale unit test `tests/unit/externalAuth/permissions.service.test.ts` <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `jest.mock('@/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service')` + mock `resolveUserEffectivePermissionService`
  - [x] Update happy-path tests assert `{ userUuid, isAdmin, permission }` (3 fields) + `mockedResolvePermission` called with `(userUuid, expect.any(Map))`
  - [x] เพิ่ม test case ใหม่: `PERMISSION_DATA_NOT_FOUND` → returns empty `{}` graceful fallback
  - [x] เพิ่ม test case ใหม่: `PERMISSION_DATA_CORRUPTED` → propagates (assert via `code: 'PERMISSION_DATA_CORRUPTED'`)
  - [x] 404 case ยัง pass + assert resolver `not.toHaveBeenCalled()`
  - [x] 5/5 tests pass (เดิม 3 → 5) — `permissions.service.ts` coverage = 100/100/100/100
- [x] **CR-M1** Unnecessary resolver call for `{}` JSONB <!-- done: 2026-05-06 -->
  - [x] ย้าย `desiredKeys.length === 0 → return` ขึ้นมา BEFORE `resolveCompanyEffectiveMenuKeysService` ใน `assertPermissionKeysWithinCompanyMenusService` (`happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts`)
  - [x] ผลลัพธ์: save permission ที่ payload เป็น `{}` ทั้ง web+mobile → 0 DB queries (เดิม 4-5 queries แล้วทิ้ง)
- [x] **CR-M2** Double DB read ใน `updateCompPermissionByUuidController` <!-- done: 2026-05-06 -->
  - [x] เพิ่ม optional `existingCompanyId?: string` param ที่ `updateCompPermissionByUuidService` — ถ้ามี → skip internal `getCompPermissionByUuid` lookup ใน P4.4 validation
  - [x] Controller ส่ง `compPermission.companyId` (ที่ได้จาก `getCompPermissionByUuidService` ตอนต้น flow) เข้า service → 1 lookup ต่อ request (เดิม 2)
  - [x] Backward compat: caller อื่นไม่กระทบ (param เป็น optional, default fallback ทำงานเดิม)
- [x] **CR-L1** `logger.error` with `level: 'info'` workaround <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `// TODO: switch to logger.info when the diagnostic logger gains an info level (Phase 5 prep)` comment ที่ `permissions.service.ts:46` (option a — minimal change ตามที่ task spec ระบุ)
- [x] **CR-L2** `z.any()` violates no-`any` rule <!-- done: 2026-05-06 -->
  - [x] Replace `z.record(z.string(), z.any())` → `z.record(z.string(), z.unknown())` ที่ `happywork-backend/src/schemas/compPermission.ts` (6 occurrences: lines 14, 15, 37, 38, 137, 138)
  - [x] Downstream interface (`CreateCompPermissionInterface.permission: object` / `UpdateCompPermissionInterface.permission: object`) compatible — `Record<string, unknown>` ⊂ `object`
- [ ] **CR-L3** _DEFERRED_ — Test `IT-J` inline logic refactor (cosmetic, all 25 tests still pass; tracked for Phase 5 prep)

### Verification

- **Grep verification (post-fix):**
  - H1: `getCompPermissionByUuidService = async (uuid: string, cache?: ResolverCache)` confirmed; controller pass `cache` to single service call
  - H2: test file imports `resolveUserEffectivePermissionService` + `jest.mock(...)` registered; 5 tests pass
  - M1: `desiredKeys.length === 0 → return` ขึ้นก่อน `resolveCompanyEffectiveMenuKeysService` (line 59 vs 61 in service)
  - M2: `updateCompPermissionByUuidService(uuid, data, existingCompanyId?)` signature confirmed; controller passes `compPermission.companyId`
- **TS compile:** `NODE_OPTIONS="--max-old-space-size=8192" npx tsc --noEmit` exit 0 (full project)
- **Prettier:** 5 modified files conformant (`prettier --write` reports unchanged)
- **Tests after fix:**
  - `tests/unit/externalAuth/permissions.service.test.ts` — 5/5 PASS (เพิ่ม 2 test cases)
  - `tests/integration/sale-dashboard/permission-resolver.integration.spec.ts` — 25/25 PASS (no regression)
  - `tests/integration/sale-dashboard/companyFeature.integration.spec.ts` — 17/17 PASS (no regression)

### Files modified (5)

- `happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts` — H1 cache share + M2 pass companyId
- `happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts` — H1 optional cache param + M1 early-return + M2 optional existingCompanyId param
- `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts` — L1 TODO comment
- `happywork-backend/src/schemas/compPermission.ts` — L2 z.any → z.unknown (6 spots)
- `happywork-backend/tests/unit/externalAuth/permissions.service.test.ts` — H2 mock resolver + 2 new test cases

---

## Phase 5: Cleanup (LATER — แยก scope)

> ทำหลังจาก Phase 1-4 deploy stable แล้ว และ confirm ว่า v1 caller ไม่ active

- [ ] **P5.1** Migrate caller ของ v1 `featureManagement` module ไปใช้ resolver/companyFeature ใหม่
  - [ ] grep `getAllFeaturesService`, `toggleFeatureService` → migrate
- [ ] **P5.2** Drop migration: `*-drop-sale_dashboard_disabled_features.ts`
- [ ] **P5.3** Drop module: `happywork-backend/src/modules/v1/featureManagement/`
- [ ] **P5.4** Drop constant: `happywork-backend/src/constant/featureDefinitions.ts`
- [ ] **P5.5** Eventually drop `const_packages` (หลัง v1 caller ที่อ้างอิง const_packages migrate ครบ)
- [ ] **P5.6** Drop column `comp_companies.add_ons` (jsonb) ที่ migrate ไป comp_company_addons แล้ว

---

## URL Fix — OpenAPI basePath cleanup (Phase 2 routes)

> Bug: Phase 2 routes (B2/B3/B4) ใช้ `basePath = '/api/v2/admin/...'` ใน OpenAPI registration ทั้งที่ Express mount จริงคือ `/api/v2/{features|packages|addons}` (ไม่มี `/admin`). Swagger จึงโชว์ path ผิด, client ที่ทดสอบผ่าน Swagger จะ hit URL ผิด → 404.
> Fix: Pure path string update — เปลี่ยน `basePath` constant + comment URL ทุกบรรทัดให้ตรงกับ Express mount จริง. ไม่แตะ Express routing / middleware chain.

- [x] **PURL.1** `happywork-backend/src/api/v2/sale-dashboard/feature/feature.routes.ts` — `basePath = '/api/v2/features'` + comment URLs (lines 27, 43, 53, 63, 73, 83) <!-- done: 2026-05-05 -->
- [x] **PURL.2** `happywork-backend/src/api/v2/sale-dashboard/packageCrud/packageCrud.routes.ts` — `basePath = '/api/v2/packages'` + comment URLs (lines 34, 50, 66, 83, 93, 103, 113, 123) <!-- done: 2026-05-05 -->
- [x] **PURL.3** `happywork-backend/src/api/v2/admin/addon/addon.routes.ts` — `basePath = '/api/v2/addons'` + comment URLs (lines 29, 45, 55, 65, 75, 85) <!-- done: 2026-05-05 -->
- [x] **PURL.4** Verify grep — ไม่มี `/api/v2/admin/{features,packages,addons}` หลงเหลือใน `src/` <!-- done: 2026-05-05 -->
- [x] **PURL.5** TS compile clean (`tsc --noEmit` exit 0) + Prettier formatted (3 files unchanged after run) <!-- done: 2026-05-05 -->

---

## Status Value Migration — Phase 2 modules: `archived` → `inactive`

> Decision: User สั่งให้ Phase 2 รับเฉพาะ `'active'` + `'inactive'` (ไม่ใช้ `'archived'`). Frontend ส่ง `inactive` มา. Phase 2 modules (feature, packageCrud, addon) เท่านั้น — modules อื่นที่ใช้ archived ไม่แตะ.
>
> **Investigation result (2026-05-05):**
>
> - **DB column type:** `status_type VARCHAR(25)` ไม่มี CHECK constraint, ไม่มี ENUM type (ดู `tableTemplate.ts`) → **ไม่ต้อง alter schema**
> - **DB current data:** comp_features=14 active, comp_packages=7 active, comp_addons=9 active (ตรวจ 2026-05-05) — **zero archived rows** → **ไม่ต้อง backfill data**
> - **Zod filter schema:** ทั้ง 3 interface ใช้ `z.enum(['active', 'inactive'])` อยู่แล้ว → **ไม่ต้องแก้ schema**
> - **Repo soft-delete literal:** ใช้ `StatusTypeEnumCode.ARCHIVED` ใน 3 ที่ (feature/packageCrud/addon repository soft-delete) → **ต้อง swap เป็น `INACTIVE`**
> - **Shared enum approach:** Option B (light) — ใช้ `StatusTypeEnumCode.INACTIVE` ที่มีอยู่แล้วใน `src/constant/general.ts`, ไม่แตะ enum (ไม่ลบ `ARCHIVED` กัน break modules อื่น)

- [x] **PSV.1** Investigation — ตรวจ DB schema, current data, code references; สรุป "ไม่ต้องสร้าง migration" <!-- done: 2026-05-05 -->
- [x] **PSV.2** `happywork-backend/src/modules/v2/sale-dashboard/feature/feature.repository.ts` — `softDeleteFeatureRepository` ใช้ `StatusTypeEnumCode.INACTIVE` แทน `ARCHIVED` + comment อัปเดต <!-- done: 2026-05-05 -->
- [x] **PSV.3** `happywork-backend/src/modules/v2/sale-dashboard/packageCrud/packageCrud.repository.ts` — `softDeletePackageRepository` ใช้ `StatusTypeEnumCode.INACTIVE` + comment ของ `getPackageByUuidIncludingArchivedRepository` + `getPackageFeaturesRepository` อัปเดต (ตัด wording archived) <!-- done: 2026-05-05 -->
- [x] **PSV.4** `happywork-backend/src/modules/v2/sale-dashboard/addon/addon.repository.ts` — `softDeleteAddonRepository` ใช้ `StatusTypeEnumCode.INACTIVE` <!-- done: 2026-05-05 -->
- [x] **PSV.5** `happywork-backend/src/modules/v2/sale-dashboard/packageCrud/packageCrud.service.ts` — comment ของ `updatePackageUuidService` (3 บรรทัด) อัปเดตจาก "archived" → "inactive ที่ถูก soft-delete" <!-- done: 2026-05-05 -->
- [x] **PSV.6** Verify grep — ไม่มี `'archived'` / `"archived"` / `StatusTypeEnumCode.ARCHIVED` หลงเหลือใน Phase 2 sale-dashboard modules (feature/packageCrud/addon) <!-- done: 2026-05-05 -->
- [x] **PSV.7** TS compile clean (`NODE_OPTIONS=--max-old-space-size=8192 npx tsc --noEmit` exit 0) <!-- done: 2026-05-05 -->
- [x] **PSV.8** Prettier formatted — 4 ไฟล์ unchanged after run (already conformant) <!-- done: 2026-05-05 -->
- [x] **PSV.9** DB re-verify หลัง code change — `SELECT status_type, COUNT(*) GROUP BY` ของ 3 tables ยังคงเป็น active เท่านั้น (ไม่มี state mutation จาก code change) <!-- done: 2026-05-05 -->
- [ ] **PSV.10** Smoke test — `GET /api/v2/sale-dashboard/features?statusType=archived` คาดหวัง 400 (Zod reject) — รอ user/QA verify บน dev server <!-- pending: requires dev server running -->

---

## Phase 3 CR Fix Wave (2026-05-06)

> Code review verdict: PASS-with-fixes (0 critical / 2 high / 3 medium / 4 low). 7 of 9 findings fixed in this wave; 2 deferred with rationale below.
> Source: `document/requirement/feature-management/cr-phase3.md`

### Fixed in this wave (7)

- [x] **CR-H1** Consolidate duplicate override-query — removed `getOverridesByCompanyRepository` from `companyFeature.repository.ts`; service now imports `getCompanyOverridesRepository` from `permissionResolver.repository.ts`; `OverrideRowDb` is now a type alias of `CompanyOverrideRow` to keep adapter API stable <!-- done: 2026-05-06 -->
  - Files: `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/{companyFeature.repository.ts, companyFeature.interface.ts, companyFeature.service.ts}`
- [x] **CR-H2** Removed `console.error` in 3 worker sagas + `console.log` startup banner from watcher in `happywork-sale-cms/src/store/sagas/sale-dashboard-company-features.ts`. Failure actions still surface errors via reducer → toast (no client logger added — none exists in project) <!-- done: 2026-05-06 -->
- [x] **CR-M1** Removed `'23505'` (unique_violation) from DB-error race-condition catch in `setCompanyFeatureOverrideService` — UPSERT's `ON CONFLICT` resolves it atomically so the code path is dead. Kept `'40001'` + `'40P01'`; added inline comment explaining why <!-- done: 2026-05-06 -->
  - File: `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts`
- [x] **CR-M2** Added `WHERE jsonb_typeof(f.menu_keys) = 'array'` guard to `getFeatureMenuKeysRepository` LATERAL `jsonb_array_elements_text` query — prevents 500 if any row has malformed `menu_keys` JSONB (defense at read path) <!-- done: 2026-05-06 -->
  - File: `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts`
- [x] **CR-M3** Eliminated duplicate Seed-fallback logic — extracted `resolveEffectivePackageIdService` into `permissionResolver.service.ts` (cached under `packageId:{companyId}` cache key); both `resolveCompanyEffectiveFeatureIdsService` and `companyFeature.service.buildCompanyFeatureList` now consume this single helper. Adapter still receives `packageFeatureIds` + `addonFeatureIds` separately for `source` attribution per SSD §3.3 <!-- done: 2026-05-06 -->
  - Files: `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts`, `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts`
- [x] **CR-L2** Dialog now stays open during in-flight submit. `handleConfirmSet` / `handleConfirmReset` no longer close synchronously; an effect watches `togglingFeatureUuid` and auto-closes when it transitions from `targetItem.featureUuid` → `null` (success or error). User now sees Confirm button's `isSubmitting` state. Cancel still closes synchronously <!-- done: 2026-05-06 -->
  - File: `happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx`
- [x] **CR-L4** `getCompanyPackageIdRepository` now selects only `packageId` (was over-fetching `id`) <!-- done: 2026-05-06 -->
  - File: `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts`

### Deferred (2)

- [ ] **CR-L1** `logger.error` with `{ level: 'warn' }` workaround pattern — defer to Phase 4 prep. Touches the cross-cutting diagnostic logger module; adding a real `logger.warn()` requires impact assessment beyond this wave's scope. NULL-package path is operationally rare today; will be hit on every permission request in Phase 4, so fix is required before Phase 4 ships.
- [ ] **CR-L3** `EMPTY_COUNTS` spread → reduce/`Object.fromEntries` — deferred. Reviewer flagged as cosmetic-only; current implementation is functionally correct and safe (shallow spread does not mutate the constant).

### Verification

- TS type-check (`tsc --noEmit`): both `happywork-backend` + `happywork-sale-cms` pass clean (backend run with `NODE_OPTIONS=--max-old-space-size=8192` due to project size, not related to this wave)
- Prettier (`prettier --write`): all 7 edited files conformant
- Grep verifications:
  - `getOverridesByCompanyRepository`: 0 callers/decls remaining (only doc-comment references)
  - `console.*` in saga file: 0
  - `'23505'` in companyFeature.service: 0
  - `jsonb_typeof(f.menu_keys) = 'array'`: present in repo
  - `resolveEffectivePackageIdService`: 1 export + 2 call sites (resolver self + companyFeature service)
  - `getCompanyPackageIdRepository`: `.select('packageId')` (single column)

---

## Phase 5: Mobile Menu Parity (Added 2026-05-06)

> Mirror PermissionMobileDefault end-to-end. Plan: `/Users/ball/.claude/plans/lead-menu-elegant-finch.md`. Cleanup เลื่อนเป็น Phase 6.

### Phase 5 Wave 1: Schema + Helper (Backend)

- [x] **P5.1.1** สร้าง migration M9 — `happywork-backend/src/database/postgresql/migrations/data/20260506-1800-alter-comp_features-add-mobile_menu_keys.ts` <!-- done: 2026-05-06 -->
  - [x] up: ALTER TABLE comp_features ADD mobile_menu_keys JSONB NOT NULL DEFAULT '[]'::jsonb
  - [x] up: CREATE INDEX idx_comp_features_mobile_menu_keys_gin ... USING GIN (mobile_menu_keys)
  - [x] down: DROP INDEX + DROP COLUMN
  - [ ] verify: รัน migrate → column + GIN index มีจริง _(deferred — DB shared, Lead/DevOps จะรันเอง)_
- [x] **P5.1.2** Confirm mapping mobile_menu_keys ของ SEED tier 4 features กับ user (basic_attendance / basic_leave / basic_reports / max_10_users) ก่อน implement M10 <!-- done: 2026-05-06 — user-confirmed Q7=A+B mapping -->
- [x] **P5.1.3** สร้าง migration M10 — `20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts` (data migration backfill SEED tier ตาม mapping; idempotent ผ่าน `WHERE code = ? AND status_type = 'active'`) <!-- done: 2026-05-06 -->
- [x] **P5.1.4** Update model — `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts` เพิ่ม `mobileMenuKeys!: string[]` ใส่ใน `jsonAttributes` (+ jsonSchema entry) <!-- done: 2026-05-06 -->
- [x] **P5.1.5** เพิ่ม helper `getAvailableMobileMenuKeys()` ใน `happywork-backend/src/constant/compPermission.ts` — refactor walker เป็น parameterized ACTION_KEYS (web=5 / mobile=4 ไม่มี export); อ่าน leaf paths จาก `PermissionMobileDefault` (Dev/Prod variants ผ่าน env-aware export ที่มีอยู่แล้ว) <!-- done: 2026-05-06 -->

### Phase 5 Wave 2: Feature CRUD extension (Backend)

- [x] **P5.2.1** `happywork-backend/src/modules/v2/sale-dashboard/feature/feature.interface.ts` เพิ่ม `mobileMenuKeys: z.array(z.string()).default([])` ใน input/output Zod <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `mobileMenuKeys` ใน `featureBaseSchema` (default `[]` → optional via `partial()` ใน updateFeatureSchema)
  - [x] เพิ่ม `mobileMenuKeys: readonly string[]` ใน `Feature`, `CreateFeatureRepositoryInput`, `UpdateFeatureRepositoryInput`
  - [x] เพิ่ม `menuKeysAvailableResponseSchema` + `MenuKeysAvailableResponse` type `{ web, mobile }` (breaking — ห้าม keep `menuKeys` legacy)
- [x] **P5.2.2** `feature.repository.ts` INSERT/UPDATE/SELECT include `mobile_menu_keys` <!-- done: 2026-05-06 -->
  - [x] `createFeatureRepository` insert `mobileMenuKeys: [...input.mobileMenuKeys]`
  - [x] `updateFeatureRepository` patch `mobileMenuKeys` only if `!== undefined` (preserve partial-update semantics)
  - [x] SELECT path: Objection auto-includes all columns ผ่าน model jsonAttributes — ไม่ต้องแก้ `buildBaseQuery`/`findById`/`findOne`
- [x] **P5.2.3** `feature.service.ts` + `feature.adapter.ts` pass-through field <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `validateMobileMenuKeys` helper (mirror `validateMenuKeys`); ใช้ `getAvailableMobileMenuKeys()` + error code `INVALID_MOBILE_MENU_KEY` (P5.2.5)
  - [x] `createFeatureService` call `validateMobileMenuKeys(input.mobileMenuKeys)` ก่อน insert
  - [x] `updateFeatureService` call `validateMobileMenuKeys` ถ้า `input.mobileMenuKeys !== undefined`
  - [x] `convertFeatureToResponse` (adapter) map `row.mobileMenuKeys` → `mobileMenuKeys` (defensive `Array.isArray` guard)
- [x] **P5.2.4** **BREAKING:** `feature.controller.ts` `getAvailableMenuKeysController` response shape เปลี่ยนเป็น `{web: string[], mobile: string[]}` (Q3=B) <!-- done: 2026-05-06 -->
  - [x] `getAvailableMenuKeysService` returns `{ web: getAvailableMenuKeys(), mobile: getAvailableMobileMenuKeys() }`
  - [x] Controller passes service result through directly (no wrap in `{ menuKeys }`)
- [x] **P5.2.5** Validate mobile_menu_keys ทุก key อยู่ใน `getAvailableMobileMenuKeys()` (reject 400 ถ้าไม่ใช่) <!-- done: 2026-05-06 -->
  - [x] Error code `INVALID_MOBILE_MENU_KEY` (distinct from web `INVALID_MENU_KEY` — frontend Wave 6 จะ surface แยก field)
  - [x] Error payload includes `{ invalidKeys: [...] }` array สำหรับ UX feedback

### Phase 5 Wave 3: Permission Resolver (Backend)

- [x] **P5.3.1** `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts` — `resolveCompanyEffectiveMobileMenuKeysService(companyId)` mirror <!-- done: 2026-05-06 -->
  - [x] Reuse `resolveCompanyEffectiveFeatureIdsService` (memoized) → ไม่ duplicate package/addon/override pipeline
  - [x] Cache key `mobileMenuKeys:{companyId}` ผ่าน `ResolverCache` (กัน duplicate call ภายใน request เดียว)
- [x] **P5.3.2** `filterPermissionMobileByMenuKeysService(jsonb, allowed)` mirror <!-- done: 2026-05-06 -->
  - [x] Delegate to `filterPermissionByMenuKeysService` — walker action-key-set-agnostic (mobile leaves เป็น subset ของ web ACTION_KEYS)
  - [x] Caller-readability: separate function exported (caller อ่านชัดว่าเป็น mobile flow)
- [x] **P5.3.3** `extractMobileMenuKeysFromPermissionJsonbService(jsonb)` mirror <!-- done: 2026-05-06 -->
  - [x] Delegate to `extractMenuKeysFromPermissionJsonbService` (ดู P5.3.2 rationale)
- [x] **P5.3.4** `permissionResolver.repository.ts` aggregate `mobile_menu_keys` จาก effective features (reuse `resolveCompanyEffectiveFeatureIdsService`) <!-- done: 2026-05-06 -->
  - [x] `getFeatureMobileMenuKeysRepository(featureIds[])` — pre-fetch + LATERAL `jsonb_array_elements_text(f.mobile_menu_keys)` + `WHERE IN`
  - [x] M2-style guard: `jsonb_typeof(f.mobile_menu_keys) = 'array'` กัน corrupt jsonb crash
  - [x] Filter `feature.status_type='active'` (Q4=B mirror ของ web counterpart)
- [x] **P5.3.5** In-request cache extend ให้รวม mobile keys + filtered mobile permission <!-- done: 2026-05-06 -->
  - [x] `permissionResolver.interface.ts` JSDoc — append `mobileMenuKeys:{companyId}` + `packageId:{companyId}` ใน cache keys list
  - [x] `MOBILE_MENU_KEYS_KEY` constant ใน service (mirror `MENU_KEYS_KEY`)
  - [x] No N+1: `getFeatureMobileMenuKeysRepository` ถูก call 1 ครั้ง/resolver invocation; cache hit `mobileMenuKeys:{companyId}` → 0 call

### Phase 5 Wave 4: External Auth (Backend)

- [x] **P5.4.1** `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts` — `getPermissionsService` filter ทั้ง `permission` + `permission_mobile` <!-- done: 2026-05-06 -->
  - [x] เพิ่ม `resolveUserEffectivePermissionMobileService` ใน resolver (mirror web — รับ optional `cache?: ResolverCache`); reuse `getUserPermissionJsonbRepository`
  - [x] Cache shared (single `Map`) ระหว่าง web + mobile call → `featureIds:{companyId}` + `packageId:{companyId}` hit cache; mobile path เพิ่มแค่ batch `getFeatureMobileMenuKeysRepository` 1 call
  - [x] PERMISSION_DATA_NOT_FOUND graceful → `permissionMobile = {}`; PERMISSION_DATA_CORRUPTED → propagate
- [x] **P5.4.2** `/api/external/auth/v1/permissions/:userUuid` response เพิ่ม `permissionMobile` (Phase 4 W1 pattern — empty `{}` graceful, NOT skeleton; consistency กับ web `permission` field) <!-- done: 2026-05-06 -->
  - [x] `permissions.interface.ts` — `PermissionsResponse.permissionMobile: Record<string, unknown>` additive (backward compat)
  - [x] Override checklist text: ใช้ `{}` graceful per task spec consistency note (Phase 4 W1 pattern); reasoning: skeleton-with-false จะ surface UI ที่ไม่ effective — `{}` ตรงกับ Q-C policy
  - [x] Controller untouched — response shape extends ผ่าน return type
  - [x] Repository extend: `getUserPermissionJsonbRepository` select `cp.permission_mobile` ด้วยใน round trip เดียว → `UserPermissionRow.permissionMobileJsonb: PermissionJsonb | null`
- [x] **P5.4.3** `getEmployeePermissionService` (Phase 4 Fix 1 ที่ `compPermissionMapping.service.ts`) extend mobile path <!-- done: 2026-05-06 -->
  - [x] BUG FIX: เดิม mobile branch ใช้ `filterPermissionByMenuKeysService` + web `allowedKeys` ทับ mobile baseline → ผิด → เปลี่ยนเป็น `filterPermissionMobileByMenuKeysService` + `allowedMobileKeys`
  - [x] เพิ่ม `resolveCompanyEffectiveMobileMenuKeysService` call (parallel กับ web ผ่าน `Promise.all` — shared cache → mobile call hit `featureIds`+`packageId` cache, เพิ่มแค่ batch mobile_menu_keys lookup)
  - [x] Q-A=STRICT enforced ใน mobile path เช่นเดียวกับ web — owner ไม่ bypass filter (baseline filter ก่อน deepMerge)
  - [x] Post-merge re-filter pass mirror web (Q-C=no-migration policy) — ใช้ `allowedMobileKeys` ที่ resolved แล้ว, ไม่เรียก resolver ซ้ำ
- [x] **Test update** `tests/unit/externalAuth/permissions.service.test.ts` — 7 cases covering both branches + cache-share assertion <!-- done: 2026-05-06 -->

### Phase 5 Wave 5: compPermission admin BUG FIX + mobile validation (Backend)

- [x] **P5.5.1** `happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts` — split validation: <!-- done: 2026-05-06 -->
  - [x] `permission` JSONB → validate ผ่าน effective web menu keys (เดิม)
  - [x] `permission_mobile` JSONB → validate ผ่าน effective **mobile** menu keys (FIX BUG)
  - [x] BUG FIX: `assertPermissionKeysWithinCompanyMenusService` เดิมรวม keys ของ web + mobile แล้ว validate ผ่าน effective web menu keys อันเดียว → admin save mobile permission ที่มี mobile keys → reject 400 invalid key (หรือถ้า key ชื่อบังเอิญตรง → save ผ่านแต่ผิด context); ตอนนี้ split 2 path independent + cache shared
  - [x] Early-return optimization (CR M1) คงไว้ทั้ง 2 path — empty `{}` ไม่เรียก resolver
  - [x] catch handler ทั้ง create + update preserve `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` ด้วย
- [x] **P5.5.2** เพิ่ม error code `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` (HTTP 400) <!-- done: 2026-05-06 -->
  - [x] ใช้ `errorBadRequest()` กับ `data.invalidKeys` list (mirror web counterpart)
  - [x] Distinct error code (ไม่ใช่ same as `INVALID_MENU_KEY_FOR_COMPANY`) — frontend แยก toast/inline message ได้
- [x] **P5.5.3** Audit `getCompPermissionByUuidController` (deepMerge baseline) — ใช้ mobile filter ด้วย <!-- done: 2026-05-06 -->
  - [x] Service `getCompPermissionByUuidService` filter `permissionMobile` ผ่าน `filterPermissionMobileByMenuKeysService` + `allowedMobileKeys` (เดิมใช้ web `allowedKeys` ผิด namespace) — `Promise.all` parallel resolve
  - [x] Controller `getCompPermissionByUuidController` filter `PermissionMobileDefault` baseline ผ่าน mobile filter + mobile keys (เดิมใช้ web `allowedKeys` ทั้ง 2 baseline) — cache shared
- [x] **P5.5.4** Audit `/comp-permission/default?companyUuid=` — ใช้ mobile filter ด้วย <!-- done: 2026-05-06 -->
  - [x] `getCompPermissionDefaultListController` resolve ทั้ง web + mobile menu keys (parallel ผ่าน `Promise.all`, shared cache → mobile hit `featureIds`+`packageId` cache)
  - [x] filter `PermissionDefault` ผ่าน web keys, filter `PermissionMobileDefault` ผ่าน mobile keys (shape `{permission, permissionMobile}` คงเดิม — ไม่ break)
- [x] **P5.5.5** Audit owner STRICT — propagate ไป mobile path <!-- done: 2026-05-06 -->
  - [x] ทุก mobile filter call ใน Wave 5 ใช้ baseline-filter-before-deepMerge pattern (เหมือน W3.5/W4) → owner=true ไม่ bypass effective menu filter — Q-A=STRICT enforced ใน mobile namespace แล้ว
  - [x] `getEmployeePermissionService` (W4 ทำแล้ว — covered transitively); `getCompPermissionByUuidController` (Wave 5 W5 fix); `/comp-permission/default` (Wave 5 W5 fix); pre-save validation (Wave 5 W5 fix) — **ครบทุก endpoint ที่แตะ mobile namespace**

### Phase 5 Wave 6: Type + Redux + Service (Frontend)

- [x] **P5.6.1** `happywork-sale-cms/src/types/store/sale-dashboard-feature-management.ts` — Feature type เพิ่ม `mobileMenuKeys: string[]`; `AvailableMenuKeysPayload` → `{web: string[], mobile: string[]}` <!-- done: 2026-05-06 -->
  - [x] `Feature` type field `mobileMenuKeys: string[]` (additive — non-optional; backend defaults `[]` for legacy rows)
  - [x] `CreateFeatureRequest.mobileMenuKeys?: string[]` optional (backend defaults `[]`); `UpdateFeatureRequest.data.mobileMenuKeys?: string[]`
  - [x] **BREAKING (Q3=B)** renamed `AvailableMenuKeysResponse` → `AvailableMenuKeysPayload` with `{web: string[]; mobile: string[]}` shape
  - [x] Root state slot `availableMenuKeys: AvailableMenuKeysPayload` (was `string[]`)
- [x] **P5.6.2** Redux slice (action + reducer + saga + service) sale-dashboard-feature-management รองรับ field ใหม่ <!-- done: 2026-05-06 -->
  - [x] Reducer initial state `availableMenuKeys: { web: [], mobile: [] }`
  - [x] `sdGetAvailableMenuKeysSuccess` handler maps backend `{web, mobile}` envelope (defensive `Array.isArray` per namespace)
  - [x] Action types unchanged (payload shape extends via TS only)
  - [x] Saga unchanged (response.data passes through to reducer; service `getAvailableMenuKeysRequest` JSDoc updated)
  - [x] Hook `useSaleDashboardFeatureManagement` return type auto-updates via state slot — `useCallback` dispatcher refs already wrapped (Phase 2 max-depth bug pattern preserved)
  - [x] **Caller breakage (Wave 7 will fix):** 5 sites consuming `availableMenuKeys` as flat `string[]` were patched with `availableMenuKeys.web` bridge + Wave 7/8 TODO comment to keep TS clean and pre-Wave-6 Web-only behavior intact:
    - `src/sections/feature-management/feature-create-view.tsx` (FeatureForm prop) — Wave 7 P5.7.3
    - `src/sections/feature-management/feature-edit-view.tsx` (FeatureForm prop) — Wave 7 P5.7.3
    - `src/sections/feature-management/feature-detail-view.tsx` (MenuTreePreview prop) — Wave 7 P5.7.4
    - `src/sections/feature-management/components/feature-form.tsx` (consumes prop downstream — type unchanged at component boundary; Wave 7 P5.7.3 will add `webMenuKeys`/`mobileMenuKeys` props)
    - `src/sections/package-management/components/package-effective-menus-preview.tsx` (MenuTreePreview prop + length check) — Wave 8 P5.8.1 (Page D side-by-side); current Page B preview surfaces only Web for now
  - [x] `src/sections/feature-management/utils/feature-form.ts` Yup schema option `availableMenuKeys?: string[]` unchanged at boundary (callers already filter to a flat list before passing) — Wave 7 will refactor when Mobile col added
- [ ] **P5.6.3** Redux slice company-features ขยาย response include `effectiveMobileMenuKeys` + `permissionMobile` — **deferred to Wave 8**: backend `companyFeature` endpoint ยังไม่ extend mobile (Wave 4 ขยาย `/external/auth/permissions` + `/auth/user-info` แต่ companyFeature endpoint แยก), so FE slice ไม่มี field ให้ sync; Wave 8 จะ pickup เมื่อ backend extend companyFeature endpoint หรือ Page D ต้องการแสดง mobile column

### Phase 5 Wave 7: Feature CRUD UI (Page A — Frontend)

- [x] **P5.7.1** `src/components/feature-management/menu-keys-select.tsx` รองรับ side-by-side mode (discriminated union `mode: "single" | "split"`) <!-- done: 2026-05-06 -->
  - [x] Internal `MenuKeysColumn` extracted (search + grouped checkbox tree) ใช้ร่วมทั้ง 2 modes
  - [x] `mode="split"` รับ `webOptions/webValue/onWebChange/webLabel/webError/webHelperText` + mobile counterparts; render `Grid xs=12 md=6` 2 columns
  - [x] `mode="single"` (default) คง backward compat — ไม่มี caller อื่นใน sale-cms (single caller = `feature-form.tsx` migrated)
  - [x] Empty state per column ใช้ i18n keys `featureManagement.menu_keys.no_web_available` / `no_mobile_available`
- [x] **P5.7.2** `src/components/feature-management/menu-tree-preview.tsx` รองรับ side-by-side display (discriminated union `mode: "single" | "split"`) <!-- done: 2026-05-06 -->
  - [x] Internal `MenuTreeColumn` extracted (group by category, show enabled leaves only) ใช้ร่วมทั้ง 2 modes
  - [x] `mode="split"` รับ `webMenuKeys/mobileMenuKeys` + optional `webAvailableKeys/mobileAvailableKeys` (ใช้ filter visible groups) + per-side `emptyMessage`/`label`
  - [x] Header per column = label + count (เช่น "Web Menus (3)") + iconify monitor/smartphone icon
  - [x] `mode="single"` (default) คง backward compat สำหรับ Wave 8 caller `package-effective-menus-preview.tsx` (ยังใช้ Wave 6 bridge `availableMenuKeys.web`)
- [x] **P5.7.3** `src/sections/feature-management/components/feature-form.tsx` render Web col + Mobile col <!-- done: 2026-05-06 -->
  - [x] `Props.availableMenuKeys` type changed `string[]` → `AvailableMenuKeysPayload` (Q3=B breaking — caller migration in P5.7.6/P5.7.7)
  - [x] `createFeatureSchema` รับ option ใหม่ `availableMobileMenuKeys` + Yup test `mobile-menu-keys-valid` (error key `featureManagement.validation.mobile_menu_keys_invalid`)
  - [x] Section "menu_keys" render `<MenuKeysSelect mode="split">` ผ่าน nested Controller (ใส่ `name="menuKeys"` outer + `name="mobileMenuKeys"` inner; เก็บ field.onChange ตรงๆ ไม่ wrap useCallback)
  - [x] Error surface: `errors.menuKeys.message` → `webError`; `errors.mobileMenuKeys.message` → `mobileError` — distinct
- [x] **P5.7.4** `src/sections/feature-management/feature-detail-view.tsx` render side-by-side preview <!-- done: 2026-05-06 -->
  - [x] Drop Wave 6 bridge TODO comment + เปลี่ยน `<MenuTreePreview>` ใช้ `mode="split"` รับ `webMenuKeys={detail.menuKeys}` + `mobileMenuKeys={detail.mobileMenuKeys ?? []}` + per-side `webAvailableKeys/mobileAvailableKeys` + `webEmptyMessage/mobileEmptyMessage`
- [x] **P5.7.5** i18n: เพิ่ม keys `web_menu_keys`, `mobile_menu_keys`, helpers, validation, empty states (en+th parity, follow sale-cms snake_case nested convention) <!-- done: 2026-05-06 -->
  - [x] `featureManagement.section.{web_menu_keys, mobile_menu_keys}`
  - [x] `featureManagement.form.{web_menu_keys, web_menu_keys_helper, mobile_menu_keys, mobile_menu_keys_helper}`
  - [x] `featureManagement.menu_keys.{web_empty, mobile_empty, no_web_available, no_mobile_available}`
  - [x] `featureManagement.validation.mobile_menu_keys_invalid` + reword `menu_keys_invalid` → "web menu key is invalid" (distinct error message per namespace)
  - [x] Q4=Open resolved: chose nested `web_menu_keys` / `mobile_menu_keys` (snake_case) consistent with existing `featureManagement.form.menu_keys` convention; rejected `webMenus`/`mobileMenus` flat camelCase form
  - [x] EN + TH parity verified: `featureManagement` key count = 64/64
- [x] **P5.7.6** Wave 6 bridge cleanup ใน `feature-create-view.tsx` + `feature-edit-view.tsx` — drop `availableMenuKeys.web` selector + ส่ง `availableMenuKeys` whole object ลง `<FeatureForm>` (TODO Wave 7 comments removed) <!-- done: 2026-05-06 -->
- [x] **P5.7.7** `src/sections/feature-management/utils/feature-form.ts` — Form types/defaults/Yup/payload builders ขยาย `mobileMenuKeys: string[]` <!-- done: 2026-05-06 -->
  - [x] `FeatureFormData.mobileMenuKeys: string[]`; `DEFAULT_FEATURE_FORM_VALUES.mobileMenuKeys = []`
  - [x] `mapFeatureToFormValues` map `feature.mobileMenuKeys ?? []`
  - [x] `buildCreateFeaturePayload` + `buildUpdateFeaturePayload` send `mobileMenuKeys: data.mobileMenuKeys ?? []`
  - [x] `createFeatureSchema` รับ optional `availableMobileMenuKeys?: string[]`; mobile array test mirrors web
- [x] **P5.7.V** Verification — TS clean (`npx tsc --noEmit`) + prettier conformant + i18n parity 64/64 + grep no Wave 7 TODOs left in feature-management section <!-- done: 2026-05-06 -->

### Phase 5 Wave 8: Page D Company Features Tab (Backend extension + Frontend) <!-- done: 2026-05-06 -->

> Lead approval 2026-05-06: Wave 8 includes BE extension because Page D's mobile column has no data otherwise. Decision (Q-shape resolved): per-row `menuKeys`+`mobileMenuKeys` (additive on `CompanyFeatureItem`) **plus** top-level `effectiveMenuKeys`+`effectiveMobileMenuKeys` aggregate sets — flexible for both row-level affordances and aggregate side-by-side preview. Backward compat: pre-Wave-8 callers reading `data.list` still work.

- [x] **P5.8.BE.1** Backend `companyFeature.interface.ts` extend `CompanyFeatureItem` with `menuKeys: readonly string[]` + `mobileMenuKeys: readonly string[]`; extend `CompanyFeatureListResponse` with `effectiveMenuKeys` + `effectiveMobileMenuKeys` aggregate fields <!-- done: 2026-05-06 -->
- [x] **P5.8.BE.2** Backend `companyFeature.adapter.ts` — `computeCompanyFeatureItem` extract `feature.menuKeys` + `feature.mobileMenuKeys` per row (defensive `Array.isArray`) <!-- done: 2026-05-06 -->
- [x] **P5.8.BE.3** Backend `companyFeature.service.getCompanyFeaturesService` — change return type to `CompanyFeatureListResponse`; resolve effective menu keys via `resolveCompanyEffectiveMenuKeysService` + `resolveCompanyEffectiveMobileMenuKeysService` (shared `ResolverCache` Map → 2 batch DB calls only); sort sets for stable response <!-- done: 2026-05-06 -->
- [x] **P5.8.BE.4** Backend `companyFeature.controller.getCompanyFeaturesController` — pass full envelope through `res.success(result, ...)` (drop `{ list }` wrap) <!-- done: 2026-05-06 -->
- [x] **P5.8.1** `src/sections/client-management/feature-tab/components/company-effective-menus-preview.tsx` (NEW) render Web col + Mobile col via `<MenuTreePreview mode="split">` reading from slice's `effectiveMenuKeys`/`effectiveMobileMenuKeys`; uses `availableMenuKeys.{web,mobile}` for namespace catalog filtering — mounted in `company-features-tab-view.tsx` after the feature list <!-- done: 2026-05-06 -->
  - Decision: created **new** Page-D-specific component instead of refactoring `package-management/components/package-effective-menus-preview.tsx` (Page B preview reads per-PACKAGE state via `useSaleDashboardPackageManagement`; Page D reads per-COMPANY via `useSaleDashboardCompanyFeatures`). Page B preview keeps single-mode (Wave 6 bridge stable for Web-only Phase B preview).
- [x] **P5.8.2** `src/sections/client-management/feature-tab/components/company-feature-row.tsx` — render per-row Web/Mobile menu count chips (icon + count) wrapped in tooltip "Unlocks N web / M mobile menus"; defensive `?? 0` so pre-Wave-8 backend payload doesn't break the row <!-- done: 2026-05-06 -->
- [x] **P5.8.3** Redux slice (P5.6.3 deferred from Wave 6) — extend `CompanyFeatureItem` type with optional `menuKeys?` + `mobileMenuKeys?`; extend Root state with `effectiveMenuKeys: string[]` + `effectiveMobileMenuKeys: string[]`; `sdGetCompanyFeaturesSuccess` reducer populates new fields with `Array.isArray` defensive guard; hook `useSaleDashboardCompanyFeatures()` exposes new fields via state spread <!-- done: 2026-05-06 -->
- [x] **P5.8.4** i18n — added 5 keys under `companyFeatures.*` (en+th parity 42/42): `effective_menus.{title, subtitle, web_empty, mobile_empty}` + `row.menu_keys_tooltip` (interpolated `{{web}}`/`{{mobile}}`) <!-- done: 2026-05-06 -->
- [x] **P5.8.V** Verification — TS clean (BE + sale-cms `npx tsc --noEmit` 0 errors); prettier conformant on all 11 modified files; en/th `companyFeatures` parity 42/42; no Wave 6 bridge TODOs left in feature-tab section <!-- done: 2026-05-06 -->

### Phase 5 Wave 9: QA Combined Parameterized Tests

- [ ] **P5.9.1** Extend Phase 2 unit tests cover mobile_menu_keys (combined parameterized web+mobile)
- [ ] **P5.9.2** Extend Phase 3 resolver tests (32 → cover mobile path)
- [ ] **P5.9.3** Extend Phase 4 integration tests (25 → cover external auth permissionMobile + compPermission split-bug regression)
- [ ] **P5.9.4** New test: `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` 400

### Phase 5 Wave 10: Code Review

- [x] **P5.10.1** CR pass (single after W1-W9) <!-- done: 2026-05-06; PASS-with-fixes — see cr-phase5.md -->
- [x] **P5.10.2** CR Fix Wave (5 fixed + 2 deferred) <!-- done: 2026-05-07; see "Phase 5 CR Fix Wave" below -->

## Phase 5 CR Fix Wave (2026-05-07)

### CR Fixes (5 ticked + 2 deferred)

Reviewer verdict: PASS-with-fixes (BLOCKED จนกว่า H1 fix). Fix targets enumerated ใน
`document/requirement/feature-management/cr-phase5.md`. Skipped: M3 (walker mobile/web action set —
documentation-only design smell) + L3 (FE implicit dependency on `availableMenuKeys` — UX-level,
constraint comment sufficient).

- [x] **CR-H1** Eliminate duplicate `getUserPermissionJsonbRepository` call per request
      <!-- done: 2026-05-07 -->
      - Added cache key `userPermissionJsonb:{userUuid}` ใน ResolverCache (interface JSDoc updated)
      - New helper `loadUserPermissionRowCached` ใน `permissionResolver.service.ts` รวม lookup + cache
      - Both `resolveUserEffectivePermissionService` + `resolveUserEffectivePermissionMobileService`
        ใช้ helper เดียวกัน → 2 → 1 DB hit ต่อ `/permissions/:uuid` request
      - Updated `permissions.service.ts` JSDoc (Performance section) เพื่อ reflect H1 fix
      - Test TC-23 + cross-cache test updated to assert `getUserPermissionJsonb.toHaveBeenCalledTimes(1)`
- [x] **CR-M1** `buildCompanyFeatureList` รับ `cache?: ResolverCache` + memoize 4 batch queries
      <!-- done: 2026-05-07 -->
      - Service-level cache keys (`cf:packageFeatureIds:{packageId}`, `cf:activeAddonIds:{companyId}`,
        `cf:overrides:{companyId}`, `cf:activeFeatures`, `cf:addonFeatureIds:{...}`) ป้องกัน duplicate
        ระหว่าง list build + 2 resolver calls
      - JSDoc cache-share rationale updated to enumerate new keys
      - Test TC-2 updated for new signature `(companyId, cache)` ที่ resolveEffectivePackageId rcv
- [x] **CR-M2** Migration M10 sequential `for...await` → `Promise.all`
      <!-- done: 2026-05-07 -->
      - 4 row-targeted UPDATEs are independent (different `code`, partial unique on `status_type='active'`)
      - Both `up()` + `down()` converted; comment เพิ่ม rationale
- [ ] **CR-M3** (deferred) Walker `hasActionKey` มี 5 web action keys ใช้กับ mobile JSONB
      - Documentation-only design smell — current `Array.some()` semantic correct (mobile leaves
        always have `read` → matched). Defer พร้อม comment ใน existing JSDoc (`extractMobile...` block
        line 319-325 ของ permissionResolver.service.ts)
- [x] **CR-L2** `createCompPermissionService` catch ordering — generalize CustomError pass-through
      <!-- done: 2026-05-07 -->
      - Added structural type-guard `isCustomHttpError(error)` ใน compPermission.service.ts header
      - Refactored `createCompPermissionService` + `updateCompPermissionByUuidService` catch blocks:
        check structural HTTP error first → AppError → wrap-as-500 fallback
      - Removes brittle `errCode === 'INVALID_MENU_KEY_FOR_COMPANY'` exact-match list
- [ ] **CR-L1** (deferred) `OwnerPermissionMobile` JSON round-trip clone
      - Cosmetic; module-load 1x; outside Phase 5 scope (Phase 1 code per CR note)
- [ ] **CR-L3** (deferred) Frontend `company-effective-menus-preview.tsx` implicit `availableMenuKeys`
      - UX-level; current consumer flows always trigger Page A load before Page D access
      - Future improvement: dispatch `sdGetAvailableMenuKeysRequest` ใน Page D mount (separate task)

### Verification

- TS: `NODE_OPTIONS="--max-old-space-size=8192" npx tsc --noEmit` exit 0
- Prettier: applied to all 7 modified files (1 file reformatted by prettier — migration M10)
- Unit tests:
  - `permissionResolver.service.test.ts` — 57/57 pass (TC-23 + cross-cache test updated for H1)
  - `permissions.service.test.ts` — pass (resolver mocked, no signature change)
  - `companyFeature.service.test.ts` — 39/39 pass (TC-2 updated for `resolveEffectivePackageId(id, cache)`)
- Integration tests:
  - `permission-resolver.integration.spec.ts` — pass
  - `companyFeature.integration.spec.ts` — pass
  - `feature-management.integration.spec.ts` — pre-existing failure (cannot find
    `@/utils/responseFormatter`) — unrelated to CR fix wave; documented as pre-existing in handoff

### Files modified (7)

- `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts` (H1 — cache helper)
- `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.interface.ts` (H1 — JSDoc)
- `src/modules/v1/externalAuth/permissions.service.ts` (H1 — JSDoc Performance)
- `src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts` (M1 — cache passthrough)
- `src/database/postgresql/migrations/data/20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts` (M2)
- `src/modules/v1/admin/compPermission/compPermission.service.ts` (L2 — type-guard)
- `tests/unit/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.test.ts` (H1 assertion)
- `tests/unit/modules/v2/sale-dashboard/companyFeature/companyFeature.service.test.ts` (M1 assertion)

### Phase 5 Wave 11: Closeout

- [ ] **P5.11.1** BA validation Phase 5
- [ ] **P5.11.2** Push hw-be + hw-sale-cms branch `ball/feature/feature-management`
- [ ] **P5.11.3** Update `.ai-memory/current.md` Active Feature → Phase 5 closeout
- [ ] **P5.11.4** Update summary.md Progress Log entry

### Phase 5 Doc Update Wave (parallel)

- [ ] **P5.D.1** ssd.md — เพิ่ม mobile schema + endpoint shapes + resolver mobile path + compPermission BUG fix entry (ใช้เป็น reference สำหรับ implementation team)
- [ ] **P5.D.2** backend.md — playbook for mobile API contract
- [ ] **P5.D.3** frontend.md — playbook for Web|Mobile UI pattern
- [ ] **P5.D.4** testcase.md — เพิ่ม mobile parity test cases (combined parameterized)
- [ ] **P5.D.5** cr.md — extend code-review checklist + log Phase 4 W2 BUG fix entry
- [ ] **P5.D.6** prd.md — ขยาย UX (Web|Mobile cols, helper bar)

---

## Unit Test Tracker

| Test Case                                                                                               | ผลที่คาดหวัง                                                | ผลการ Test | สถานะ      |
| ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ---------- | ---------- |
| Create feature with valid menu_keys                                                                     | สำเร็จ, return Feature                                      | -          | - [ ] ผ่าน |
| Create feature with invalid menu_key                                                                    | reject 400                                                  | -          | - [ ] ผ่าน |
| Create feature with duplicate code                                                                      | reject 400                                                  | -          | - [ ] ผ่าน |
| Delete feature with package usage                                                                       | reject 400                                                  | -          | - [ ] ผ่าน |
| Delete feature without usage                                                                            | success 200                                                 | -          | - [ ] ผ่าน |
| Replace package features                                                                                | features ของ package update ตรง                             | -          | - [ ] ผ่าน |
| Get package effective menus                                                                             | dedupe ครบทุก feature                                       | -          | - [ ] ผ่าน |
| Create addon with is_quantifiable=true, no max_quantity                                                 | reject 400                                                  | -          | - [ ] ผ่าน |
| Toggle company feature (enabled override)                                                               | upsert row ใน comp_company_features                         | -          | - [ ] ผ่าน |
| Get company features                                                                                    | คืน features พร้อม source ตาม mapping                       | -          | - [ ] ผ่าน |
| Resolver: package only                                                                                  | effective = package features                                | -          | - [ ] ผ่าน |
| Resolver: package + addon                                                                               | effective = union                                           | -          | - [ ] ผ่าน |
| Resolver: package + override disabled                                                                   | effective ไม่มี feature นั้น                                | -          | - [ ] ผ่าน |
| Resolver: package + override enabled (feature นอก package)                                              | effective มี feature เพิ่ม                                  | -          | - [ ] ผ่าน |
| Filter permission JSONB                                                                                 | leaf path ที่ไม่อยู่ใน allowed → ตัดออก                     | -          | - [ ] ผ่าน |
| Permission API for company with disabled feature                                                        | response permission ไม่มี key นั้น                          | -          | - [ ] ผ่าน |
| RBAC: super_admin → see Feature Management menu                                                         | เห็นเมนู                                                    | -          | - [ ] ผ่าน |
| RBAC: admin → not see Feature Management menu                                                           | ไม่เห็นเมนู                                                 | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: Create feature with valid mobile_menu_keys                                          | success, return Feature with both menuKeys + mobileMenuKeys | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: Create feature with invalid mobile menu_key                                         | reject 400                                                  | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: GET /features/menu-keys/available shape                                             | response = `{web: [...], mobile: [...]}`                    | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: External auth response include permissionMobile filtered                            | leaf path ที่ไม่อยู่ใน mobile effective → ตัดออก            | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: compPermission save with invalid permission_mobile key                              | reject 400 `INVALID_MOBILE_MENU_KEY_FOR_COMPANY`            | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: Phase 4 W2 BUG fix regression — permission_mobile with key in web only (not mobile) | reject 400 (เดิม pass ผิด)                                  | -          | - [ ] ผ่าน |
| **Phase 5 Mobile**: Resolver mobile path returns effectiveMobileMenuKeys                                | union/delta logic ตรงกับ web path                           | -          | - [ ] ผ่าน |

---

## Cross-Session Continuity Tips

ถ้า session ใหม่อ่าน checklist นี้:

1. เช็ค checkbox ที่ติ๊กแล้ว → ทำต่อจากข้อถัดไป
2. ถ้า Phase 1 ยังไม่เสร็จ → focus migrations + models ก่อน
3. ถ้าจะเริ่ม code module ใหม่ → ดู `feature-management.backend.md` หรือ `feature-management.frontend.md` ประกอบ
4. ถ้าเจอ regression ระหว่างทำ Phase 4 → กลับไปดู resolver logic ใน `feature-management.backend.md` § 2.5

## Reference Patterns

- Backend: `happywork-backend/src/modules/v1/employee/request/*` (interface/service/repository structure)
- Backend Routes: `happywork-backend/src/api/v1/employee/request/*` (controller + routes pattern)
- Frontend: `happywork-sale-cms/src/app/dashboard/survey/*` + `src/sections/survey/*` (App Router with separate pages, sections + components + utils + enum)
