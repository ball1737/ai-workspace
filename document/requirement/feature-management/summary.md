---
feature: feature-management
type: feature-summary
created: 2026-05-04
updated: 2026-05-04
owner: SA (initial), Lead (ongoing updates)
status: draft
---

# Feature Summary — Feature Management

> ระบบจัดการ feature ของ package และลูกค้า (per-company) — DB-driven, override per company, filter `comp_permission` ที่ runtime

---

## 1. Current State (สถานะปัจจุบัน)

### โครงสร้างที่มีอยู่

- HappyWork app แสดงเมนูผ่าน `comp_permission.permission` (JSONB) ที่ map ไปยัง user ผ่าน `comp_permission_mapping`
- โครงสร้าง permission ทั้งระบบมาจาก constant `PermissionDefault` ใน `happywork-backend/src/constant/compPermission.ts`
- มี 3 ชั้น: main category (14 รายการ) → subcategory (24 รายการ) → action fields (read/create/update/delete/export)
- มี role preset 4 แบบ: Owner / Admin / Approver / Employee
- รองรับ multilingual (JSONB pattern สำหรับ `name`, `description`)
- Stripe flow v2 (production) ใช้ constant `packageMasterData`/`addonsMasterData` (ใน `happywork-backend/src/modules/v2/admin/package/package.interface.ts`) — ไม่ได้อ่านจาก `const_packages` table แล้ว
- Subscription state ใหม่อยู่ใน `comp_companies` (selected_package_uuid, package_tier, add_ons jsonb) + `comp_subscription_historys`

### ไฟล์ที่เกี่ยวข้อง

- `happywork-backend/src/constant/compPermission.ts` — PermissionDefault structure
- `happywork-backend/src/database/postgresql/migrations/data/20250919-1300-add-comp_permission.ts`
- `happywork-backend/src/database/postgresql/migrations/data/20250922-1300-add-comp_permission_mapping.ts`
- `happywork-backend/src/api/external/auth/permissions.controller.ts`
- `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts`
- `happywork-backend/src/modules/v1/featureManagement/*` (legacy, by-exception pattern)
- `happywork-backend/src/modules/v2/admin/package/package.interface.ts` (packageMasterData)
- `happywork-backend/src/database/postgresql/models/data/compCompanies.model.ts`

### Limitations / Pain points

- ทุก `comp_company` เห็นเมนูชุดเดียวกัน (เพราะ base คือ constant ที่ฮาร์ดโค้ด)
- ไม่มี mechanism ผูก feature → package → company ในระดับ database
- เปิด/ปิด feature เฉพาะบางบริษัทไม่ได้
- Legacy `featureManagement` v1 module ใช้ `sale_dashboard_disabled_features` table (by-exception) แต่ feature/addon ฮาร์ดโค้ด → ไม่ยืดหยุ่น
- `const_packages` table มี 12 records (legacy ที่ไม่ใช้แล้วใน v2 stripe flow)

---

## 2. Planned Changes (สิ่งที่จะทำ)

### เปลี่ยน

- `comp_companies.package_id` FK: `const_packages.id` → `comp_packages.id` (ใหม่) [keep column name]
- ทุก endpoint ที่ return `comp_permission.permission` → filter ผ่าน resolver ก่อนตอบ

### เพิ่ม

- 7 tables ใหม่: `comp_features`, `comp_packages`, `comp_package_features`, `comp_addons`, `comp_addon_features`, `comp_company_features`, `comp_company_addons`
- 4 modules backend ใหม่ใต้ `v2/admin/`: `feature/`, `packageCrud/`, `addon/`, `companyFeature/`
- 1 shared resolver module: `permissionResolver/`
- 3 หน้า CRUD บน CMS (`/dashboard/feature-management`, `/dashboard/package-management`, `/dashboard/addon-management`) + tab "Features" ใน client detail
- Helper `getAvailableMenuKeys()` ใน `compPermission.ts` (return leaf paths)

### ลบ

