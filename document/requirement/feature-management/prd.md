---
feature: feature-management
type: product-requirement-document
created: 2026-05-04
updated: 2026-05-04
owner: SA
status: draft
---

# Product Requirement Document — Feature Management

> มุมมองของผู้ใช้และ business — ตอบคำถาม "ผู้ใช้ได้อะไร, ใช้ยังไง, value คืออะไร"

---

## 1. Overview (ภาพรวม)

ระบบ Feature Management เป็นชุดเครื่องมือใน CMS ที่ช่วยให้ทีม Sales/Operations ของ HappyWork จัดการชุด feature ที่ลูกค้าแต่ละรายได้รับ — โดยอิงจาก package ที่ลูกค้าซื้อ + addon ที่เลือกเสริม + ความสามารถ override เฉพาะรายลูกค้า

ระบบนี้แทนที่ระบบเก่าที่ feature/package ถูก hardcode ไว้ใน source code ทำให้ Sales ขายแพ็กเกจที่มี feature ต่างกันได้, Operations เปิด/ปิด feature ให้ลูกค้ารายตัวได้, และ user ของลูกค้าเห็นเมนูใน HappyWork app ตามสิทธิ์ที่บริษัทตัวเองได้รับ

## 2. User Personas (ผู้ใช้)

| Persona | Description | Need |
|---------|------------|------|
| **Sales Lead (Internal)** | ทีม Sales ที่กำหนด pricing tier + feature set ของแต่ละ package | ปรับ feature/package โดยไม่ต้องรอ engineer; preview เมนูที่ลูกค้าจะเห็นจริง |
| **Operations / Customer Success (Internal)** | ทีมที่ดูแลลูกค้าหลังขาย | เปิด feature เพิ่มให้ลูกค้า key account; ปิด feature ที่ลูกค้าไม่ได้ซื้อ; audit ประวัติการเปลี่ยน |
| **Super Admin** | Internal ที่มี full access บน CMS | จัดการ master data (feature/package/addon); จัดการ override per company |
| **End User (Customer Employee)** | พนักงานของบริษัทลูกค้าที่ใช้ HappyWork app | เห็นเมนูที่ตัวเองมีสิทธิ์ใช้ — ตามที่บริษัทซื้อ + admin ของบริษัท assign role |

## 3. User Stories

### US-1: Sales Lead จัดการ feature ในระบบ
- Story:
  ```
  As a Sales Lead,
  I want to define what features exist in HappyWork and which menus they unlock,
  So that I can build packages that map clearly to what customers see in the product.
  ```
- Acceptance Criteria:
  - [ ] Given a super_admin login, when I open `/dashboard/feature-management`, then I see a list of all features with code, name, and number of menus they unlock
  - [ ] Given I click "Create Feature", when I fill in code, name (TH/EN), description, and select menu_keys from the available list, then the feature is created
  - [ ] Given a feature is used by 2 packages, when I try to delete it, then I see an error "This feature is used by 2 packages and 0 addons. Remove the references first."

### US-2: Sales Lead กำหนด feature ของ package
- Story:
  ```
  As a Sales Lead,
  I want to assign features to each package,
  So that customers buying a specific tier get exactly the features promised.
  ```
- Acceptance Criteria:
  - [ ] Given I'm on the package detail page, when I click "Edit Features", then I see a dialog with checkboxes of all available features
  - [ ] Given I select 5 features and click Save, when I revisit the package detail page, then "Effective Menus" preview shows the union of menus from all 5 features (deduplicated)
  - [ ] Given a feature is selected, when I deselect it and Save, then the package no longer includes that feature

### US-3: Sales Lead กำหนด addon ที่ปลด feature เพิ่ม
- Story:
  ```
  As a Sales Lead,
  I want to define addons (e.g., HappyBeacon, E-Slip) that unlock specific features when customers purchase them on top of their package,
  So that we can monetize advanced features separately.
  ```
- Acceptance Criteria:
  - [ ] Given I create an addon "HappyBeacon", when I check "Allow Quantity Selection" and set max_quantity = 100, then the addon is saved as quantifiable
  - [ ] Given an addon is quantifiable but I leave max_quantity empty, when I click Save, then I see validation error
  - [ ] Given I'm on the addon detail page, when I click "Edit Features", then I select the features this addon unlocks (e.g., HappyBeacon unlocks "Location Tracking" and "Beacon Reporting")

### US-4: Operations override feature ของลูกค้ารายตัว
- Story:
  ```
  As an Operations team member,
  I want to enable or disable specific features for a particular customer (overriding their package default),
  So that I can give VIP customers extra features during trial, or remove features for special arrangements.
  ```
