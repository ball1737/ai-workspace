---
feature: feature-management
type: requirement
created: 2026-05-04
updated: 2026-05-04
owner: SA
status: draft
---

# Requirement — Feature Management

> เอกสาร requirement หลักของ feature นี้ — เขียนหลังอ่าน code base ที่เกี่ยวข้องครบและ grill กับ stakeholder ครบแล้ว

---

## 1. Background (ที่มา)

ระบบ HappyWork ปัจจุบันแสดงเมนูใน app ผ่าน `comp_permission.permission` (JSONB) โดย base structure ของเมนูทั้งระบบมาจาก **constant** `PermissionDefault` ใน `happywork-backend/src/constant/compPermission.ts`

ผลที่เกิดขึ้น:
- ทุกบริษัท (`comp_company`) เห็นเมนูชุดเดียวกัน
- ไม่สามารถเปิด/ปิดเมนูเฉพาะบางบริษัทตาม package ที่ลูกค้าซื้อได้
- ไม่มี mechanism ผูก feature → package → company ในระดับ database
- มี `featureManagement` v1 module + `sale_dashboard_disabled_features` table อยู่แล้วในรูปแบบ by-exception แต่ feature/addon ฮาร์ดโค้ดอยู่ใน constant `featureDefinitions.ts` → ไม่ยืดหยุ่น
- v2 Stripe flow ใหม่ใช้ constant `packageMasterData`/`addonsMasterData` (ไม่ผ่าน DB) → master data ของ package/addon ยัง decentralize

ผลกระทบทาง business:
- Sales ขายแพ็กเกจที่มี feature ต่างกันให้ลูกค้าไม่ได้
- ไม่มีระบบ addon ที่ปลด feature เพิ่ม
- ไม่มี audit trail ของการเปลี่ยน feature เปิด/ปิดของลูกค้า

## 2. Goals (เป้าหมาย)

### เป้าหมายหลัก
- จัดการ feature/package/addon ผ่าน CMS ได้ (CRUD)
- ลูกค้าแต่ละราย (`comp_company`) มี feature set ที่ต่างกันได้ตาม package + addons + override
- เมนูใน HappyWork app ของ user แสดงตาม feature ที่บริษัทตัวเองได้รับ
- ทุก endpoint ที่อ่าน comp_permission filter ผ่าน effective features ก่อนตอบ

### เป้าหมายรอง
- รักษา backward compatibility กับ comp_permission JSONB เดิม (ไม่ทำลาย data)
- รองรับ multilingual (TH/EN) ตามมาตรฐาน JSONB pattern
- โครงสร้างพร้อมรองรับ Phase 5 (cleanup v1 legacy) ในอนาคต

## 3. Non-Goals (สิ่งที่จะไม่ทำในรอบนี้)

- ไม่ migrate v1 caller ทั้งหมดที่ใช้ `const_packages` (ยกเว้น `comp_companies.package_id` FK) — Phase 5
- ไม่ drop `const_packages` table ในงานนี้
- ไม่ drop / migrate `featureDefinitions.ts` constant ในงานนี้
- ไม่ drop `featureManagement` v1 module + `sale_dashboard_disabled_features` table ในงานนี้
- ไม่แตะ Stripe sync flow (`v2/admin/package/package.service.ts`)
- ไม่ทำ versioning / change history ของ feature/package configuration
- ไม่ทำ feature dependencies (เช่น "feature A ต้องเปิดก่อน feature B")
- ไม่ทำ time-based expiration ของ company override
- ไม่ทำ action-level gating (read/create/update/delete) — feature คุมแค่ visibility

## 4. Stakeholders

| Role | Person / Team | Responsibility |
|------|--------------|----------------|
| Product / Sales | Sales Lead | กำหนด package tier + features ที่ขายให้ลูกค้า |
| Backend Engineering | Backend Team | สร้าง schema/API + migrate FK |
| Frontend Engineering | Frontend Team | สร้าง 3 หน้า CRUD + tab ใน client detail |
| Super Admin (Internal) | Internal Operations | ใช้งาน CMS เพื่อจัดการ master data + override |
| QA | QA Team | regression test resolver + permission integration |
| DevOps | DevOps | DB migration deployment + rollback |

## 5. Functional Requirements (ความต้องการเชิงฟังก์ชัน)

