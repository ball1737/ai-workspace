---
feature: feature-management
type: testcase
created: 2026-05-04
updated: 2026-05-04
owner: QA
status: draft
---

# Test Cases — Feature Management

> รายการ test case ทั้งหมดของ feature — เขียนโดย QA หลัง SA finish SSD
> เริ่มจาก unit test, ในอนาคตจะเพิ่ม e2e + screenshot

---

## 1. Test Strategy

| Test Type   | Coverage                                                         | Tool                       | Owner        |
| ----------- | ---------------------------------------------------------------- | -------------------------- | ------------ |
| Unit        | Backend services + repositories (validation, business rules)     | Jest                       | Backend dev  |
| Unit        | Frontend reusable components (form, validation, RBAC visibility) | Jest + RTL                 | Frontend dev |
| Integration | Backend API e2e (CRUD flow + resolver)                           | Jest + supertest + test DB | QA           |
| E2E         | (future) — full UI flow                                          | Playwright                 | QA           |
| Manual      | CMS golden path + cross-system (CMS → backend → app)             | —                          | QA           |
| Performance | Resolver latency under load                                      | Artillery / k6             | QA + Backend |
| Migration   | Migration up/down + backfill correctness on dev/staging          | Manual + script            | Backend      |

---

## 2. Unit Test Cases

### UT-1: feature.service

- File: `src/modules/v2/admin/feature/feature.service.spec.ts`
- Description: business logic ของ feature CRUD
- Setup: mock `feature.repository`, mock `getAvailableMenuKeys()`
- Cases:
  - [ ] `createFeatureService` with valid input → return Feature, repository called once
  - [ ] `createFeatureService` with duplicate code → throw AppError 'CODE_EXISTS' (400)
  - [ ] `createFeatureService` with menu_keys ที่ไม่อยู่ใน PermissionDefault → throw AppError 'INVALID_MENU_KEY' (400)
  - [ ] `updateFeatureService` partial update — only `name` → repository called with patch
  - [ ] `deleteFeatureService` ที่ไม่มี usage → repository soft-delete called
  - [ ] `deleteFeatureService` ที่มี package usage 2 + addon usage 1 → throw AppError 'FEATURE_IN_USE' พร้อมข้อความ "used by 2 packages and 1 addon"

### UT-2: packageCrud.service

- File: `src/modules/v2/admin/packageCrud/packageCrud.service.spec.ts`
- Cases:
  - [ ] `createPackageService` with duplicate code → reject
  - [ ] `replacePackageFeaturesService` with valid feature uuids → transaction: delete old + insert new
  - [ ] `replacePackageFeaturesService` with invalid feature uuid → reject all (transaction rollback)
  - [ ] `getPackageEffectiveMenusService` → return dedupe union of features' menu_keys
  - [ ] `deletePackageService` with companies referenced → reject

### UT-3: addon.service

- File: `src/modules/v2/admin/addon/addon.service.spec.ts`
- Cases:
  - [ ] `createAddonService` with `is_quantifiable=true, max_quantity=null` → reject 'MAX_QUANTITY_REQUIRED'
  - [ ] `createAddonService` with `is_quantifiable=true, max_quantity=10` → success
  - [ ] `createAddonService` with `is_quantifiable=false, max_quantity=null` → success
  - [ ] `replaceAddonFeaturesService` valid → success
  - [ ] `deleteAddonService` with company purchase → reject

### UT-4: companyFeature.service

- File: `src/modules/v2/admin/companyFeature/companyFeature.service.spec.ts`
- Cases:
  - [ ] `getCompanyFeaturesService(uuid)` company has package only → all features in package source='package', other features source='default-disabled'
  - [ ] company with package + addon → addon features source='addon'
  - [ ] company with override (enabled feature outside package) → source='override-enabled'
  - [ ] company with override (disabled feature in package) → source='override-disabled', enabled=false
  - [ ] `setCompanyFeatureOverrideService(enabled=true, reason="...")` → upsert row in `comp_company_features` with override_status='enabled'
  - [ ] `setCompanyFeatureOverrideService` existing override → update
  - [ ] `removeCompanyFeatureOverrideService` → row deleted, return updated CompanyFeatureItem reflecting default

### UT-5: permissionResolver.service