- Acceptance Criteria:
  - [ ] Given I'm on `/dashboard/client-management/{uuid}` and click the "Features" tab, then I see all features in the system with their current status and source (Package / Addon / Override / Default-disabled)
  - [ ] Given I toggle a feature ON for this company (which was disabled by default), when prompted I enter reason "Trial extension by sales team", then the toggle saves and source badge changes to "Override (Enabled)"
  - [ ] Given a feature was overridden, when I click "Reset to Default", then the override is removed and the feature returns to package/addon default state

### US-5: End user เห็นเมนูตาม feature ของบริษัท
- Story:
  ```
  As an end user of a customer company,
  I want to see only the menus my company has access to in HappyWork app,
  So that I'm not confused by features I can't use.
  ```
- Acceptance Criteria:
  - [ ] Given my company has feature "Payroll" disabled (via override), when I login to HappyWork app, then the Payroll menu and all submenus are hidden
  - [ ] Given my company purchases the HappyBeacon addon, when I login as an admin role, then I see the Location Tracking menu (which is unlocked by HappyBeacon)
  - [ ] Given my role is Employee (limited permission), when I login, then I see only menus that are both (a) within my role's permissions and (b) within my company's effective features

## 4. User Journey / Flow

### Journey A: Setup ครั้งแรก (Phase 1-2 deploy)
```
[Super Admin login CMS]
  → [Master Data > Feature Management]
  → Create features (with menu_keys)
  → [Master Data > Package Management]
  → Create packages → assign features to each package
  → [Master Data > Addon Management]
  → Create addons → assign features to each addon
  → [Validate] Open package detail, check "Effective Menus Preview" matches expectations
```

### Journey B: ลูกค้าใหม่สมัคร package
```
[Customer signs up via marketing site]
  → Customer's selected_package_uuid recorded in comp_companies
  → [System auto] comp_companies.package_id linked to comp_packages
  → [End user logs in to HappyWork app]
  → Backend resolver: package.features → effective menu_keys → filter comp_permission
  → User sees menus matching their package
```

### Journey C: Operations เปิด feature ให้ลูกค้า key account
```
[Operations login CMS]
  → [Client Management > {customer name}]
  → Click "Features" tab
  → Filter by "Default Disabled"
  → Toggle feature "Advanced Reporting" → ON
  → Enter reason "Pilot test for upsell"
  → Save
  → [End user of that customer logs in]
  → Sees "Advanced Reporting" menu now (didn't before)
```

### Journey D: ลูกค้าซื้อ addon เสริม
```
[Customer purchases HappyBeacon addon via subscription flow]
  → Stripe webhook updates comp_companies.add_ons
  → [Background job migrates to comp_company_addons]
  → [End user logs in]
  → Resolver: package.features ∪ addon.features (HappyBeacon unlocks Location Tracking)
  → Menu Location Tracking appears
```

## 5. UI / UX Requirements

### Screens / Pages ที่ต้องมี
1. **Feature Management** — `/dashboard/feature-management`
   - List page (table + search + filter + create button)
   - Create page
   - Detail page (with usage info: which packages/addons reference)
   - Edit page
2. **Package Management** — `/dashboard/package-management`
   - List page
   - Create page
   - Detail page (with "Features" section + "Effective Menus" preview)
   - Edit page
3. **Addon Management** — `/dashboard/addon-management`
   - List page
   - Create page (with conditional `max_quantity` field)
   - Detail page (with "Features Unlocked" section)
   - Edit page
4. **Client Detail — Features Tab** — `/dashboard/client-management/{uuid}` (existing page, new tab)
   - List of all features with toggle + source badge + reset button

### Interaction patterns
- Pages ใช้ Next.js App Router แยก `page.tsx` (list/create/[id]/edit) ตาม pattern ของ `survey/*`
- Forms ใช้ React Hook Form + Yup
- Multilingual fields = paired TH/EN inputs (component `multilingual-text-field.tsx`)
- Menu selection = tree-style multi-select (group by main category)
- Source badge = colored chip (Package/Addon/Override-enabled/Override-disabled/Default)
- Confirm dialog เมื่อ delete หรือ toggle (with optional reason field)

### Accessibility requirements
- Keyboard navigation ใน form ทุกหน้า
- Screen reader compatibility (label + aria-label ครบ)
- Error message ชัดเจน (Yup validation message in TH)

### Mobile / responsive
- CMS แสดงผลบน desktop เป็นหลัก (super_admin ใช้งานบน laptop)
- Tablet: ใช้งานได้ในระดับ list view + form
- Mobile: ไม่ใช่ priority ในงานนี้

## 6. Business Rules (กฏทางธุรกิจ)