### FR-1: Feature CRUD
- Description: super_admin สร้าง/แก้ไข/ลบ feature ผ่าน CMS โดย feature ประกอบด้วย code, name (TH/EN), description (TH/EN), `menu_keys` (path เมนูใน PermissionDefault ที่จะเห็นเมื่อมี feature)
- Acceptance Criteria:
  - [ ] super_admin สร้าง feature ใหม่ได้ พร้อมเลือก menu_keys จาก dropdown ที่ list เมนูทั้งหมดของระบบ
  - [ ] ระบบ validate `code` ห้ามซ้ำ (lowercase + underscore)
  - [ ] ระบบ validate ทุก `menu_keys` ต้องอยู่ใน `PermissionDefault` (reject 400 ถ้าไม่ใช่)
  - [ ] super_admin แก้ไข feature (รวม menu_keys) → save → ทุกบริษัทที่ได้รับ feature นี้เห็นเมนูใหม่ทันที
  - [ ] super_admin ลบ feature → block ถ้ามี package/addon อ้างอิง (return 400 พร้อมข้อความ)
  - [ ] List feature รองรับ search, sort by code/name/sortOrder, pagination

### FR-2: Package CRUD + จัดการ feature ของ package
- Description: super_admin สร้าง/แก้ไข/ลบ package พร้อมระบุ features ที่ package นี้ปลด unlock
- Acceptance Criteria:
  - [ ] CRUD package ครบ: code, name, description, billing_interval, price, user_limit, is_recommend, is_contact_us, is_active
  - [ ] super_admin replace feature list ของ package ได้ (`PUT /packages/:uuid/features`)
  - [ ] หน้า detail แสดง preview "เมนูทั้งหมดที่ package นี้ปลด unlock" — รวม `menu_keys` ของทุก feature, dedupe
  - [ ] ลบ package → block ถ้ามี company อ้างอิง (`comp_companies.package_id`)
  - [ ] Schema ตรงตาม `subscription-package-summary.md` (id, uuid, name JSONB, billing_cycle, num_of_employees → user_limit, prices → price_amount/currency, is_recommend, is_contact_us)

### FR-3: Addon CRUD + จัดการ feature ของ addon
- Description: super_admin จัดการ addon ที่ลูกค้าซื้อเสริมจาก package ได้ — addon ปลด features เพิ่มเติม
- Acceptance Criteria:
  - [ ] CRUD addon: code, name, description, billing_interval (month/year/null=oneTime), price, **is_quantifiable**, **max_quantity**
  - [ ] is_quantifiable=true → max_quantity required (validation)
  - [ ] super_admin replace features ของ addon ได้
  - [ ] ลบ addon → block ถ้ามี company purchase อยู่ใน `comp_company_addons`

### FR-4: Per-company feature override (Delta model)
- Description: super_admin override feature ของแต่ละ company ได้ — feature เปิดเสริมจาก package, ปิดบาง feature, หรือ reset กลับไปใช้ default จาก package
- Acceptance Criteria:
  - [ ] หน้า "Features" tab ใน client detail แสดง feature ทั้งหมด พร้อม source: `package` / `addon` / `override-enabled` / `override-disabled` / `default-disabled`
  - [ ] super_admin toggle feature → upsert row ใน `comp_company_features` (override_status = enabled/disabled, มี reason field)
  - [ ] super_admin "Reset to default" → ลบ row ใน `comp_company_features` → กลับไปใช้ default จาก package/addon
  - [ ] หลัง toggle → reload หน้า → state ตรง

### FR-5: Permission Resolver Integration
- Description: ทุก endpoint ที่ return `comp_permission.permission` ต้อง filter ผ่าน effective features ของบริษัทก่อนตอบ
- Acceptance Criteria:
  - [ ] `GET /api/external/auth/v1/permissions` (HappyWork app) → return permission JSONB ที่ filter แล้ว (ไม่มี key ที่ feature ปิด)
  - [ ] หน้า permission management ใน CMS เดิม → filter เมนูที่แสดงตาม company effective menus
  - [ ] เมื่อ save permission ของ user → validate JSONB ต้องไม่มี key นอกเหนือ enabled menus (reject 400)
  - [ ] Resolver logic: `effective = (package.features ∪ addons.features ∪ overrides[enabled]) − overrides[disabled]`
  - [ ] Performance: in-request memoization → ไม่ N+1 ตอน resolve user หลายคน