- File: `src/modules/v2/admin/permissionResolver/permissionResolver.service.spec.ts`
- Cases:
  - [ ] `resolveCompanyEffectiveFeatureIdsService` package only → return package.features
  - [ ] package + addon → return union (no duplicates)
  - [ ] package + override disabled → return package.features minus disabled
  - [ ] package + override enabled (feature not in package) → return package.features plus enabled
  - [ ] package + addon + override disabled (disable feature that addon unlocked) → result excludes that feature
  - [ ] `resolveCompanyEffectiveMenuKeysService` → return Set<string> = union of menu_keys (dedupe)
  - [ ] `filterPermissionByMenuKeysService(jsonb, allowedKeys)` — leaf path "payroll.payrollSetting" not in allowed → drop key from result
  - [ ] empty allowedKeys → return empty object {}
  - [ ] all keys allowed → return original JSONB
  - [ ] In-request memoization → same companyId resolved twice in same request → repository called once

### UT-6: getAvailableMenuKeys helper

- File: `src/constant/compPermission.spec.ts`
- Cases:
  - [ ] return all leaf paths from PermissionDefaultDev + PermissionDefaultProd (deduplicated, sorted)
  - [ ] include "dashboard" (main, no submenu)
  - [ ] include "payroll.payrollSetting" (sub)
  - [ ] not include "payroll" (because it has subs, not a leaf)

### UT-7: requireSuperAdmin middleware

- File: `src/middlewares/requireSuperAdmin.middleware.spec.ts`
- Cases:
  - [ ] `req.user.role = 'super_admin'` → next() called
  - [ ] `req.user.role = 'admin'` → throw AppError 403
  - [ ] `req.user` undefined → throw AppError 403

### UT-8: Frontend reusable components (basic smoke)

- File: `src/components/feature-management/*.spec.tsx`
- Cases:
  - [ ] `<MultilingualTextField>` renders TH+EN inputs
  - [ ] `<MenuKeysSelect>` with availableKeys → renders tree with checkbox
  - [ ] `<FeatureSourceBadge source="package">` renders Chip with correct color/label

---

## 3. Integration Test Cases

### IT-1: Feature CRUD flow (super_admin)

- Description: full e2e CRUD with super_admin token, real test DB
- Pre-condition: test DB migrated, super_admin user exists, get token
- Steps:
  1. POST /api/v2/admin/features → 201
  2. GET /api/v2/admin/features → list contains created
  3. GET /api/v2/admin/features/:uuid → return Feature
  4. PUT /api/v2/admin/features/:uuid → 200
  5. DELETE /api/v2/admin/features/:uuid → 200 (soft-delete, status_type=archived)
- Expected: each step returns expected status; DB state matches

### IT-2: Package + Feature link

- Description: CRUD package + replaceFeatures
- Steps:
  1. Create 3 features
  2. Create 1 package
  3. PUT /packages/:uuid/features with featureUuids=[3 uuids]
  4. GET /packages/:uuid → return PackageWithFeatures (3 features)
  5. GET /packages/:uuid/effective-menus → return dedupe union
  6. PUT /packages/:uuid/features with featureUuids=[2 uuids] → 1 feature removed
  7. DELETE feature → reject (still in package)

### IT-3: Addon + Feature link

- Similar to IT-2

### IT-4: Company Feature override (Phase 3)

- Description: get + toggle + reset
- Pre-condition: company A has package P, P has features [f1, f2]; addon X (with feature f3) purchased by company A
- Steps:
  1. GET /companies/A/features → list ทั้งหมด, f1+f2 source='package', f3 source='addon', อื่นๆ source='default-disabled'
  2. PUT /companies/A/features/f4 with enabled=true, reason='trial' → upsert override
  3. GET /companies/A/features → f4 source='override-enabled', enabled=true
  4. PUT /companies/A/features/f1 with enabled=false → override row override_status=disabled
  5. GET /companies/A/features → f1 source='override-disabled', enabled=false
  6. DELETE /companies/A/features/f1 → ลบ override
  7. GET → f1 source='package', enabled=true (กลับ default)

### IT-5: Resolver end-to-end (Phase 4)