- ไม่ลบในงานนี้ (Phase 6 = แยก phase ทีหลัง — เลื่อนจาก Phase 5 เดิม 2026-05-06):
  - `featureManagement` v1 module
  - `sale_dashboard_disabled_features` table
  - `featureDefinitions.ts` constant
  - `const_packages` table (eventual)

### เพิ่ม (Phase 5 — Mobile Menu Parity, added 2026-05-06)

> User แจ้งว่า "ลืม" mobile menu — `PermissionMobileDefault` ที่อยู่ในไฟล์ `compPermission.ts` เป็น dead constant (ไม่มีโค้ด consume)

- Column ใหม่: `comp_features.mobile_menu_keys` JSONB NOT NULL DEFAULT `'[]'` + GIN index (parallel กับ `menu_keys`)
- Helper ใหม่: `getAvailableMobileMenuKeys()` ใน `compPermission.ts` (อ่าน leaf paths จาก `PermissionMobileDefault`)
- Resolver functions: `resolveCompanyEffectiveMobileMenuKeysService`, `filterPermissionMobileByMenuKeysService`, `extractMobileMenuKeysFromPermissionJsonbService`
- Response shape change (BREAKING): `GET /features/menu-keys/available` → `{ web: string[], mobile: string[] }`
- External Auth: เพิ่ม `permissionMobile` field ใน response ของ `/external/auth/v1/permissions/:userUuid` (graceful empty fallback)
- compPermission admin: split validation (BUG FIX Phase 4 W2 oversight) + error code ใหม่ `INVALID_MOBILE_MENU_KEY_FOR_COMPANY`
- Frontend UI: Web col + Mobile col side-by-side ทุกที่ที่แก้/แสดง menu (Page A edit/detail, Page D Company Features tab)
- Migrations: M9 (ALTER ADD COLUMN) + M10 (data backfill SEED tier 4 features)
- Plan reference: `/Users/ball/.claude/plans/lead-menu-elegant-finch.md`

### ผลกระทบที่คาดว่าจะเกิด

- ✅ super_admin จัดการ feature/package/addon ผ่าน CMS ได้
- ✅ override ของบริษัทรายตัวได้
- ✅ user ใน HappyWork app เห็นเมนูตามสิทธิ์ที่บริษัทตัวเองได้รับ
- ⚠️ Schema migration ต้อง backfill `comp_companies.package_id` ก่อน drop FK เก่า (high risk)
- ⚠️ ทุก API ที่อ่าน comp_permission ต้อง regression test

---

## 3. ⚠️ Files Reviewed (ไฟล์ที่ SA อ่านครบแล้ว)

### Folders covered

- `happywork-backend/src/constant/`
- `happywork-backend/src/database/postgresql/migrations/data/` (subset เกี่ยวกับ permission/package/feature)
- `happywork-backend/src/database/postgresql/models/data/` (subset)
- `happywork-backend/src/modules/v1/admin/expenseTypes/` (เคย — ภายหลังเปลี่ยน reference เป็น `v1/employee/request/`)
- `happywork-backend/src/modules/v1/employee/request/`
- `happywork-backend/src/api/v1/employee/request/`
- `happywork-backend/src/modules/v1/externalAuth/`
- `happywork-backend/src/modules/v1/featureManagement/`
- `happywork-backend/src/modules/v1/admin/package/`
- `happywork-backend/src/modules/v2/admin/package/`
- `happywork-backend/src/modules/v2/admin/subscription/`
- `happywork-backend/src/modules/v2/admin/payment/`
- `happywork-backend/_docs/requirement/subscription/`
- `happywork-sale-cms/src/app/dashboard/survey/`
- `happywork-sale-cms/src/sections/survey/`
- `happywork-sale-cms/src/components/`
- `happywork-sale-cms/src/store/`
- `happywork-sale-cms/src/utils/rbac.ts`
- `happywork-sale-cms/src/layouts/dashboard/config-navigation.tsx`

### Files read

