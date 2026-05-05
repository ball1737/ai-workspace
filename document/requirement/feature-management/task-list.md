---
feature: feature-management
type: task-list
created: 2026-05-04
updated: 2026-05-04
owner: Lead
---

# Task List — Feature Management

> รายการ task ของ feature นี้ — Lead แตก task และ assign agent
> Agent อัพเดต status ใน `.ai-memory/agent/{role}/current-task.md` ของตัวเอง แล้ว Lead sync มาที่นี่

---

## Task ID Convention

`FEAT-NNN`

## Status Values

- `pending` — รอเริ่ม
- `in-progress` — กำลังทำ
- `blocked` — ติดอยู่
- `review` — เสร็จรอ review
- `done` — เสร็จสมบูรณ์
- `cancelled` — ยกเลิก

---

## Phase 1: Analysis & Design

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-001 | Read code base + identify scope (PermissionDefault, comp_permission, v2 stripe flow, packageMasterData, request/* and survey/* references) | SA | done | summary.md §3 | 2026-05-04 |
| FEAT-002 | Write requirement.md | SA | done | requirement.md | 2026-05-04 |
| FEAT-003 | Write prd.md | SA | done | prd.md | 2026-05-04 |
| FEAT-004 | Write ssd.md | SA | done | ssd.md | 2026-05-04 |
| FEAT-005 | Validate design vs requirement | BA | pending | — | |
| FEAT-006 | Resolve open questions in requirement.md §9 | SA + Sales | pending | requirement.md §9 | |

---

## Phase 2: Implementation — Backend Schema (Migration + Models)

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-101 | Create migration M1 `*-create-comp_features.ts` + seed (FEATURES + ADDONS, menu_keys=[]) | Backend | pending | ssd.md §3.1.1 | |
| FEAT-102 | Create migration M2 `*-create-comp_packages.ts` + seed packageMasterData (preserve UUID!) | Backend | pending | ssd.md §3.1.2 | |
| FEAT-103 | Create migration M3 `*-create-comp_package_features.ts` | Backend | pending | ssd.md §3.1.3 | |
| FEAT-104 | Create migration M4 `*-create-comp_addons.ts` + seed addonsMasterData | Backend | pending | ssd.md §3.1.4 | |
| FEAT-105 | Create migration M5 `*-create-comp_addon_features.ts` | Backend | pending | ssd.md §3.1.5 | |
| FEAT-106 | Create migration M6 `*-create-comp_company_features.ts` | Backend | pending | ssd.md §3.1.6 | |
| FEAT-107 | Create migration M7 `*-create-comp_company_addons.ts` + migrate from `comp_companies.add_ons` | Backend | pending | ssd.md §3.1.7 | |
| FEAT-108 | Pre-validation script for M8 (check selected_package_uuid match packageMasterData) | Backend | pending | ssd.md §10 | |
| FEAT-109 | Create migration M8 `*-alter-comp_companies-package-id-fk.ts` (backfill + recreate FK) | Backend | pending | ssd.md §3.2 | |
| FEAT-110 | Run `npm run db:migrate:status` + `migrate:latest` on dev → verify | Backend | pending | testcase.md §3 | |
| FEAT-111 | Create model `compFeatures.model.ts` with relationMappings | Backend | pending | ssd.md §3.3 | |
| FEAT-112 | Create model `compPackages.model.ts` | Backend | pending | | |
| FEAT-113 | Create model `compPackageFeatures.model.ts` | Backend | pending | | |
| FEAT-114 | Create model `compAddons.model.ts` | Backend | pending | | |
| FEAT-115 | Create model `compAddonFeatures.model.ts` | Backend | pending | | |
| FEAT-116 | Create model `compCompanyFeatures.model.ts` | Backend | pending | | |
| FEAT-117 | Create model `compCompanyAddons.model.ts` | Backend | pending | | |

---

## Phase 2: Implementation — Backend Master CRUD

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-120 | Add helper `getAvailableMenuKeys()` in `compPermission.ts` | Backend | pending | ssd.md §6 | |
| FEAT-121 | Create `requireSuperAdmin` middleware (if not exists) | Backend | pending | ssd.md §9 | |
| FEAT-130 | Module: `feature/feature.interface.ts` (Zod schemas + types) | Backend | pending | ssd.md §3.3 | |
| FEAT-131 | Module: `feature/feature.repository.ts` | Backend | pending | | |
| FEAT-132 | Module: `feature/feature.service.ts` (validate code unique, validate menu_keys) | Backend | pending | | |
| FEAT-133 | Module: `feature/feature.adapter.ts` | Backend | pending | | |
| FEAT-134 | Routes: `feature/feature.controller.ts` + `feature.routes.ts` | Backend | pending | ssd.md §4.1 | |
| FEAT-135 | Register feature routes in v2 admin router | Backend | pending | | |
| FEAT-140 | Module: `packageCrud/packageCrud.interface.ts` | Backend | pending | | |
| FEAT-141 | Module: `packageCrud/packageCrud.repository.ts` (CRUD + replaceFeatures + effectiveMenus) | Backend | pending | | |
| FEAT-142 | Module: `packageCrud/packageCrud.service.ts` | Backend | pending | | |
| FEAT-143 | Module: `packageCrud/packageCrud.adapter.ts` | Backend | pending | | |
| FEAT-144 | Routes: `packageCrud/packageCrud.controller.ts` + routes | Backend | pending | ssd.md §4.2 | |
| FEAT-145 | Register packageCrud routes | Backend | pending | | |
| FEAT-150 | Module: `addon/addon.interface.ts` (validate is_quantifiable→max_quantity) | Backend | pending | | |
| FEAT-151 | Module: `addon/addon.repository.ts` | Backend | pending | | |
| FEAT-152 | Module: `addon/addon.service.ts` | Backend | pending | | |
| FEAT-153 | Module: `addon/addon.adapter.ts` | Backend | pending | | |
| FEAT-154 | Routes: `addon/addon.controller.ts` + routes | Backend | pending | ssd.md §4.3 | |
| FEAT-155 | Register addon routes | Backend | pending | | |

---

## Phase 2: Implementation — Frontend Setup + Master CRUD UI

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-200 | RBAC update in `rbac.ts` (add features, packages, addons, company_features resources) | Frontend | pending | ssd.md §9 | |
| FEAT-201 | paths.ts — add path constants | Frontend | pending | | |
| FEAT-202 | config-navigation.tsx — add Master Data section (super_admin only) | Frontend | pending | | |
| FEAT-203 | i18n: en.json + th.json — add translation keys | Frontend | pending | | |
| FEAT-210 | Reusable: `multilingual-text-field.tsx` | Frontend | pending | ssd.md §5 | |
| FEAT-211 | Reusable: `menu-keys-select.tsx` (tree multi-select) | Frontend | pending | | |
| FEAT-212 | Reusable: `menu-tree-preview.tsx` (read-only tree) | Frontend | pending | | |
| FEAT-213 | Reusable: `feature-source-badge.tsx` | Frontend | pending | | |
| FEAT-220 | Redux slice: feature-management (action+reducer+saga+service+types+hook) | Frontend | pending | | |
| FEAT-221 | Redux slice: package-management | Frontend | pending | | |
| FEAT-222 | Redux slice: addon-management | Frontend | pending | | |
| FEAT-230 | Section: feature-management — enum + form util + components (table, form, delete-dialog, status, usage-info) | Frontend | pending | ssd.md §5 | |
| FEAT-231 | Section: feature-management views (list, create, detail, edit) | Frontend | pending | | |
| FEAT-232 | Pages: app/dashboard/feature-management/{page, create/page, [id]/page, [id]/edit/page} | Frontend | pending | | |
| FEAT-240 | Section: package-management — enum + form util + components | Frontend | pending | | |
| FEAT-241 | Section: package-management views | Frontend | pending | | |
| FEAT-242 | Pages: app/dashboard/package-management/* | Frontend | pending | | |
| FEAT-243 | PackageDetailView with PackageFeaturesSection + edit dialog + EffectiveMenusPreview | Frontend | pending | | |
| FEAT-250 | Section: addon-management — enum + form util + components (with addon-quantifiable-fields) | Frontend | pending | | |
| FEAT-251 | Section: addon-management views | Frontend | pending | | |
| FEAT-252 | Pages: app/dashboard/addon-management/* | Frontend | pending | | |

---

## Phase 3: Per-Company Toggle (Backend + Frontend)

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-301 | Module: `companyFeature/companyFeature.interface.ts` | Backend | pending | ssd.md §4.4 | |
| FEAT-302 | Module: `companyFeature/companyFeature.repository.ts` | Backend | pending | | |
| FEAT-303 | Module: `companyFeature/companyFeature.service.ts` (getCompanyFeatures with source mapping) | Backend | pending | | |
| FEAT-304 | Module: `companyFeature/companyFeature.adapter.ts` | Backend | pending | | |
| FEAT-305 | Routes: companyFeature controller + routes | Backend | pending | | |
| FEAT-306 | Register companyFeature routes | Backend | pending | | |
| FEAT-310 | Module: `permissionResolver/permissionResolver.interface.ts` | Backend | pending | ssd.md §6.2 | |
| FEAT-311 | Module: `permissionResolver/permissionResolver.repository.ts` (batch queries) | Backend | pending | | |
| FEAT-312 | Module: `permissionResolver/permissionResolver.service.ts` (resolveCompanyEffectiveMenuKeysService, filterPermissionByMenuKeysService, resolveUserEffectivePermissionService, in-request memoization) | Backend | pending | | |
| FEAT-320 | Redux slice: company-features | Frontend | pending | | |
| FEAT-321 | Section: client-management/feature-tab + components (list, row, toggle-confirm-dialog, source-filter) | Frontend | pending | ssd.md §5 | |
| FEAT-322 | Integrate Features tab into existing client-management detail page | Frontend | pending | | |

---

## Phase 4: Resolver Integration

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-401 | Modify `permissions.controller.ts` (external/auth) — wrap with resolver | Backend | pending | ssd.md §6.2 | |
| FEAT-402 | Modify `permissions.service.ts` (v1/externalAuth) — call resolveUserEffectivePermissionService | Backend | pending | | |
| FEAT-403 | Modify `compPermission` admin module — list filter | Backend | pending | | |
| FEAT-404 | Modify `compPermissionMapping` admin module — save validation | Backend | pending | | |
| FEAT-405 | (If applicable) Modify CMS permission management page — filter UI | Frontend | pending | | |
| FEAT-406 | Pre-deploy: backfill check (every active company has feature mapping via package) | DevOps + Backend | pending | ssd.md §10 | |
| FEAT-407 | Compare script: effective menus of sample 100 companies = expected | Backend | pending | | |

---

## Phase 4: Quality

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-501 | Code review (backend) | Code Reviewer | pending | — | |
| FEAT-502 | Code review (frontend) | Code Reviewer | pending | — | |
| FEAT-503 | Write unit tests (backend services) | QA + Backend | pending | testcase.md §2 | |
| FEAT-504 | Write integration tests (resolver e2e flow) | QA + Backend | pending | testcase.md §3 | |
| FEAT-505 | Run unit + integration tests | QA | pending | testcase.md §6 | |
| FEAT-506 | Manual / E2E test (CMS golden path + RBAC) | QA | pending | testcase.md §4-5 | |
| FEAT-507 | Regression test: existing permission API | QA | pending | testcase.md §3 | |
| FEAT-508 | Performance test: resolver latency under load | QA + Backend | pending | testcase.md §3 | |
| FEAT-509 | Final BA validation | BA | pending | — | |

---

## Phase 4: Closeout

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-601 | Update summary.md "Final Outcome" | Lead | pending | summary.md §7 | |
| FEAT-602 | Clear .ai-memory/current.md | Lead | pending | — | |
| FEAT-603 | Update .ai-memory/summary.md | Lead | pending | — | |
| FEAT-604 | Deploy plan + rollback rehearsal | DevOps | pending | ssd.md §10 | |

---

## Phase 5: Cleanup (LATER — แยก scope)

> ทำหลังจาก Phase 1-4 deploy stable แล้ว และ confirm ว่า v1 caller ไม่ active

| ID | Task | Owner | Status | Doc Ref | Updated |
|----|------|-------|--------|---------|---------|
| FEAT-701 | Migrate caller of v1 `featureManagement` module → new resolver/companyFeature | Backend | pending | — | |
| FEAT-702 | Drop migration: `*-drop-sale_dashboard_disabled_features.ts` | Backend | pending | — | |
| FEAT-703 | Drop module: `src/modules/v1/featureManagement/` | Backend | pending | — | |
| FEAT-704 | Drop constant: `src/constant/featureDefinitions.ts` | Backend | pending | — | |
| FEAT-705 | Migrate v1 callers of `const_packages` (paymentProcessing, subscription, trialExpiration, customers, leads, quotations, employees/employeeLimit) | Backend | pending | — | |
| FEAT-706 | Drop `const_packages` (after FEAT-705) | Backend | pending | — | |
| FEAT-707 | Drop column `comp_companies.add_ons` (jsonb) | Backend | pending | — | |

---

## Blocked Tasks

_(none)_

| ID | Reason | Waiting on |
|----|--------|-----------|
|    |        |           |