- Description: ทดสอบ resolver ผ่าน existing permission API
- Pre-condition: same as IT-4
- Steps:
  1. GET /api/external/auth/v1/permissions/:userUuid (user ของ company A) → permission JSONB ที่ filter แล้ว (มีแค่ menu ของ f1+f2+f3, ไม่มีอื่น)
  2. Toggle f1 disabled → re-fetch → menu ของ f1 หายไปจาก JSONB
  3. Toggle f4 enabled → re-fetch → menu ของ f4 เพิ่มเข้ามา
  4. Reset f1 override → re-fetch → menu f1 กลับมา

### IT-6: RBAC enforcement

- Steps:
  1. admin token call any /api/v2/admin/features endpoint → 403
  2. manager token → 403
  3. viewer token → 403
  4. super_admin token → 200

### IT-7: Migration M8 dry-run

- Description: รัน M8 บน sample data, ตรวจ backfill
- Pre-condition: test DB seed sample comp_companies (10 rows: 7 มี selected_package_uuid, 3 ไม่มี)
- Steps:
  1. Run M2 (seed comp_packages)
  2. Run M8 step (a)+(b) — dry run (BEGIN; ... ROLLBACK)
  3. ตรวจ: 7 rows มี package_id ตรงกับ comp_packages ที่ uuid match selected_package_uuid; 3 rows มี package_id = Seed
  4. ตรวจ: ไม่มี orphan FK
- Expected: 100% success

---

## 4. E2E Test Cases (Future)

### E2E-1: Super Admin Master Data Flow

- Description: super_admin login → CRUD feature/package/addon → assign features → preview menus
- User actions:
  1. Login as super_admin
  2. Navigate to Master Data > Feature Management
  3. Create 3 features
  4. Navigate to Package Management → create 1 package → assign 3 features
  5. View package detail → verify Effective Menus Preview
  6. Navigate to Addon Management → create 1 addon → assign 1 feature
- Expected: ทุกขั้นตอน save success; preview ตรงตามที่กำหนด
- Screenshot location: `_docs/requirement/feature-management/test-results/{date}/E2E-1.png`

### E2E-2: Operations Override Feature for Company

- Description: operations เปิด feature ให้บริษัท key account
- User actions:
  1. Login as super_admin
  2. Open client detail of company A
  3. Click "Features" tab
  4. Filter "Default Disabled"
  5. Toggle "Advanced Reporting" → enter reason → confirm
  6. Re-open tab → verify source='override-enabled', reason persists
- Expected: state persists across reload

### E2E-3: End User App reflects company features

- Description: end user เห็นเมนูตาม override
- User actions:
  1. Override feature X disabled for company A
  2. Login as user of company A in HappyWork app
  3. Verify menu X is hidden
- Expected: menu hidden

### E2E-4: RBAC for Frontend

- Description: admin ไม่เห็นเมนู Master Data
- User actions:
  1. Login as admin (not super_admin)
  2. Verify left sidebar — no Master Data section
  3. Try direct URL /dashboard/feature-management → 403 view
- Expected: blocked

---

## 5. Manual Test Cases

### MT-1: Migration on dev environment

- Steps:
  1. Backup dev DB
  2. `npm run db:migrate:status` → list pending
  3. `npm run db:migrate:latest` → apply M1-M8
  4. Run post-migration verification queries (ดู `feature-management.migration.md`)
  5. Verify all checks pass
- Expected: all migrations success, no orphan FK, seed data correct
- Tested by: {QA name}
- Date: {YYYY-MM-DD}
- Result: pass / fail / blocked

### MT-2: Migration rollback

- Steps:
  1. After MT-1, run `npm run db:migrate:rollback`
  2. Verify all 8 tables dropped + comp_companies FK reverted
- Expected: clean rollback

### MT-3: CMS Performance under feature load

- Steps:
  1. Seed 100 features, 30 packages, 10 addons
  2. Open Feature Management list page
  3. Measure initial load time
  4. Test pagination
- Expected: page loads < 2s, smooth pagination

### MT-4: Cross-system flow

- Steps:
  1. Setup: create feature, package, assign feature
  2. Set company A's package to that package
  3. Login as user of company A in HappyWork app
  4. Verify menu unlocked by feature visible
  5. Remove feature from package → re-login user → menu disappear
- Expected: cross-system flow works

### MT-5: Concurrent override (race condition)

