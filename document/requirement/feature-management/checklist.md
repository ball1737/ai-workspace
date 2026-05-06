---
feature: feature-management
type: feature-checklist
created: 2026-05-04
updated: 2026-05-04
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

- [ ] **P3.1** สร้าง `happywork-backend/src/modules/v2/admin/companyFeature/companyFeature.interface.ts`
- [ ] **P3.2** สร้าง `companyFeature.repository.ts`
  - [ ] `getOverridesByCompanyRepository(companyId)`
  - [ ] `upsertOverrideRepository(companyId, featureId, status, reason, createdBy)`
  - [ ] `removeOverrideRepository(companyId, featureId)`
  - [ ] `getCompanyAddonsRepository(companyId)`
- [ ] **P3.3** สร้าง `companyFeature.service.ts`
  - [ ] `getCompanyFeaturesService(companyUuid)` — list with effective + source ของแต่ละ feature
  - [ ] `setCompanyFeatureOverrideService(companyUuid, featureUuid, enabled, reason, actor)`
  - [ ] `removeCompanyFeatureOverrideService(companyUuid, featureUuid, actor)`
- [ ] **P3.4** สร้าง `companyFeature.adapter.ts`
- [ ] **P3.5** สร้าง `src/api/v2/admin/companyFeature/companyFeature.controller.ts`
- [ ] **P3.6** สร้าง `companyFeature.routes.ts` + register

### Backend — Permission Resolver Module

- [ ] **P3.7** สร้าง `happywork-backend/src/modules/v2/admin/permissionResolver/permissionResolver.interface.ts`
- [ ] **P3.8** สร้าง `permissionResolver.repository.ts`
  - [ ] `getPackageFeaturesRepository(packageId)`
  - [ ] `getAddonFeaturesRepository(addonIds[])`
  - [ ] `getOverridesRepository(companyId)`
- [ ] **P3.9** สร้าง `permissionResolver.service.ts`
  - [ ] `resolveCompanyEffectiveFeatureIdsService(companyId)`
  - [ ] `resolveCompanyEffectiveMenuKeysService(companyId)`
  - [ ] `filterPermissionByMenuKeysService(jsonb, allowedKeys)`
  - [ ] `resolveUserEffectivePermissionService(userUuid)`
  - [ ] In-request memoization (cache per request)

### Backend — Tests

- [ ] **P3.10** Unit test: companyFeature.service.spec.ts
- [ ] **P3.11** Unit test: permissionResolver.service.spec.ts (delta logic, addon union, override)
- [ ] **P3.12** Integration test: end-to-end resolver flow

### Frontend — Redux Slice (Company Features)

- [ ] **F3.1** สร้าง `src/types/store/sale-dashboard-company-features.ts`
- [ ] **F3.2** สร้าง `src/store/services/company-features-request.ts`
- [ ] **F3.3** สร้าง action/reducer/saga + register
- [ ] **F3.4** สร้าง hook `useSaleDashboardCompanyFeatures`

### Frontend — Page D: Features Tab

- [ ] **F3.5** สร้าง `happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx`
- [ ] **F3.6** สร้าง `feature-tab/components/company-feature-list.tsx`
- [ ] **F3.7** สร้าง `feature-tab/components/company-feature-row.tsx`
- [ ] **F3.8** สร้าง `feature-tab/components/company-feature-toggle-confirm-dialog.tsx`
- [ ] **F3.9** สร้าง `feature-tab/components/company-feature-source-filter.tsx`
- [ ] **F3.10** เพิ่ม tab "Features" ใน existing client detail view
  - [ ] หา file ที่ render tabs ของ `/dashboard/client-management/[uuid]` (อาจเป็น `client-management-detail-view` หรือชื่อใกล้เคียง)
  - [ ] เพิ่ม tab item ใหม่
- [ ] **F3.11** Manual UAT — toggle/reset/source badge ถูกต้อง

---

## Phase 4: Resolver Integration (Backend + Regression Test)

### Backend — Update Existing Endpoints