| Path                                                                                                                       | Read date  | Note                                                      |
| -------------------------------------------------------------------------------------------------------------------------- | ---------- | --------------------------------------------------------- |
| `happywork-backend/src/constant/compPermission.ts`                                                                         | 2026-05-04 | PermissionDefault structure (14 main, 24 sub)             |
| `happywork-backend/src/constant/package.ts`                                                                                | 2026-05-04 | PackageEnum + PackageAddonsEnum                           |
| `happywork-backend/src/constant/featureDefinitions.ts`                                                                     | 2026-05-04 | FEATURES + ADDONS (legacy)                                |
| `happywork-backend/src/database/postgresql/migrations/data/20250919-1300-add-comp_permission.ts`                           | 2026-05-04 | Schema baseline                                           |
| `happywork-backend/src/database/postgresql/migrations/data/20250922-1300-add-comp_permission_mapping.ts`                   | 2026-05-04 |                                                           |
| `happywork-backend/src/database/postgresql/migrations/data/20251008-0006-recreate-packages-table.ts`                       | 2026-05-04 | const_packages 12 records                                 |
| `happywork-backend/src/database/postgresql/migrations/data/20260107-0002-create-comp-subscription-historys-table.ts`       | 2026-05-04 | New v2 stripe history                                     |
| `happywork-backend/src/database/postgresql/migrations/data/20260204-0704-create-sale-dashboard-disabled-features-table.ts` | 2026-05-04 | Legacy by-exception                                       |
| `happywork-backend/src/database/postgresql/migrations/tableTemplate.ts`                                                    | 2026-05-04 | Reference for new migrations                              |
| `happywork-backend/src/database/postgresql/models/data/compCompanies.model.ts`                                             | 2026-05-04 | selected_package_uuid + add_ons fields                    |
| `happywork-backend/src/modules/v1/employee/request/request.service.ts`                                                     | 2026-05-04 | Backend reference pattern                                 |
| `happywork-backend/src/modules/v1/employee/request/request.repository.ts`                                                  | 2026-05-04 |                                                           |
| `happywork-backend/src/modules/v1/employee/request/request.interface.ts`                                                   | 2026-05-04 |                                                           |
| `happywork-backend/src/api/v1/employee/request/request.routes.ts`                                                          | 2026-05-04 |                                                           |
| `happywork-backend/src/api/v1/employee/request/request.controller.ts`                                                      | 2026-05-04 |                                                           |
| `happywork-backend/src/modules/v1/externalAuth/permissions.service.ts`                                                     | 2026-05-04 | Caller ที่ต้องแก้ใน Phase 4                               |
| `happywork-backend/src/api/external/auth/permissions.controller.ts`                                                        | 2026-05-04 |                                                           |
| `happywork-backend/src/modules/v1/featureManagement/featureManagement.service.ts`                                          | 2026-05-04 | Legacy module                                             |
| `happywork-backend/src/modules/v1/admin/package/package.service.ts`                                                        | 2026-05-04 |                                                           |
| `happywork-backend/src/modules/v2/admin/package/package.service.ts`                                                        | 2026-05-04 | Stripe sync (อย่าแก้)                                     |
| `happywork-backend/src/modules/v2/admin/package/package.interface.ts`                                                      | 2026-05-04 | packageMasterData (7 records: Seed + LITE/CORE/PRO mo+yr) + addonsMasterData (9 records: HappyBeacon + E_SLIP/EVALUATION/HAPPY_PM/E_SIGNATURE mo+yr)          |
| `happywork-backend/_docs/requirement/subscription/subscription-package-summary.md`                                         | 2026-05-04 | Schema reference                                          |
| `happywork-sale-cms/src/app/dashboard/survey/page.tsx`                                                                     | 2026-05-04 | Frontend page wrapper pattern                             |
| `happywork-sale-cms/src/app/dashboard/survey/[id]/page.tsx`                                                                | 2026-05-04 |                                                           |
| `happywork-sale-cms/src/app/dashboard/survey/create/page.tsx`                                                              | 2026-05-04 |                                                           |
| `happywork-sale-cms/src/app/dashboard/survey/[id]/edit/page.tsx`                                                           | 2026-05-04 |                                                           |
| `happywork-sale-cms/src/sections/survey/*`                                                                                 | 2026-05-04 | List/Create/Detail/Edit views + components + utils + enum |
| `happywork-sale-cms/src/utils/rbac.ts`                                                                                     | 2026-05-04 | RBAC matrix                                               |
| `happywork-sale-cms/src/layouts/dashboard/config-navigation.tsx`                                                           | 2026-05-04 | Menu config                                               |
| `happywork-sale-cms/PERMISSION_GUIDE.md`                                                                                   | 2026-05-04 | Existing permission system docs                           |
| `happywork-sale-cms/PERMISSION_IMPLEMENTATION.md`                                                                          | 2026-05-04 |                                                           |