### FR-6: Authorization
- Description: เฉพาะ super_admin เข้าถึงเมนู feature/package/addon management ได้
- Acceptance Criteria:
  - [ ] Backend: `requireSuperAdmin` middleware ทุก route
  - [ ] Frontend: เมนู "Master Data" + 3 sub-menus visible เฉพาะ super_admin
  - [ ] Tab "Features" ใน client detail visible เฉพาะ super_admin
  - [ ] RBAC matrix: admin / manager / viewer ทั้งหมด no access

### FR-7: Database Schema Migration
- Description: สร้าง 7 tables ใหม่ + migrate `comp_companies.package_id` FK
- Acceptance Criteria:
  - [ ] Tables: `comp_features`, `comp_packages`, `comp_package_features`, `comp_addons`, `comp_addon_features`, `comp_company_features`, `comp_company_addons`
  - [ ] All ใช้ `tableTemplate` (มี id, uuid, timestamps, status_type)
  - [ ] Multilingual JSONB pattern สำหรับ name/description (ตาม AI-RULES-MULTILINGUAL.md)
  - [ ] Seed: `comp_packages` จาก packageMasterData (7 records, preserve UUID; UNIQUE composite `(code, billing_interval)` เพราะ LITE/CORE/PRO มี 2 records monthly+yearly), `comp_addons` จาก addonsMasterData (9 records, UNIQUE composite `(code, billing_interval)` เพราะ E_SLIP/EVALUATION/HAPPY_PM/E_SIGNATURE มี 2 records monthly+yearly)
  - [ ] Backfill `comp_companies.package_id` ใหม่ผ่าน `selected_package_uuid` lookup; row ที่ไม่มี → default Seed
  - [ ] FK migration: drop เก่า → recreate ใหม่
  - [ ] Migration มี down() ทำ rollback ได้

## 6. Non-Functional Requirements (ความต้องการเชิงคุณภาพ)

| Type | Requirement | Note |
|------|------------|------|
| Performance | Resolver < 50ms ต่อ user lookup; cached per request | In-request memoization |
| Performance | Permission API latency increase < 20% เมื่อเทียบ baseline | Index บน comp_company_features.company_id |
| Security | super_admin only middleware ทุก route ของ feature management | RBAC + JWT validation |
| Security | Validate menu_keys ไม่ให้ inject path ที่ไม่มีอยู่จริง | Whitelist จาก PermissionDefault |
| Security | Soft-delete pattern (status_type = archived) | ไม่ hard delete master data |
| Scalability | รองรับ feature 100+, package 20+, addon 20+ | Index ที่จำเป็น (code unique, GIN ของ name) |
| Scalability | รองรับ company 10K+ ที่มี override row | ดู query plan ของ resolver |
| Accessibility | Keyboard navigation ใน UI ของ CRUD form | MUI default + react-hook-form |
| Multilingual | name, description JSONB pattern (TH/EN) | ตาม AI-RULES-MULTILINGUAL.md |
| Auditability | เก็บ created_by, updated_by, archived_by ทุก row | tableTemplate ทำให้แล้ว |

## 7. Constraints (ข้อจำกัด)

### Technical
- ใช้ Knex.js + Objection.js (ตามมาตรฐาน backend)
- ใช้ Zod สำหรับ validation
- ใช้ JSONB pattern สำหรับ multilingual data (Project-wide consistency rule)
- ใช้ MUI + React Hook Form + Yup ใน frontend
- Next.js App Router pattern (เหมือน `survey/*`)
- ไม่ใช้ class-based services (functional pattern ตาม AI-RULES)

### Business
- Phase 4 deploy ต้อง regression test ทุก endpoint ที่ใช้ comp_permission ก่อน
- super_admin จัดการเอง (manual) ในการ map feature → package → addon ครั้งแรกผ่าน CMS

### Time
- Phase 1 (schema + migration): ~2-3 วัน
- Phase 2 (master CRUD BE+FE): ~5-7 วัน
- Phase 3 (per-company toggle + resolver): ~3-5 วัน
- Phase 4 (resolver integration): ~3-5 วัน + regression test
- รวม ~13-20 วัน (ไม่รวม Phase 5 cleanup)

## 8. Assumptions (สมมติฐาน)