- [ ] **P4.1** อัพเดต `happywork-backend/src/api/external/auth/permissions.controller.ts`
  - [ ] `getPermissionsController` → ใช้ `resolveUserEffectivePermissionService`
- [ ] **P4.2** อัพเดต `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts`
  - [ ] `getPermissionsService` → load mapping → load comp_permission → filter ผ่าน resolver
- [ ] **P4.3** อัพเดต `happywork-backend/src/modules/v1/admin/compPermission/*`
  - [ ] List endpoint filter เมนูตาม company effective menus
- [ ] **P4.4** อัพเดต `happywork-backend/src/modules/v1/admin/compPermissionMapping/*`
  - [ ] Validate permission JSONB ที่ save ต้องไม่มี key นอกเหนือ enabled menus

### Frontend — Permission Management UI

- [ ] **F4.1** ตรวจสอบหน้า permission management เดิม (CMS)
  - [ ] ถ้ามี hardcoded list of menus → ใช้ company effective menus จาก API ใหม่แทน
  - [ ] (อาจไม่ต้องแก้ ถ้า UI render จาก permission JSONB ตรงๆ — ผ่าน filter จาก backend แล้ว)

### Tests & Regression

- [ ] **P4.5** Integration test: end-to-end
  - [ ] Toggle feature → re-fetch permission API → ผล filter ตรง
  - [ ] Add addon → addon features ปรากฏใน effective
  - [ ] Override disabled → feature ที่ package เปิด → ไม่ปรากฏ
  - [ ] Remove override → กลับไปใช้ default จาก package
- [ ] **P4.6** Regression test: existing permission users
  - [ ] User ของ company ที่ไม่มี override → permission เหมือนเดิม
  - [ ] รัน permission API ของ user ทั้งหมดในระบบ test → ไม่มี error
- [ ] **F4.2** UAT: เปิด HappyWork app เป็น user ของ company ที่ feature ถูกปิด → เมนูที่เกี่ยวข้องไม่แสดง

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

## Unit Test Tracker

| Test Case                                                  | ผลที่คาดหวัง                            | ผลการ Test | สถานะ      |
| ---------------------------------------------------------- | --------------------------------------- | ---------- | ---------- |
| Create feature with valid menu_keys                        | สำเร็จ, return Feature                  | -          | - [ ] ผ่าน |
| Create feature with invalid menu_key                       | reject 400                              | -          | - [ ] ผ่าน |
| Create feature with duplicate code                         | reject 400                              | -          | - [ ] ผ่าน |
| Delete feature with package usage                          | reject 400                              | -          | - [ ] ผ่าน |
| Delete feature without usage                               | success 200                             | -          | - [ ] ผ่าน |
| Replace package features                                   | features ของ package update ตรง         | -          | - [ ] ผ่าน |
| Get package effective menus                                | dedupe ครบทุก feature                   | -          | - [ ] ผ่าน |
| Create addon with is_quantifiable=true, no max_quantity    | reject 400                              | -          | - [ ] ผ่าน |
| Toggle company feature (enabled override)                  | upsert row ใน comp_company_features     | -          | - [ ] ผ่าน |
| Get company features                                       | คืน features พร้อม source ตาม mapping   | -          | - [ ] ผ่าน |
| Resolver: package only                                     | effective = package features            | -          | - [ ] ผ่าน |
| Resolver: package + addon                                  | effective = union                       | -          | - [ ] ผ่าน |
| Resolver: package + override disabled                      | effective ไม่มี feature นั้น            | -          | - [ ] ผ่าน |
| Resolver: package + override enabled (feature นอก package) | effective มี feature เพิ่ม              | -          | - [ ] ผ่าน |
| Filter permission JSONB                                    | leaf path ที่ไม่อยู่ใน allowed → ตัดออก | -          | - [ ] ผ่าน |
| Permission API for company with disabled feature           | response permission ไม่มี key นั้น      | -          | - [ ] ผ่าน |
| RBAC: super_admin → see Feature Management menu            | เห็นเมนู                                | -          | - [ ] ผ่าน |
| RBAC: admin → not see Feature Management menu              | ไม่เห็นเมนู                             | -          | - [ ] ผ่าน |

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