| Rule ID | Description | Enforced where |
|---------|------------|----------------|
| BR-1 | Feature `code` ต้อง unique ทั้งระบบ; lowercase + underscore + digits เท่านั้น | Backend create/update validation + DB unique constraint |
| BR-2 | Feature `menu_keys` ทุก path ต้องอยู่ใน `PermissionDefault` | Backend service validation (`getAvailableMenuKeys()`) |
| BR-3 | ลบ feature ไม่ได้ถ้ามี package/addon อ้างอิง | Backend `deleteFeatureService` block |
| BR-4 | ลบ package ไม่ได้ถ้ามี company อ้างอิง (`comp_companies.package_id`) | Backend `deletePackageService` block |
| BR-5 | ลบ addon ไม่ได้ถ้ามี company purchase อยู่ (`comp_company_addons`) | Backend `deleteAddonService` block |
| BR-6 | Addon `is_quantifiable=true` → `max_quantity` required (> 0) | Backend + Frontend validation |
| BR-7 | Effective features ของ company = `(package.features ∪ addons.features ∪ overrides[enabled]) − overrides[disabled]` | `permissionResolver` service |
| BR-8 | Comp_permission JSONB เก็บข้อมูลเต็ม — filter ตอนอ่านเท่านั้น | Resolver pattern |
| BR-9 | Override row ของ feature ที่ถูกลบ → CASCADE ลบไปด้วย | DB FK ON DELETE CASCADE |
| BR-10 | `comp_companies.package_id` ต้องไม่เป็น NULL ทุก row หลัง M8 (default Seed) | Migration M8 step (b) |
| BR-11 | เฉพาะ `super_admin` role เข้าถึง feature management ได้ | `requireSuperAdmin` middleware + Frontend RBAC |
| BR-12 | Multilingual field (`name`, `description`) ต้องมี TH + EN ทั้งคู่ในการสร้าง | Zod + Yup schema |

## 7. Success Metrics (ตัววัดความสำเร็จ)

### Quantitative
- Time to onboard new package: ลดจาก ~3 วัน (ต้อง engineer push code) เป็น < 30 นาที (super_admin ทำผ่าน CMS)
- จำนวน feature/package/addon ที่ active ในระบบ (เพิ่มขึ้นต่อเดือน)
- จำนวนการ override per company (audit trail) ที่บันทึกใน `comp_company_features`
- Permission API latency: ไม่เกิน +20% เทียบ baseline

### Qualitative
- Sales feedback: สามารถสร้าง package ใหม่ได้เองโดยไม่พึ่ง engineer
- Operations feedback: เปิด feature ให้ key account ได้เร็ว
- End user feedback: ไม่เห็น menu ที่ไม่ได้ใช้
- ไม่มี regression bug จาก permission filter ใน Phase 4

## 8. Out of Scope (ไม่ทำใน feature นี้)

- ไม่ทำ versioning ของ feature/package configuration (audit log แค่ใน override)
- ไม่ทำ feature dependencies (เช่น "feature A ต้องเปิดก่อน feature B")
- ไม่ทำ time-based expiration ของ override (`comp_company_features.expires_at`)
- ไม่ทำ bulk override (ตั้งค่า feature ของหลาย company พร้อมกัน)
- ไม่ทำ self-service portal สำหรับลูกค้าให้ override เอง (เฉพาะ internal super_admin)
- ไม่ migrate v1 caller ของ `const_packages` ทั้งหมด (Phase 5)
- ไม่ทำ public API สำหรับลูกค้าดึงข้อมูล feature ของตัวเอง

## 9. Dependencies (พึ่งพา)

- **v2 Stripe flow**: ต้อง stable ใน production ก่อน (เพราะ M2 seed ใช้ packageMasterData UUID, M8 migrate ใช้ selected_package_uuid)
- **`comp_companies.selected_package_uuid` data quality**: ต้อง pre-validate ว่า uuid ของ company active ทั้งหมด match กับ packageMasterData (มี script ตรวจใน migration.md)
- **Existing permission system**: ต้องไม่มี breaking change ระหว่าง Phase 1-3 (Phase 4 เป็นจุดเปลี่ยน)
- **`comp_companies.add_ons` (jsonb) format**: ต้องสำรวจ format จริงก่อนเขียน M7 migration logic

## 10. Risks (ความเสี่ยง)

| Risk | Impact | Mitigation |
|------|--------|-----------|
| User เห็นเมนูน้อยลงหลัง Phase 4 deploy (regression) | High | Pre-deploy: backfill ให้ทุกบริษัทมี feature ครบจาก package; integration test ทุก permission API |
| Migration M8 ทำให้ FK orphan | High | Pre-validation script ก่อน apply; rollback strategy พร้อม |
| Performance ของ resolver แย่กว่าเดิมเมื่อมี company เยอะ | Medium | In-request memoization; index `(company_id)`; load test ก่อน deploy |
| Sales/Operations ใช้ CMS ไม่เป็น | Low | Training session + documentation (PRD ใช้ guide ได้) |
| Constants `featureDefinitions.ts` กับ `comp_features` table out of sync ระหว่าง phase | Medium | Phase 4 ก่อน switch over → run script เปรียบเทียบ |
| Addon ที่ expired ไม่ auto-disable feature | Low (out of scope) | Open Question ใน requirement.md §9 — ทำใน future iteration |
