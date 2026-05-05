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

- [ ] **P2.1** อัพเดต `happywork-backend/src/constant/compPermission.ts`
  - [ ] เพิ่ม helper `getAvailableMenuKeys(): string[]` — recursive walk ของ PermissionDefault, return leaf paths

### Backend — Middleware

- [ ] **P2.2** สร้าง `happywork-backend/src/middlewares/requireSuperAdmin.middleware.ts` (ถ้ายังไม่มี)
  - [ ] เช็ค `req.user.role === 'super_admin'` → ผ่าน, ไม่ใช่ → throw 403

### Backend — Feature Module

- [ ] **P2.3** สร้าง `happywork-backend/src/modules/v2/admin/feature/feature.interface.ts`
  - [ ] Zod schemas: `featureBaseSchema`, `createFeatureSchema`, `updateFeatureSchema`, `listFeaturesSchema`
  - [ ] TS types: `Feature`, `FeatureWithUsage`
- [ ] **P2.4** สร้าง `feature.repository.ts`
  - [ ] `getFeatureByIdRepository`, `getFeatureByUuidRepository`, `getFeatureByCodeRepository`
  - [ ] `listFeaturesRepository(filter, pagination)`
  - [ ] `createFeatureRepository`, `updateFeatureRepository`, `softDeleteFeatureRepository`
  - [ ] `countFeatureUsageRepository(featureId)` (count packages + addons)
- [ ] **P2.5** สร้าง `feature.service.ts`
  - [ ] `getFeatureListService`, `getFeatureByUuidService`
  - [ ] `createFeatureService` (validate code unique, validate menu_keys exist)
  - [ ] `updateFeatureService`, `deleteFeatureService` (block ถ้ามี usage)
  - [ ] `getAvailableMenuKeysService`
- [ ] **P2.6** สร้าง `feature.adapter.ts` — convert/normalize for response
- [ ] **P2.7** สร้าง `happywork-backend/src/api/v2/admin/feature/feature.controller.ts`
  - [ ] `getFeatureListController`, `getFeatureByUuidController`
  - [ ] `createFeatureController`, `updateFeatureController`, `deleteFeatureController`
  - [ ] `getAvailableMenuKeysController`
- [ ] **P2.8** สร้าง `feature.routes.ts` — Express router with middleware chain
- [ ] **P2.9** Register route ใน `src/api/v2/admin/index.routes.ts`

### Backend — Package CRUD Module

- [ ] **P2.10** สร้าง `happywork-backend/src/modules/v2/admin/packageCrud/packageCrud.interface.ts`
- [ ] **P2.11** สร้าง `packageCrud.repository.ts`
  - [ ] CRUD บน comp_packages
  - [ ] `getPackageFeaturesRepository(packageId)`, `replacePackageFeaturesRepository(packageId, featureIds[])` (transaction)
  - [ ] `countPackageUsageRepository` (count comp_companies)
- [ ] **P2.12** สร้าง `packageCrud.service.ts`
  - [ ] CRUD services
  - [ ] `replacePackageFeaturesService`
  - [ ] `getPackageEffectiveMenusService` (รวม menu_keys ของ features, dedupe)
- [ ] **P2.13** สร้าง `packageCrud.adapter.ts`
- [ ] **P2.14** สร้าง `src/api/v2/admin/packageCrud/packageCrud.controller.ts`
  - [ ] CRUD controllers
  - [ ] `replacePackageFeaturesController`, `getPackageEffectiveMenusController`
- [ ] **P2.15** สร้าง `packageCrud.routes.ts` + register

### Backend — Addon Module

- [ ] **P2.16** สร้าง `happywork-backend/src/modules/v2/admin/addon/addon.interface.ts`
  - [ ] Validation: is_quantifiable=true → max_quantity required
- [ ] **P2.17** สร้าง `addon.repository.ts`
  - [ ] CRUD + `getAddonFeaturesRepository`, `replaceAddonFeaturesRepository`
- [ ] **P2.18** สร้าง `addon.service.ts`
  - [ ] CRUD + replaceAddonFeaturesService + delete validation (block ถ้ามี company purchase)
- [ ] **P2.19** สร้าง `addon.adapter.ts`
- [ ] **P2.20** สร้าง `src/api/v2/admin/addon/addon.controller.ts` + `addon.routes.ts`
- [ ] **P2.21** Register route

### Backend — Tests

- [ ] **P2.22** Unit tests: `feature.service.spec.ts`
- [ ] **P2.23** Unit tests: `packageCrud.service.spec.ts`
- [ ] **P2.24** Unit tests: `addon.service.spec.ts`
- [ ] **P2.25** Integration test: ใช้ super_admin token CRUD ครบ flow

### Frontend — Setup

- [ ] **F2.1** อัพเดต `happywork-sale-cms/src/utils/rbac.ts`
  - [ ] เพิ่ม Resource: `features`, `packages`, `addons`, `company_features`
  - [ ] อัพเดต PERMISSIONS matrix (super_admin only)