- Steps:
  1. Two super_admin sessions try toggle same feature for same company simultaneously
- Expected: one succeeds, other gets 409 (or last-write-wins, depending on impl)

---

## 6. Test Execution Log

| Run | Date       | Type        | Pass | Fail | Skip | Note                                                                                          |
| --- | ---------- | ----------- | ---- | ---- | ---- | --------------------------------------------------------------------------------------------- |
| 1   | 2026-05-06 | Integration | 25   | 0    | 0    | Phase 4 Wave 4 — `permission-resolver.integration.spec.ts` (P4.5+P4.6+F4.2 IT-A → IT-L)       |
| 2   | 2026-05-06 | Integration | 17   | 0    | 0    | Sibling regression — `companyFeature.integration.spec.ts` re-run, no cross-spec leakage       |
| 3   | 2026-05-06 | Unit        | 1    | 2    | 0    | Pre-existing stale unit (`tests/unit/externalAuth/permissions.service.test.ts`) — see DEF-001 |

### Run 1 detail — Phase 4 Wave 4 (`permission-resolver.integration.spec.ts`)

| Group                                            | TC count | Pass   | Fail  |
| ------------------------------------------------ | -------- | ------ | ----- |
| W1 `getPermissionsService`                       | 4        | 4      | 0     |
| W3.5 Fix 1 `getEmployeePermissionService`        | 10       | 10     | 0     |
| W2 + W3.5 Fix 2 `getCompPermissionByUuidService` | 4        | 4      | 0     |
| P4.4 save validation                             | 5        | 5      | 0     |
| W3.5 Fix 3 `/comp-permission/default`            | 2        | 2      | 0     |
| **Total**                                        | **25**   | **25** | **0** |

Mapping ของ checklist cases → TC:

| Checklist case                         | TC ใน spec                                                                        |
| -------------------------------------- | --------------------------------------------------------------------------------- |
| IT-A toggle override → re-fetch        | `IT-A: company has override-disabled feature → menu of that feature filtered out` |
| IT-B addon → permission opens new menu | `IT-B: addon adds feature F → menu key surfaces in baseline`                      |
| IT-C override-disabled → menu absent   | `IT-C: override-disabled feature that package enables → menu absent`              |
| IT-D remove override → revert          | `IT-D: remove override → resolver returns full package menus`                     |
| IT-E baseline (no override)            | `IT-E: company without override → permission baseline`                            |
| IT-F multiple users no error           | `IT-F: response shape has all 4 fields … no crash` + W1 error propagation         |
| IT-G owner (Q-A=STRICT)                | `IT-G: OWNER of company that disabled feature X → menu X not visible`             |
| IT-H non-owner parity                  | `IT-H: NON-OWNER of same company → parity`                                        |
| IT-I /default?companyUuid filtered     | `IT-I: when companyUuid provided → resolver called with company.id`               |
| IT-J /default backward compat          | `IT-J: when companyUuid NOT provided → return full baseline`                      |
| IT-K admin role-edit + owner           | `IT-K: owner role + company that disabled feature → menu absent` + parity         |
| IT-L save validation reject            | `IT-L: createCompPermissionService rejects 400` + 4 supplementary cases           |

---

## 7. Defects Found

| ID      | Test Case                                                       | Description                                                                                                                                                                                                                                                                                                                      | Severity | Status | Assigned to         |
| ------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------ | ------------------- |
| DEF-001 | `tests/unit/externalAuth/permissions.service.test.ts` (2 cases) | Pre-existing unit test ยังเช็ค shape เก่า `{ userUuid, isAdmin }` ไม่มี field `permission` ที่ Phase 4 W1 เพิ่มเข้ามา (intentional contract change ตาม `PermissionsResponse`). Production code ทำงานถูก, unit test stale ต้องอัพเดต expected ให้ include `permission: {}`. ไม่ใช่ regression ของ Wave 4 — เป็น oversight ของ W1. | minor    | open   | Backend (W1 author) |

---

## 8. Test Results Folder

หลัง run test ให้เก็บผล + screenshot ที่:

```
_docs/requirement/feature-management/test-results/{YYYY-MM-DD}/
├── summary.md
├── unit-results.json
├── integration-results.json
└── screenshots/
```