### Files explicitly excluded

| Path                                                          | Reason for exclusion                         |
| ------------------------------------------------------------- | -------------------------------------------- |
| `happywork-backend/node_modules/`                             | Dependency                                   |
| `happywork-backend/dist/`                                     | Build output                                 |
| `happywork-backend/coverage/`                                 | Test report                                  |
| `happywork-sale-cms/node_modules/`, `next-env.d.ts`, `.next/` | Build/dependencies                           |
| `happywork-backend/tests/`                                    | Existing tests (อ่านเฉพาะ pattern reference) |
| Migration files ที่ไม่เกี่ยวข้อง (เช่น `emp_attendance_*`)    | Out of scope                                 |

---

## 4. Architecture Decisions

| Decision                                          | Alternatives considered                                | Reason chosen                                                                                                                                 |
| ------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Feature ↔ menu = mixed leaf paths (D)**         | Main category only, sub category only, arbitrary group | บาง main menu ไม่มี sub menu (dashboard, aiChat, newsFeed) — mixed leaf รองรับทุกกรณี                                                         |
| **Filter `comp_permission` ที่ runtime (C)**      | Strip ใน JSONB ตอน save, snapshot at update            | ไม่ทำลาย JSONB เดิม, rollback ง่าย, audit trail สมบูรณ์                                                                                       |
| **Delta/Override per-company (B)**                | Snapshot ทั้ง feature list, hybrid                     | ถ้า package เพิ่ม feature → company ใช้ package เดียวกันได้ feature ใหม่อัตโนมัติ; data lean                                                  |
| **Greenfield + Deprecate (A)**                    | Extend in-place, hybrid adapter                        | Schema เดิม (`featureManagement` + `sale_dashboard_disabled_features`) ไม่มี slot ใส่ menu_keys + addon mapping                               |
| **สร้าง `comp_packages` ใหม่ + migrate FK (B1)**  | ใช้ `const_packages` ต่อ + alter, two-table model      | v2 Stripe flow ไม่ใช้ const_packages แล้ว; user เลือก B เพราะ flow ใหม่ลด risk                                                                |
| **Addon table แยก (A1)**                          | flag `kind` ใน feature table                           | Addon มี price/billing/Stripe ของตัวเอง; lifecycle ต่างกัน                                                                                    |
| **Action gating = visibility only**               | Action-level gating (read/create/update/delete)        | Requirement ระบุชัดว่า "เมนูที่จะเห็น"; actions ยังให้ comp_permission ตัดสิน                                                                 |
| **API namespace = v2**                            | v1, mixed                                              | Align กับ Stripe flow v2 (ที่ผ่าน production)                                                                                                 |
| **Keep `const_packages` alive (C)**               | Full migrate v1 callers, drop legacy                   | Risk ต่ำสุด; v1 caller (paymentProcessing/subscription/trialExpiration/customers/leads/quotations) ยัง active บางตัว — ค่อย deprecate Phase 5 |
| **CMS menu = แยกเมนูใหม่ + tab ใน client detail** | ใต้ Settings, ใน Sidebar Master Data                   | ตามมาตรฐาน super_admin scoped, ใช้ pattern ที่มีอยู่                                                                                          |
| **Cleanup v1 = แยก phase สุดท้าย**                | รวมใน scope งานหลัก                                    | ลด scope, deploy ทีละชั้น                                                                                                                     |