- [ ] **F2.2** อัพเดต `happywork-sale-cms/src/routes/paths.ts`
  - [ ] เพิ่ม `paths.dashboard.featureManagement.{root,create,detail,edit}`
  - [ ] เพิ่ม `paths.dashboard.packageManagement.*`
  - [ ] เพิ่ม `paths.dashboard.addonManagement.*`
- [ ] **F2.3** อัพเดต `happywork-sale-cms/src/layouts/dashboard/config-navigation.tsx`
  - [ ] เพิ่ม section "Master Data" (super_admin conditional) มี 3 menu items
- [ ] **F2.4** อัพเดต `happywork-sale-cms/src/locales/langs/{en,th}.json`
  - [ ] section `featureManagement`, `packageManagement`, `addonManagement`, `companyFeatures`, `menu.master_data`

### Frontend — Reusable Components

- [ ] **F2.5** สร้าง `happywork-sale-cms/src/components/feature-management/multilingual-text-field.tsx`
- [ ] **F2.6** สร้าง `menu-keys-select.tsx` — multi-select tree (group by main category)
- [ ] **F2.7** สร้าง `menu-tree-preview.tsx` — read-only tree preview
- [ ] **F2.8** สร้าง `feature-source-badge.tsx`

### Frontend — Redux Slice (Feature Management)

- [ ] **F2.9** สร้าง `happywork-sale-cms/src/types/store/sale-dashboard-feature-management.ts`
- [ ] **F2.10** สร้าง `src/store/services/feature-management-request.ts`
- [ ] **F2.11** สร้าง `src/store/actions/sale-dashboard-feature-management.ts`
- [ ] **F2.12** สร้าง `src/store/reducers/sale-dashboard-feature-management.ts` + register ใน combineReducers
- [ ] **F2.13** สร้าง `src/store/sagas/sale-dashboard-feature-management.ts` + register ใน rootSaga
- [ ] **F2.14** สร้าง hook `useSaleDashboardFeatureManagement` (อยู่ใน reducer หรือแยกไฟล์)

### Frontend — Redux Slice (Package Management)

- [ ] **F2.15** Mirror P2.9–P2.14 สำหรับ package-management

### Frontend — Redux Slice (Addon Management)

- [ ] **F2.16** Mirror P2.9–P2.14 สำหรับ addon-management

### Frontend — Page A: Feature Management

- [ ] **F2.17** สร้าง `happywork-sale-cms/src/sections/feature-management/feature.enum.ts`
- [ ] **F2.18** สร้าง `src/sections/feature-management/utils/feature-form.ts` (Yup schema)
- [ ] **F2.19** สร้าง `src/sections/feature-management/components/feature-table.tsx`
- [ ] **F2.20** สร้าง `src/sections/feature-management/components/feature-form.tsx` (shared create+edit)
- [ ] **F2.21** สร้าง `feature-delete-dialog.tsx`
- [ ] **F2.22** สร้าง `feature-status-option.tsx`
- [ ] **F2.23** สร้าง `feature-usage-info.tsx`
- [ ] **F2.24** สร้าง `feature-list-view.tsx`
- [ ] **F2.25** สร้าง `feature-create-view.tsx`
- [ ] **F2.26** สร้าง `feature-detail-view.tsx`
- [ ] **F2.27** สร้าง `feature-edit-view.tsx`
- [ ] **F2.28** สร้าง pages:
  - [ ] `src/app/dashboard/feature-management/page.tsx`
  - [ ] `src/app/dashboard/feature-management/create/page.tsx`
  - [ ] `src/app/dashboard/feature-management/[id]/page.tsx`
  - [ ] `src/app/dashboard/feature-management/[id]/edit/page.tsx`

### Frontend — Page B: Package Management

- [ ] **F2.29** สร้าง section files (Mirror Page A pattern)
  - [ ] `package.enum.ts`, `utils/package-form.ts`
  - [ ] components: `package-table.tsx`, `package-form.tsx`, `package-delete-dialog.tsx`
  - [ ] `package-features-section.tsx` (อยู่ใน detail view)
  - [ ] `package-features-edit-dialog.tsx`
  - [ ] `package-effective-menus-preview.tsx`
- [ ] **F2.30** สร้าง views: `package-list-view.tsx`, `package-create-view.tsx`, `package-detail-view.tsx`, `package-edit-view.tsx`
- [ ] **F2.31** สร้าง pages: list, create, [id], [id]/edit

### Frontend — Page C: Addon Management

- [ ] **F2.32** สร้าง section files
  - [ ] `addon.enum.ts`, `utils/addon-form.ts`
  - [ ] components: `addon-table.tsx`, `addon-form.tsx`, `addon-delete-dialog.tsx`
  - [ ] `addon-quantifiable-fields.tsx` (conditional max_quantity)
  - [ ] `addon-features-section.tsx`, `addon-features-edit-dialog.tsx`
- [ ] **F2.33** สร้าง views: `addon-list-view.tsx`, `addon-create-view.tsx`, `addon-detail-view.tsx`, `addon-edit-view.tsx`
- [ ] **F2.34** สร้าง pages

### Frontend — Test

- [ ] **F2.35** Manual UAT — golden path Page A/B/C

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