- `comp_companies.selected_package_uuid` ของบริษัท active ทุกรายมีค่าตรงกับ UUID ใน `packageMasterData` (จะ pre-validate ก่อน migrate)
- v2 Stripe flow stable แล้ว (production) — ไม่ revert กลับ v1
- Super admin คนเดียว/ทีมเดียวเท่านั้นใช้ feature management — ไม่ต้องทำ multi-tenant อีกชั้น
- Comp_permission JSONB structure ของ company ทั้งหมดสะท้อนสิ่งที่ admin ต้องการ — Phase 4 filter จะตัดเฉพาะที่ feature ปิดเท่านั้น (ไม่เปลี่ยน data)
- `featureDefinitions.ts` constant ตรงกับ `comp_features` table หลัง seed (สำหรับ compatibility ระหว่าง phase)
- Stripe product/price ของ package + addon มี sync mechanism ใน existing v2 flow — งานนี้แค่ store stripe_product_id/stripe_price_id ใน DB เพื่ออ้างอิง

## 9. Open Questions (คำถามที่ยังไม่ได้คำตอบ)

> **Status: All resolved 2026-05-04** — ทุกคำถามได้ decision จาก stakeholder แล้ว ก่อนเริ่ม Phase 1

- [x] **Q1 — Seed `comp_package_features` mapping ครั้งแรก:** **Decision = C (Seed Cartesian product)** — Seed ทุก feature ใส่ทุก package เป็น default (เปิดเมนูทั้งหมดก่อน → admin ค่อยเข้ามาปิดเฉพาะ feature ที่ขายไม่ครบใน CMS) — เลือก C เพื่อกัน regression: ลูกค้าจะไม่เห็นเมนูหายตอน deploy → SSD/migration M3 ต้องเพิ่ม seed step
- [x] **Q2 — เปลี่ยน package (upgrade/downgrade) → override:** **Decision = A (Keep override)** — `comp_company_features` ที่ admin ตั้งไว้คงอยู่หลัง package change → ไม่ต้องเปลี่ยน schema/logic, แค่ document ใน SSD §10 เพื่อความชัดเจน
- [x] **Q3 — Addon expired (`expires_at < NOW()`):** **Decision = A (Auto-disable feature)** — Resolver ต้องเช็ค `expires_at IS NULL OR expires_at > NOW()` ก่อนนับ addon features เข้า union → SSD §6.2 + permissionResolver service ต้อง filter ตอน load `comp_company_addons`
- [x] **Q4 — Soft-delete feature → override row:** **Decision = B (FK no-CASCADE + Resolver skip archived)** — `comp_company_features.feature_id` FK เป็น `ON DELETE RESTRICT` (ไม่ใช่ CASCADE) เพื่อ keep audit trail; Resolver filter `comp_features.status_type = 'active'` ก่อนนำ feature_id ไปคำนวณ effective menus → SSD §3.1.6 + migration M6 + permissionResolver service ต้องปรับ
- [x] **Q5 — `comp_companies.add_ons` jsonb format:** **Resolved (2026-05-04)** — format = `Array<{ uuid: string; quantity: number }>` ยืนยันจาก migration `20260106-1645-add-addons-field.ts` + `compCompanies.model.ts` (line 55, 125, 189) + v2 admin payment/subscription module ใช้ format นี้สม่ำเสมอ → M7 SQL ใน `migration.md` ใช้ format นี้ได้เลย

## 10. References

### Related docs
- [`subscription-package-summary.md`](../subscription/subscription-package-summary.md) — Package schema source of truth
- [`prd.md`](./prd.md) — Product Requirement Document (user perspective)
- [`ssd.md`](./ssd.md) — System Specification Document (technical)
- [`task-list.md`](./task-list.md) — Task breakdown
- [`testcase.md`](./testcase.md) — Test cases
- [`cr.md`](./cr.md) — Change request log

### Related code paths
- `happywork-backend/src/constant/compPermission.ts` — PermissionDefault
- `happywork-backend/src/database/postgresql/migrations/data/` — Migration directory
- `happywork-backend/src/modules/v1/employee/request/*` — Backend pattern reference
- `happywork-backend/src/api/v1/employee/request/*` — Routes pattern reference
- `happywork-backend/src/modules/v2/admin/package/*` — v2 Stripe flow
- `happywork-sale-cms/src/app/dashboard/survey/*` — Frontend page pattern
- `happywork-sale-cms/src/sections/survey/*` — Frontend section pattern
- `happywork-sale-cms/src/utils/rbac.ts` — RBAC matrix