---

## 5. Risks & Mitigations

| Risk                                                                                    | Likelihood | Impact | Mitigation                                                                                         |
| --------------------------------------------------------------------------------------- | ---------- | ------ | -------------------------------------------------------------------------------------------------- |
| Migration M8 ทำให้ `comp_companies.package_id` มี orphan FK                             | Mid        | High   | Pre-validate `selected_package_uuid` ก่อน, fallback Seed default                                   |
| Resolver filter ทำให้ user เห็นเมนูน้อยกว่าเดิม (regression)                            | High       | High   | Phase 4 มี integration test เต็ม + initial state ของ company ทั้งหมดต้องมี features ครบจาก package |
| Performance N+1 ตอน resolve permission ของ user หลายคนพร้อมกัน                          | Mid        | Mid    | In-request memoization + index บน `(company_id)` ของ override table                                |
| Seed `menu_keys = []` ทำให้ feature ไม่ปลด menu จนกว่า admin จะมาแก้                    | High       | Low    | Phase 1 หลัง migrate ต้องมี admin task เซต menu_keys ก่อน Phase 4 deploy                           |
| Drop FK `const_packages` แล้ว v1 caller ที่ join const_packages พัง                     | Low        | High   | ตัวเลือก C ที่เลือก = keep const_packages alive ในงานนี้                                           |
| Addon migration จาก `comp_companies.add_ons` (jsonb) → `comp_company_addons` ผิด format | Mid        | Mid    | M7 ทำ pre-validation query + manual sample check                                                   |
| Frontend list page render ช้าตอน feature เยอะ                                           | Low        | Low    | Pagination + virtualization                                                                        |
| RBAC ผิดพลาด ทำให้ admin/manager เห็นเมนู                                               | Low        | High   | Test ทุก role + super_admin only matrix                                                            |

---

## 6. Progress Log

| Date       | Event                                                                       | Agent | Note                                   |
| ---------- | --------------------------------------------------------------------------- | ----- | -------------------------------------- |
| 2026-05-04 | Initial analysis + grill session + plan approval                            | SA    | 7 sub-decisions resolved               |
| 2026-05-04 | Documents drafted (requirement, prd, ssd, summary, task-list, testcase, cr) | SA    | Set A — decision & measurement docs    |
| 2026-05-04 | Implementation docs added (backend, frontend, migration, checklist, prompt) | SA    | Set B — engineering playbook merged in |
| 2026-05-06 | Phase 5 plan approved — Mobile Menu Parity (8 Q resolved + 11 waves)        | Lead  | Cleanup เลื่อน Phase 6 — see plan file `/Users/ball/.claude/plans/lead-menu-elegant-finch.md` |

---

## 7. Final Outcome (กรอกเมื่อจบ feature)

- Delivered: _(pending)_
- Deviations from plan: _(pending)_
- Lessons learned: _(pending)_

---

## 8. Reference Documents (ทุกไฟล์ใน folder นี้)

### Decision & Measurement (Set A)

- [requirement.md](./requirement.md) — Functional + non-functional + business rules
- [prd.md](./prd.md) — User stories + success metrics + UX flow
- [ssd.md](./ssd.md) — Solution architecture + API spec + DB schema
- [task-list.md](./task-list.md) — High-level phased task breakdown
- [testcase.md](./testcase.md) — Unit / integration / e2e test cases
- [cr.md](./cr.md) — Change request log

### Engineering Playbook (Set B)

- [backend.md](./backend.md) — Routes, modules, models, middleware, tests
- [frontend.md](./frontend.md) — Pages, components, store, RBAC, i18n
- [migration.md](./migration.md) — Migration sequence + rollback + verification (Knex SQL)
- [checklist.md](./checklist.md) — Path-linked checkbox tracker (cross-session continuity)
- [prompt.md](./prompt.md) — Raw prompt + grill clarifications
