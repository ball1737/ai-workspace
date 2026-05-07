# Stripe Sync Augmentation — Requirement Handoff

> **Purpose:** เอกสารเดียวที่ครบทุกอย่างสำหรับ "augment" (ไม่ใช่ rewrite) Stripe sync flow ให้เก็บข้อมูลเพิ่มในตารางใหม่ (`comp_packages` + `comp_company_addons`) **โดยไม่กระทบ flow เดิมที่ทำงานดีอยู่แล้ว** — ค่อย ๆ ลบ flow/table ที่ verify แล้วว่าไม่มีใครใช้, สร้าง flow ใหม่บน schema ใหม่สำหรับ feature ที่จะเพิ่มในอนาคต
>
> **Author:** Lead — 2026-05-07 (updated strategy 2026-05-07)
> **Owners:** TBD
> **Source feature branch (foundation):** `ball/feature/feature-management` (hw-backend `8c8f608e` + hw-sale-cms `829ffb0`)
> **Target start:** หลัง feature-management deploy production stable แล้ว
>
> **🔑 Guiding Strategy (user decision 2026-05-07):**
> 1. **รักษา Stripe flow เก่าไว้** — มันทำงานดีอยู่แล้ว, ห้ามแก้ caller เดิมของ `const_packages` + `comp_companies.add_ons`
> 2. **เพิ่ม dual-write** — เขียนข้อมูลลง table ใหม่ (`comp_company_addons`) เพิ่มเติมจากที่เขียน JSONB เดิม
> 3. **ลบ flow/table ที่ไม่ใช้แล้วเท่านั้น** — ต้อง verify ว่า zero caller ก่อน drop
> 4. **เพิ่ม flow ใหม่บน schema ใหม่** — feature ใหม่ใช้ resolver + comp_packages + comp_company_addons ตรง ๆ ไม่ผ่าน legacy

---

## สารบัญ

1. [Executive Summary](#1-executive-summary)
2. [Foundation — สิ่งที่ทำเสร็จใน feature-management](#2-foundation--สิ่งที่ทำเสร็จใน-feature-management)
   - 2.1 [Database Schema ใหม่](#21-database-schema-ใหม่)
   - 2.2 [Migration List](#22-migration-list)
   - 2.3 [Permission Resolver Pattern](#23-permission-resolver-pattern)
   - 2.4 [Backend Endpoints ใหม่](#24-backend-endpoints-ใหม่)
   - 2.5 [Frontend Pages ใหม่](#25-frontend-pages-ใหม่)
   - 2.6 [Stripe UUID Sync Endpoints](#26-stripe-uuid-sync-endpoints)
3. [Legacy Code & Schema Inventory (Reference Only)](#3-legacy-code--schema-inventory-reference-only)
   - 3.1 [`const_packages` Table + Callers](#31-const_packages-table--callers)
   - 3.2 [`comp_companies.add_ons` JSONB + Callers](#32-comp_companiesadd_ons-jsonb--callers)
   - 3.3 [Stripe Webhook Flows ปัจจุบัน](#33-stripe-webhook-flows-ปัจจุบัน)
   - 3.4 [Mapping Logic ที่มีอยู่](#34-mapping-logic-ที่มีอยู่)
4. [Plan สำหรับ Augmentation (Strategy: Additive, Non-Breaking)](#4-plan-สำหรับ-augmentation-strategy-additive-non-breaking)
   - 4.1 [Wave Breakdown](#41-wave-breakdown)
   - 4.2 [Risks & Mitigations](#42-risks--mitigations)
   - 4.3 [Migration Strategy](#43-migration-strategy)
   - 4.4 [Test Strategy](#44-test-strategy)
5. [Open Questions ที่ต้อง Resolve ก่อนเริ่ม](#5-open-questions-ที่ต้อง-resolve-ก่อนเริ่ม)
6. [References (File Paths)](#6-references-file-paths)
7. [Appendix — JSONB Shape Specs](#7-appendix--jsonb-shape-specs)

---

## 1. Executive Summary

### 1.1 สถานะปัจจุบันของ Stripe Sync
ระบบ Stripe sync ปัจจุบันใช้ schema เก่าและ **ทำงานได้ดีในการ production**:

- **`const_packages` table** — package master เดิม; ใช้ใน v1 code (~22 references): paymentProcessing, subscription, customers, leads, quotations, employees, trialExpiration
- **`comp_companies.add_ons` JSONB column** — เก็บ addon ที่ company ซื้อ; ใช้ใน Stripe webhook (v1+v2) + payment + cron job (~17 callers)
- **comp_packages + comp_company_addons** — schema ใหม่จาก feature-management Phase 1-7 ที่มี data backfill จาก legacy แล้ว (M7+M8) แต่ยังไม่ถูก write update โดย live flow

**ผลกระทบที่ยอมรับได้ (per user decision 2026-05-07):**
- Drift ระหว่าง JSONB เก่า กับ table ใหม่ → fix ด้วย dual-write + reconciliation job
- Schema duplication ชั่วคราว → ยอมรับได้เพื่อ minimize risk ต่อ revenue path
- Refactor caller เก่าทั้งหมด ❌ **ห้ามทำ** — risk ไม่คุ้ม

### 1.2 เป้าหมาย (Strategy ใหม่ — Additive, Non-Breaking)

**ทำ ✅:**
1. **Dual-write at Stripe webhook** — preserve `comp_companies.add_ons` JSONB write เดิม + เพิ่ม write ลง `comp_company_addons` table ด้วย (sync ผ่าน listener / helper service)
2. **Reconciliation job** — daily compare JSONB array vs table rows + alert on drift
3. **Drop dead code/table เฉพาะที่ verified zero caller** — ไม่ touch ของที่มี caller active
4. **Build new feature flows on top of new schema** — feature ที่จะเพิ่มในอนาคต ใช้ Resolver + comp_packages + comp_company_addons ตรง ๆ ไม่ผ่าน legacy

**ไม่ทำ ❌:**
1. **ห้าม refactor caller** ของ `const_packages` (12 v1 services ยังคงเดิมหมด)
2. **ห้าม refactor caller** ของ `comp_companies.add_ons` JSONB read (legacy v2 webhook ยัง write JSONB เดิม)
3. **ห้าม drop `const_packages` table** ตอนนี้ — มี active caller อยู่
4. **ห้าม drop `comp_companies.add_ons` column** ตอนนี้ — มี active caller อยู่
5. **ห้าม touch Stripe SDK call patterns** — ที่ทำงานได้แล้วต้องอยู่เหมือนเดิม

### 1.3 สิ่งที่ Foundation ให้ใช้ (จาก feature-management)
- ✅ Schema ใหม่ครบ: `comp_packages`, `comp_addons`, `comp_features`, `comp_package_features`, `comp_addon_features`, `comp_company_features`, `comp_company_addons`
- ✅ Migration M1-M11 (M9+M10+M11 รันบน dev DB แล้ว — 2026-05-07)
- ✅ `Permission Resolver` pattern (in-request cache, Q4=B status filter) — สำหรับ flow ใหม่
- ✅ Master CRUD endpoints (Sale Dashboard) สำหรับ feature/package/addon
- ✅ **Stripe UUID Sync endpoints** — `PATCH /:packageUuid/identifier` (Package B4) + `PATCH /:addonUuid/identifier` (Phase 7)
- ✅ Per-company feature toggle UI (Phase 3)
- ✅ Mobile menu parity end-to-end (Phase 5)

### 1.4 Wave Breakdown สรุป

| Wave | Scope | Risk | Est. |
|------|-------|------|------|
| A | Audit + verify Stripe metadata + decision lock | LOW | 1-2 d |
| B | Add dual-write helper to Stripe webhook (preserve old + write new) | MEDIUM | 3-5 d |
| C | Reconciliation job + observability dashboard | LOW-MEDIUM | 2-3 d |
| D | Drop dead code/table — verified-unused only | LOW | 1-2 d |
| E | Foundation/scaffolding for new feature flows on new schema | LOW | (per feature) |

**ประมาณ:** ~10-15 ไฟล์ touched (เฉพาะใน webhook + jobs + new helpers); 1 sprint ขึ้นกับ scope ที่ user pick

---

## 2. Foundation — สิ่งที่ทำเสร็จใน feature-management

### 2.1 Database Schema ใหม่

#### 2.1.1 `comp_packages` (replaces const_packages)
**Path:** `happywork-backend/src/database/postgresql/models/data/compPackages.model.ts`

| Column | Type | Constraint | Note |
|--------|------|------------|------|
| `id` | bigint | PK | numeric internal ID |
| `uuid` | uuid | UQ NOT NULL | **public ID — match Stripe product ID** |
| `code` | varchar(50) | NOT NULL | tier code (`seed`, `lite`, `core`, `pro`) |
| `name` | jsonb | NOT NULL | `{th: "...", en: "..."}` (GIN index) |
| `short_name` | varchar(50) | nullable | ใช้ใน checkout summary |
| `description` | jsonb | nullable | `{th, en}` |
| `billing_interval` | varchar | nullable | `'month'` \| `'year'` \| `null` (one-time) |
| `price_amount` | decimal(12,2) | NOT NULL | |
| `currency` | varchar(3) | NOT NULL | default `'thb'` |
| `user_limit_min` | int | nullable | replace `const_packages.numOfEmployees` |
| `user_limit_max` | int | nullable | |
| `is_recommend` | bool | default false | |
| `is_contact_us` | bool | default false | |
| `is_active` | bool | default true | display flag |
| `stripe_product_id` | varchar(255) | nullable | Stripe sync (optional manual override) |
| `stripe_price_id` | varchar(255) | nullable | Stripe sync |
| `sort_order` | int | default 0 | |
| `status_type` | varchar | default 'active' | 'active' \| 'inactive' |
| timestamps + audit | | | created_at/updated_at/created_by/updated_by |

**Constraints:**
- Composite UNIQUE `(code, billing_interval)` — same code allowed across billing intervals (month vs year)
- Index `idx_comp_packages_is_active`, GIN `idx_comp_packages_name_gin`

**Seed data (M2):** 7 packages — Seed (free), Lite month/year, Core month/year, Pro month/year. **UUIDs preserved** จาก `const_packages` เพื่อให้ Stripe ที่ใช้ UUID เดิมยัง map ได้

#### 2.1.2 `comp_addons` (replaces ad-hoc addon definitions)
**Path:** `happywork-backend/src/database/postgresql/models/data/compAddons.model.ts`

| Column | Type | Constraint | Note |
|--------|------|------------|------|
| `id` | bigint | PK | |
| `uuid` | uuid | UQ NOT NULL | **public ID — match Stripe product ID** |
| `code` | varchar(50) | NOT NULL | addon code (e.g. `e_slip`, `evaluation`) |
| `name` | jsonb | NOT NULL | `{th, en}` |
| `short_name` | varchar(50) | nullable | |
| `description` | jsonb | nullable | |
| `billing_interval` | varchar | nullable | `'month'` \| `'year'` \| `null` |
| `price_amount` | decimal(12,2) | | |
| `currency` | varchar(3) | | |
| `is_quantifiable` | bool | default false | ถ้า true → company ซื้อหลาย unit ได้ |
| `max_quantity` | int | nullable | required เมื่อ is_quantifiable=true |
| `is_recommend` | bool | default false | |
| `is_active` | bool | default true | |
| `stripe_product_id` | varchar(255) | nullable | |
| `stripe_price_id` | varchar(255) | nullable | |
| `sort_order` | int | | |
| `status_type` | varchar | | |
| timestamps + audit | | | |

**Constraints:**
- Composite UNIQUE `(code, billing_interval)`
- Index `idx_comp_addons_is_active`, GIN `idx_comp_addons_name_gin`

**Seed data (M4):** 9 addons — HappyBeacon, E-Slip month/year, Evaluation month/year, HappyPM month/year, E-Signature month/year. **UUIDs preserved** เช่นกัน

#### 2.1.3 `comp_features` (Master feature registry)
**Path:** `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts`

| Column | Type | Note |
|--------|------|------|
| `id`, `uuid` | bigint, uuid | |
| `code` | varchar(50) UQ | |
| `name`, `description` | jsonb | multilingual |
| `menu_keys` | jsonb array | web menus that this feature unlocks |
| `mobile_menu_keys` | jsonb array | **Phase 5** — mobile menus (mirror) |
| `sort_order`, `status_type` | | |

**Indexes:** GIN `idx_comp_features_name_gin`, GIN `idx_comp_features_mobile_menu_keys_gin`

**Seed data (M1):** 14 features ใน 3 tiers (SEED/CORE/PRO)

#### 2.1.4 N:M Link Tables

**`comp_package_features`** (M3): composite PK `(package_id, feature_id)`; seed = Cartesian product all active packages × all active features

**`comp_addon_features`** (M5): composite PK `(addon_id, feature_id)`; no seed (admin maps via CMS)

#### 2.1.5 `comp_company_features` (Phase 1 + Phase 3 audit-trail upgrade)
**Path:** `happywork-backend/src/database/postgresql/models/data/compCompanyFeatures.model.ts`

| Column | Type | Note |
|--------|------|------|
| `company_id` | FK CASCADE | |
| `feature_id` | FK **RESTRICT** | (Q4=B keep audit; resolver filter active features at runtime) |
| `override_status` | enum | `'enabled'` \| `'disabled'` |
| `reason` | text | nullable |
| `status_type` | | `'active'` \| `'archived'` (audit trail) |

**Constraint:** Partial UNIQUE `(company_id, feature_id) WHERE status_type='active'` — ทำให้สร้าง override ใหม่ได้แม้มี archived row เก่า (Phase 3 W4 audit-trail flow)

#### 2.1.6 `comp_company_addons` (replaces comp_companies.add_ons JSONB) ⭐
**Path:** `happywork-backend/src/database/postgresql/models/data/compCompanyAddons.model.ts`

| Column | Type | Note |
|--------|------|------|
| `id`, `uuid` | bigint, uuid | |
| `company_id` | FK CASCADE | |
| `addon_id` | FK CASCADE | |
| `quantity` | int | >= 1 |
| `purchased_at` | timestamp | |
| `expires_at` | timestamp | nullable (Q3=A: NULL = never expires) |
| `status_type` | | `'active'` \| `'archived'` |
| timestamps | | |

**Resolver behavior:** filter `expires_at IS NULL OR expires_at > NOW()` ที่ runtime (ไม่มี cron job เปลี่ยน status_type)

> ⭐ นี่คือ table หลักที่ Stripe Sync Rewrite ต้องเขียน data เข้ามา (replace JSONB)

### 2.2 Migration List

ทุก migration อยู่ที่ `happywork-backend/src/database/postgresql/migrations/data/`

| # | Filename | Phase | Purpose |
|---|----------|-------|---------|
| M1 | `20260504-1000-create-comp_features.ts` | 1 | comp_features + 14 seed |
| M2 | `20260504-1001-create-comp_packages.ts` | 1 | comp_packages + 7 seed (UUIDs preserved) |
| M3 | `20260504-1002-create-comp_package_features.ts` | 1 | N:M link + Cartesian seed |
| M4 | `20260504-1003-create-comp_addons.ts` | 1 | comp_addons + 9 seed (UUIDs preserved) |
| M5 | `20260504-1004-create-comp_addon_features.ts` | 1 | N:M link (no seed) |
| M6 | `20260504-1005-create-comp_company_features.ts` | 1 | per-company feature override |
| M7 | `20260504-1006-create-comp_company_addons.ts` | 1 | per-company addons + **backfill from add_ons JSONB** |
| M8 | `20260504-1007-alter-comp_companies-package-id-fk.ts` | 1 | replace FK from const_packages.id → comp_packages.id |
| M6.1 | `20260506-1700-alter-comp_company_features-partial-unique.ts` | 3 | partial UNIQUE WHERE status_type='active' |
| M9 | `20260506-1800-alter-comp_features-add-mobile_menu_keys.ts` | 5 | + mobile_menu_keys jsonb column |
| M10 | `20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts` | 5 | backfill SEED tier mobile menus |
| M11 | `20260507-0900-drop-sale-dashboard-disabled-features-table.ts` | 6 W6.1 | drop legacy table |

> ✅ **M9 + M10 + M11 รันบน dev DB แล้ว** (2026-05-07) — ทุก migration up-to-date

**Backfill ที่ M7 ทำให้แล้ว:**
```sql
INSERT INTO comp_company_addons (company_id, addon_id, quantity, purchased_at, ...)
SELECT cc.id, ca.id, COALESCE((addon_item->>'quantity')::int, 1), cc.created_at, ...
FROM comp_companies cc
CROSS JOIN LATERAL jsonb_array_elements(cc.add_ons) AS addon_item
JOIN comp_addons ca ON ca.uuid = (addon_item->>'uuid')::uuid
WHERE cc.add_ons IS NOT NULL AND jsonb_array_length(cc.add_ons) > 0
```

**Backfill ที่ M8 ทำให้แล้ว:**
```sql
UPDATE comp_companies cc
SET package_id = cp.id
FROM comp_packages cp
WHERE cp.uuid = cc.selected_package_uuid;

-- fallback to Seed for legacy companies without selected_package_uuid
UPDATE comp_companies
SET package_id = (SELECT id FROM comp_packages WHERE code = 'seed')
WHERE selected_package_uuid IS NULL;
```

> 💡 หลัง M7 + M8 — `comp_company_addons` มี data ครบจาก `add_ons` JSONB เดิม + `comp_companies.package_id` FK ชี้ไปที่ `comp_packages` แล้ว — แต่ **legacy code ยังเขียน `add_ons` JSONB อยู่** ดังนั้น rewrite ต้อง update writer ก่อน drop column

### 2.3 Permission Resolver Pattern

**Module:** `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/`

#### 2.3.1 Functions ที่มีให้ใช้

| Function | Input | Output | Purpose |
|----------|-------|--------|---------|
| `resolveEffectivePackageIdService(companyId, cache?)` | string | string (package_id) | get package_id with Seed fallback ถ้า NULL |
| `resolveCompanyEffectiveFeatureIdsService(companyId, cache?)` | string | `Set<string>` | union ของ package + active addon features ∪ overrides |
| `resolveCompanyEffectiveMenuKeysService(companyId, cache?)` | string | `Set<string>` | web menu keys ที่ enabled |
| `resolveCompanyEffectiveMobileMenuKeysService(companyId, cache?)` | string | `Set<string>` | mobile menu keys ที่ enabled |
| `filterPermissionByMenuKeysService(jsonb, allowedKeys)` | obj, Set | obj | filter PermissionDefault tree (web) |
| `filterPermissionMobileByMenuKeysService(jsonb, allowedKeys)` | obj, Set | obj | filter PermissionMobileDefault tree (mobile) |
| `resolveUserEffectivePermissionService(userUuid, cache?)` | string | jsonb | full filtered web permission |
| `resolveUserEffectivePermissionMobileService(userUuid, cache?)` | string | jsonb | full filtered mobile permission |

#### 2.3.2 In-request Cache (Option C)
- Type: `ResolverCache = Map<string, unknown>`
- Caller สร้าง `new Map()` ต่อ request (ไม่ใช่ global)
- Key namespace:
  - `featureIds:${companyId}`
  - `menuKeys:${companyId}` / `mobileMenuKeys:${companyId}`
  - `packageId:${companyId}`
  - `userPermissionJsonb:${userUuid}`
- Cache shared ระหว่าง web + mobile resolver call → mobile call hit cache 2/3 keys, เพิ่มแค่ 1 batch DB query

#### 2.3.3 Repository Functions (active-only filter, Q4=B)
**Path:** `permissionResolver.repository.ts`

ทุก query filter `feature.status_type='active'`, `comp_company_features.status_type='active'`, `comp_company_addons.status_type='active'` + `(expires_at IS NULL OR expires_at > NOW())` — ทำให้ resolver result reflect สิ่งที่ enabled ใน UI จริง

> 💡 **ใช้ resolver แทนการอ่าน comp_companies.add_ons เอง** — Rewrite ควร refactor caller ของ add_ons ให้ใช้ resolver functions เหล่านี้ (ไม่ใช่ raw query)

### 2.4 Backend Endpoints ใหม่

#### 2.4.1 Sale Dashboard — Master CRUD (Phase 2)

**Features** (`/api/v2/sale-dashboard/features`)
- `GET /` — list with usage count
- `POST /` — create
- `GET /:featureUuid` — detail
- `PUT /:featureUuid` — update
- `DELETE /:featureUuid` — soft-delete (archive)
- `GET /menu-keys/available` — list of all available web + mobile menu keys (PermissionDefault tree)

**Packages** (`/api/v2/sale-dashboard/packages`)
- `GET /` — list
- `POST /` — create
- `GET /:packageUuid` — detail with features + effective_menus
- `PUT /:packageUuid` — update metadata
- `DELETE /:packageUuid` — soft-delete (block ถ้ามี company ref)
- `PUT /:packageUuid/features` — replace feature list
- `GET /:packageUuid/effective-menus` — preview unlocked menus `{menuKeys, mobileMenuKeys}` ⭐ Phase 5
- **`PATCH /:packageUuid/identifier`** — ⭐ Stripe UUID sync (Phase 2 Wave B4)

**Addons** (`/api/v2/sale-dashboard/addons`)
- `GET /`, `POST /`, `GET /:addonUuid`, `PUT /:addonUuid`, `DELETE /:addonUuid`
- `PUT /:addonUuid/features` — replace feature list
- **`PATCH /:addonUuid/identifier`** — ⭐ Stripe UUID sync (**Phase 7**)

#### 2.4.2 Per-Company Features (Phase 3)

`/api/v2/sale-dashboard/companies/:companyUuid/features`
- `GET /` — list company effective features (resolver-driven)
- `POST /:featureUuid/toggle` — toggle override (enabled/disabled), archive on match-default

#### 2.4.3 External Auth Permissions (Phase 4)

`GET /api/external/auth/permissions/:userUuid` — return `{permission, permission_mobile}` filtered by company effective menus (BREAKING shape Q3=B)

#### 2.4.4 Admin Permissions (Phase 4 + 2026-05-07 fix)

`GET /api/v1/admin/comp-permission/default` — fallback to `req.user.companyId` ถ้าไม่ส่ง `companyUuid` query (security default)

### 2.5 Frontend Pages ใหม่

ทุกอย่างใน `happywork-sale-cms`:

#### 2.5.1 Master Data Pages (Phase 2)
- `/dashboard/feature-management` — list/detail/edit/create
- `/dashboard/package-management` — list/detail/edit/create
- `/dashboard/addon-management` — list/detail/edit/create

#### 2.5.2 Per-Company Feature Tab (Phase 3, relocated)
- URL: `/dashboard/package-config?from=customer&companyId={UUID}`
- Section: `src/sections/package-config-feature/package-config-feature-view.tsx` (host) → renders `<CompanyFeaturesTabView companyUuid={...}>` (`src/sections/client-management/feature-tab/`)
- Wave 6.1 ลบ `USE_LEGACY_CONFIG_FEATURE` flag + dead code path แล้ว

#### 2.5.3 Reusable UI Components
- `<MenuTreePreview mode="single|split">` — render menu tree (Web|Mobile split mode สำหรับ Phase 5)
- `<MenuKeysSelect>` — multi-select กับ menu tree
- `<MultilingualTextField>` — th/en field pair
- `<PackageUuidEditDialog>` + `<AddonUuidEditDialog>` — Stripe sync dialog

#### 2.5.4 Redux Slices (sale-dashboard-*)
- `sale-dashboard-feature-management` (list, detail, CRUD, available menu keys)
- `sale-dashboard-package-management` (list, detail, CRUD, effective menus, UUID update)
- `sale-dashboard-addon-management` (list, detail, CRUD, UUID update — Phase 7)
- `sale-dashboard-company-features` (per-company feature list, toggle)
- `sale-dashboard-package-config-feature` (legacy slice, retained for `packageInfo`/`packageOptions` use cases)

### 2.6 Stripe UUID Sync Endpoints ⭐

> **Critical for rewrite:** endpoint เหล่านี้แก้ UUID ใน DB เพื่อให้ตรงกับ Stripe product ID — **ไม่ได้เรียก Stripe API**. Rewrite อาจขยาย scope นี้ได้

#### 2.6.1 Package — `PATCH /api/v2/sale-dashboard/packages/:packageUuid/identifier`

**Backend service flow** (`packageCrud.service.ts:322-368`):
1. Validate `newUuid !== currentUuid` → throw `PACKAGE_UUID_UNCHANGED`
2. Load existing package (404 ถ้าไม่เจอ → `PACKAGE_NOT_FOUND`)
3. Conflict check (rowed including archived) → throw `PACKAGE_UUID_CONFLICT`
4. **Atomic transaction (`CompPackages.transaction()`):**
   - `UPDATE comp_packages SET uuid=newUuid WHERE uuid=currentUuid`
   - `UPDATE comp_companies SET selected_package_uuid=newUuid WHERE selected_package_uuid=currentUuid` ⭐ (denormalized cache)
5. Re-fetch + return

**Why 2-table trx:** `comp_companies.selected_package_uuid` denormalized cache สำหรับ runtime lookup ใน Stripe webhook flow (ไม่ต้อง JOIN `comp_packages`)

**Frontend:**
- Component: `src/components/package-management/package-uuid-edit-dialog.tsx`
- Trigger: button "Sync Stripe ID" ใน detail view (`src/sections/package-management/package-detail-view.tsx`)
- onSuccess → `router.replace(paths.dashboard.packageManagement.detail(updated.uuid))` (URL slug = uuid)

#### 2.6.2 Addon — `PATCH /api/v2/sale-dashboard/addons/:addonUuid/identifier`

**Backend service flow** (`addon.service.ts:284-326`):
- 5-step pattern เดียวกับ Package
- ⚠️ **Single-table transaction** (เฉพาะ `comp_addons.uuid`) — เพราะ `comp_company_addons` ใช้ FK `addon_id` numeric อย่างเดียว ไม่มี denormalized `addon_uuid` cache

**Frontend:**
- Component: `src/components/addon-management/addon-uuid-edit-dialog.tsx`
- Trigger: button ใน addon detail + addon edit view (UUID display + sync button)
- onSuccess → `router.replace` ไปยัง URL ของ uuid ใหม่

#### 2.6.3 ที่ Rewrite อาจต้องขยายเพิ่ม

ทั้ง Package + Addon UUID sync endpoint ปัจจุบัน **แค่ update DB เพื่อ match Stripe product ID** — ไม่ได้ trigger Stripe API. ใน rewrite ควรพิจารณา:

- เพิ่ม flow ที่ trigger `stripe.products.update()` หลัง UUID update (sync 2-way)
- Webhook event `product.updated` → reverse sync `comp_packages.uuid` ตาม Stripe (optional)
- Audit log table ของ UUID change events

⚠️ **Scope clarification needed:** Stripe sync rewrite จะรวม Stripe API integration ด้วยไหม หรือแค่ migrate caller? (ดู §5 Open Questions)

---

## 3. Legacy Code & Schema Inventory (Reference Only)

> 🔑 **Strategy update 2026-05-07:** ส่วนนี้เป็น **reference inventory** — ไม่ใช่ "ต้อง migrate".
> Legacy callers ทั้งหมดในส่วนนี้ **ห้าม touch** ใน wave plan. รายละเอียดอยู่ที่นี่เพื่อ:
> - Wave A audit + verify integrity
> - Wave B-C dual-write helper ต้องเข้าใจ shape ของ legacy เพื่อ mirror ให้ถูก
> - Wave D ตัดสิน candidate ที่ verified zero caller
> - Wave E feature ใหม่ ๆ ใช้ schema ใหม่ ไม่อ้าง legacy


### 3.1 `const_packages` Table + Callers

#### 3.1.1 Schema เดิม
**Model:** `happywork-backend/src/database/postgresql/models/data/constPackages.model.ts`

| Column | Type | Note |
|--------|------|------|
| id | string (bigint) | PK |
| uuid | uuid | UQ — **same UUIDs ที่ M2 seed copy ไป comp_packages** |
| name | jsonb | `{th, en}` |
| billingCycle | string | varchar — old `billing_interval` semantics |
| numOfEmployees | int | → maps to `comp_packages.user_limit_min/max` |
| prices | jsonb | nested pricing rules |
| isRecommend, isContactUs | bool | |
| status_type, timestamps | | |

**Migration creating it:** `20250804-1627-const_packages.ts` (ก่อน feature-management — pre-existing)

#### 3.1.2 Callers (~22 references — V1 legacy code)

**Group 1: Payment & Subscription (HIGH priority)**

| File | Lines | Operation | Why uses ConstPackages |
|------|-------|-----------|----------------------|
| `src/modules/v1/admin/paymentProcessing/subscriptionUpgrade.service.ts` | 4, 231, 264, 271 | `.query().findById()`, `.where('uuid', ...)` | Stripe upgrade flow — get tier name, prices |
| `src/modules/v1/admin/paymentProcessing/paymentProcessing.repository.ts` | 10 | raw knex | Package lookup for payment intent |
| `src/modules/v1/admin/paymentProcessing/paymentProcessing.service.ts` | 7, 102, 192 | `.findById()`, `.where('uuid', ...)` | Pricing for quote/invoice |
| `src/modules/v1/admin/paymentProcessing/proration.service.ts` | 4, 64 | `.findById()` | Calculate prorated pricing on downgrade |
| `src/modules/v1/admin/subscription/subscription.repository.ts` | 3, 65, 79, 81, 157 | `.findById()`, LEFT JOIN | Subscription detail (package tier names) |
| `src/modules/v1/admin/trialExpiration/trialExpiration.repository.ts` | 30, 65-70 | LEFT JOIN | Trial job — billing cycle check |
| `src/modules/v1/admin/employees/employeeLimit.service.ts` | 2, 29 | `.findById()` | Validate employee count vs `numOfEmployees` |

**Group 2: Display Only (LOW priority — read-only)**

| File | Lines | Operation |
|------|-------|-----------|
| `src/modules/v1/admin/package/package.repository.ts` | 1, 4, 9, 14 | `getPackageById/ByUuid/All` (deprecated v1 admin CRUD) |
| `src/modules/v1/admin/package/package.adapter.ts` | 1, 27 | `normalizePackageResponse(ConstPackagesType)` |
| `src/modules/v1/sale-dashboard/customers/customers.repository.ts` | 7, 116, 192, 199, 215, 260 | LEFT JOIN ใน customer list |
| `src/modules/v1/sale-dashboard/leads/leads.repository.ts` | 7, 127, 185, 208, 321 | LEFT JOIN ใน leads list |
| `src/modules/v1/sale-dashboard/quotations/quotations.repository.ts` | 3, 134, 138 | `getPackageByUuid` for quotation package ref |

#### 3.1.3 Mapping Strategy

**const_packages → comp_packages mapping:**
- ✅ UUIDs match — `comp_packages.uuid` ตั้งใจคงเดิมจาก seed (M2)
- ✅ `comp_packages.user_limit_min/max` ⊃ `const_packages.numOfEmployees`
- ⚠️ `const_packages.prices` (jsonb nested) ≠ `comp_packages.price_amount` (single decimal) — **field shape ต่างกัน**

**Required helper utility:**
```typescript
// suggested: src/modules/v1/admin/package/legacy-bridge.service.ts
const constPackageToCompPackage = async (uuid: string): Promise<CompPackage>
```

> หรือถ้าปลอดภัยพอ → switch caller ไปอ่าน comp_packages โดยตรง (preferred)

### 3.2 `comp_companies.add_ons` JSONB + Callers

#### 3.2.1 Schema เดิม
**Model field:** `compCompanies.model.ts:55`
```typescript
addOns?: Array<{ uuid: string; quantity: number }>
```

**Type:** JSONB array, nullable, default undefined/[]
**ข้อจำกัด:**
- ไม่มี `purchased_at` หรือ `expires_at` — จึงไม่รองรับ subscription expiry semantics ที่ comp_company_addons ใช้
- ไม่มี audit trail (replace whole array on update)

#### 3.2.2 Callers (~17 — Stripe webhook + payment + display)

**Group 1: WRITE (Stripe webhook ที่ sync จาก Stripe → DB)**

| File | Lines | Operation | Context |
|------|-------|-----------|---------|
| `src/api/v2/webhooks/stripe-webhook.controller.ts` | 216-221, 316, 754-775 | **WRITE** add_ons array | `customer.subscription.created` + `customer.subscription.updated` events. Extract addon items จาก `subscription.items` (metadata.addonUuid + quantity) → build `[{uuid, quantity}]` → store ที่ `comp_companies.add_ons` |
| `src/api/v1/admin/payment/stripeWebhook.controller.ts` | (legacy v1) | **WRITE** | Same logic, v1 endpoint |
| `src/jobs/syncMaxUsersFromStripe.job.ts` | various | READ + WRITE | Daily cron — sync subscription quantity from Stripe |

**Group 2: READ (display/business logic)**

| File | Lines | Operation |
|------|-------|-----------|
| `src/modules/v2/sale-dashboard/sale-dashboard.service.ts` | 364 | READ — return company profile |
| Various v1/v2 payment + subscription controllers | | READ — display/billing |

> ⚠️ **List ของ caller ที่ส่งมาจาก audit อาจไม่ครบ ~17 ทั้งหมด** — ต้อง spawn deeper audit ก่อนเริ่ม Wave D (ดู §5 Open Questions Q3)

#### 3.2.3 Mapping Strategy: `add_ons` JSONB → `comp_company_addons` rows

**Read mapping (use Resolver):**
```typescript
// แทนที่: company.addOns?.find(a => a.uuid === addonUuid)?.quantity
// ด้วย:
const featureIds = await resolveCompanyEffectiveFeatureIdsService(companyId, cache);
// หรือถ้าต้องการ addon quantity เฉพาะ:
const addonRow = await CompCompanyAddons.query()
  .where({ companyId, addonId, statusType: 'active' })
  .where(qb => qb.whereNull('expiresAt').orWhere('expiresAt', '>', knex.fn.now()))
  .first();
```

**Write mapping (Stripe webhook):**
```typescript
// แทนที่: UPDATE comp_companies SET add_ons = '[{...}]' WHERE id = ?
// ด้วย:
await CompCompanyAddons.transaction(async (trx) => {
  // 1. Archive existing active addon rows for company
  await CompCompanyAddons.query(trx)
    .where({ companyId, statusType: 'active' })
    .patch({ statusType: 'archived', archivedBy: 'stripe-webhook', archivedAt: trx.fn.now() });
  
  // 2. Insert new active rows
  await CompCompanyAddons.query(trx).insert(
    addonItems.map(item => ({
      companyId,
      addonId: lookupAddonIdByUuid(item.uuid),
      quantity: item.quantity,
      purchasedAt: trx.fn.now(),
      expiresAt: subscription.current_period_end ? new Date(...) : null,
      statusType: 'active',
      createdBy: 'stripe-webhook',
    }))
  );
});
```

> ⚠️ **Decision needed (§5 Q4):** archive-then-insert pattern หรือ upsert pattern? Phase 3 เลือก archive-then-INSERT-always สำหรับ comp_company_features เพื่อ audit trail — น่าจะ apply pattern เดียวกัน

### 3.3 Stripe Webhook Flows ปัจจุบัน

#### 3.3.1 Webhook Handlers (v2)
**Path:** `happywork-backend/src/api/v2/webhooks/stripe-webhook.controller.ts`

**Events ที่ subscribe:**
- `customer.subscription.created` — new subscription
- `customer.subscription.updated` — modified (sync max_users + add_ons)
- `customer.subscription.deleted` — canceled
- `invoice.payment_succeeded` — payment processed
- `invoice.payment_failed`
- `customer.created` / `customer.updated`
- (อาจมีอีก ดู line 100+)

**Stripe SDK calls:**
```typescript
stripe.subscriptions.retrieve(subscriptionId, {
  expand: ['items.data.price.product']  // เพื่อได้ product metadata
})
stripe.customers.retrieve(customerId)
stripe.products.list()
stripe.prices.list()
```

**Data flow ของ subscription event:**
1. Stripe POST → endpoint
2. Validate signature (`src/middlewares/stripeWebhook.middleware.ts`)
3. Extract event type → handler
4. Lookup company by `stripeCustomerId`
5. Extract metadata:
   - `metadata.packageUuid` (comp_packages.uuid)
   - `metadata.addonUuid` + quantity (comp_addons.uuid + qty per item)
6. Update `comp_companies`:
   - `maxUsers` (from subscription quantity)
   - `add_ons` (JSONB) ⚠️ legacy field ที่จะ drop
   - `stripeSubscriptionId`, `selectedPackageUuid`, `billingInterval`
7. Log payment to `sale_dashboard_stripe_payments`

#### 3.3.2 Webhook v1 (legacy)
**Path:** `happywork-backend/src/api/v1/admin/payment/stripeWebhook.controller.ts`

มี logic คล้าย v2 — Rewrite ต้องตัดสิน:
- Migrate v1 → v2 (deprecate v1)
- หรือ keep v1 + update logic

#### 3.3.3 Cron Job
**Path:** `happywork-backend/src/jobs/syncMaxUsersFromStripe.job.ts`

Daily sync ที่:
- ดึง active subscriptions จาก Stripe
- ตรวจ quantity → update `comp_companies.maxUsers` + `add_ons`

⚠️ **ต้อง update job นี้ด้วย** ใน rewrite

### 3.4 Mapping Logic ที่มีอยู่

#### 3.4.1 const_packages → comp_packages (M8)
**Path:** `20260504-1007-alter-comp_companies-package-id-fk.ts`

ทำเสร็จแล้วใน M8 — `comp_companies.package_id` (FK numeric) ชี้ไปที่ `comp_packages.id` แล้ว. ดังนั้นสำหรับ `comp_packages` ไม่ต้อง backfill เพิ่ม

#### 3.4.2 add_ons JSONB → comp_company_addons (M7)
**Path:** `20260504-1006-create-comp_company_addons.ts:44-62`

ทำเสร็จแล้วใน M7 — `comp_company_addons` rows มี data จาก `add_ons` JSONB เดิมแล้ว

⚠️ **ปัญหา:** หลัง M7 deploy → Stripe webhook ยังเขียนใน `add_ons` JSONB → data drift จาก `comp_company_addons` (legacy backfill เก่า + JSONB อัพเดตใหม่ vs comp_company_addons rows เก่าที่ M7 backfill).

> 💡 **ก่อน rewrite caller — ต้อง re-run M7 backfill** เพื่อ catch up หรือเขียน script sync data ใหม่ (ดู §4.3 Migration Strategy)

#### 3.4.3 Stripe Product Metadata Format (ต้อง verify)
- Webhook code ใช้ `metadata.packageUuid` + `metadata.addonUuid` → **ใช้ UUID** (ไม่ใช่ id)
- ดังนั้น Stripe product registry ของลูกค้าใน production มี `metadata.packageUuid = comp_packages.uuid` (after M2 seed)
- **Risk:** ถ้า Stripe product ไหน metadata.packageUuid ยังเป็น UUID ของ const_packages เก่าที่ไม่ตรงกับ comp_packages → mapping จะ fail
- ⚠️ **ต้อง verify ใน production** ว่า Stripe products ทุกตัว metadata ถูกต้อง (ดู §5 Q1)

---

## 4. Plan สำหรับ Augmentation (Strategy: Additive, Non-Breaking)

> 🔑 **Core principle:** เพิ่มเลเยอร์ใหม่ + รักษาเลเยอร์เดิม. **ห้าม touch caller ของ `const_packages` หรือ `comp_companies.add_ons` ที่ทำงานอยู่ในปัจจุบัน** ยกเว้นจุดเดียว — Stripe webhook ที่จะเพิ่ม dual-write

### 4.1 Wave Breakdown

#### Wave A — Audit & Decision (1-2 วัน, LOW risk)

**Goals:**
1. Verify Stripe metadata format ใน production (sample 5-10 subscriptions)
2. Verify integrity ของ M7 backfill: `comp_company_addons` rows == sum of `comp_companies.add_ons` array length
3. Resolve §5 Open Questions (เฉพาะที่จำเป็นสำหรับ Wave B-E)
4. Define list of "dead code/table" candidates (Wave D pre-work)

**Tasks:**
- A.1 Sample 5-10 production Stripe subscriptions — verify `metadata.packageUuid` + `metadata.addonUuid` ตรงกับ `comp_packages.uuid` + `comp_addons.uuid`
- A.2 Run integrity queries (ดู §4.2.1 verification queries)
- A.3 Stakeholder lock decision: dual-write idempotency strategy (Q4) + which dead code to drop (Q5+Q6)
- A.4 Update plan + handoff doc

**Deliverable:** Decision lock — เริ่ม Wave B ได้

---

#### Wave B — Add Dual-Write Helper (3-5 วัน, MEDIUM risk) ⭐ Core Wave

**Goals:** Stripe webhook (v1+v2) + cron job — เก็บ `add_ons` JSONB write เดิม + เพิ่ม mirror write ลง `comp_company_addons` table

**Strategy:** **Additive** — เขียน helper service ที่ append-only ทำงานหลังจาก legacy write เสร็จ; ถ้า new write fail → log warning แต่ **ไม่ rollback** legacy (ห้าม break flow เดิม)

**Tasks:**

| ID | Task | File | Note |
|----|------|------|------|
| B.1 | สร้าง `src/modules/v2/billing/stripe-addon-sync.service.ts` | new | append-only mirror writer; archive-then-insert pattern |
| B.2 | เพิ่ม `syncCompanyAddonsFromStripeSubscription()` helper — รับ `companyId`, `subscription.items`, `trx?` | B.1 | input shape mirrors webhook payload |
| B.3 | Hook B.2 เข้า v2 webhook handler `customer.subscription.created` — **หลัง** legacy write | `api/v2/webhooks/stripe-webhook.controller.ts` | wrap in try/catch + Sentry log; ห้าม throw |
| B.4 | Hook เข้า `customer.subscription.updated` | same | |
| B.5 | Hook เข้า `customer.subscription.deleted` → archive ทุก active row ของ company | same | |
| B.6 | Hook เข้า v1 webhook (ถ้ายัง active — Q6) | `api/v1/admin/payment/stripeWebhook.controller.ts` | optional based on Q6 decision |
| B.7 | Hook เข้า cron `syncMaxUsersFromStripe.job.ts` | `jobs/syncMaxUsersFromStripe.job.ts` | append mirror after legacy update |
| B.8 | สร้าง idempotency layer — ใช้ `stripeEventId` ใน new helper เพื่อกัน duplicate ทุก retry | B.1 | UNIQUE constraint หรือ in-memory dedup |
| B.9 | Sandbox E2E test — Stripe test mode subscribe/update/cancel | tests/ | verify both `add_ons` JSONB + `comp_company_addons` row state |
| B.10 | Deploy with feature flag `ADDON_DUAL_WRITE_ENABLED=true` (default true; toggle off ถ้าฉุกเฉิน) | env | rollback safety |

**Critical safety rule:** ใน B.3-B.7 — ใหม่ helper **ไม่ throw**:
```typescript
try {
  await stripeAddonSyncService.sync(companyId, subscription, trx);
} catch (err) {
  logger.error('stripe-addon-sync mirror failed (legacy write succeeded)', { err, companyId, eventId });
  Sentry.captureException(err, { extra: { wave: 'B', flow: 'dual-write' } });
  // intentionally NOT rethrowing — legacy flow continues
}
```

---

#### Wave C — Reconciliation Job + Observability (2-3 วัน, LOW-MEDIUM risk)

**Goals:** ตรวจ drift ระหว่าง JSONB เดิม กับ table ใหม่ — alert ถ้าเจอ; auto-fix ถ้า safe

**Tasks:**

| ID | Task | File |
|----|------|------|
| C.1 | สร้าง cron `src/jobs/reconcileAddonsFromJsonb.job.ts` — daily | new |
| C.2 | Logic: compare `comp_companies.add_ons` JSONB array vs `comp_company_addons` active rows ต่อ company | C.1 |
| C.3 | Output: list of companies with drift + diff details (which addonUuid + qty mismatch) | C.1 |
| C.4 | Auto-fix mode (config flag): apply backfill SQL ของ M7 อีกรอบสำหรับ company ที่ drift | C.1 |
| C.5 | Alert mode: send Slack/Sentry ถ้า drift > threshold | C.1 |
| C.6 | Dashboard panel (Grafana/Sentry) — count drift events ต่อวัน | infra |
| C.7 | Backfill ครั้งแรกหลัง Wave B deploy: re-run M7 backfill SQL manually เพื่อ catch up data ที่ JSONB write หลัง M7 ยังไม่ mirror (ก่อน Wave B) | manual |

> 💡 **Wave C มีอยู่เพื่อ confidence** — ถ้า dual-write ใน Wave B reliable → drift = 0 ตลอด → Wave C job แค่ idle confirming

---

#### Wave D — Drop Verified-Unused Code/Table (1-2 วัน, LOW risk)

**Goals:** กวาดของที่ไม่ใช้แล้วออก — เฉพาะที่ verify zero caller

**Pre-flight (ต้องผ่านก่อน drop):**
```bash
# 1. ConstPackages model ต้องไม่มี caller (ทุกอันยัง active — ดังนั้น D ไม่ touch ConstPackages)
grep -r "ConstPackages\|const_packages" happywork-backend/src/ | grep -v "\.spec\.ts"
# Expected for Wave D: > 0 (legacy ยังใช้อยู่ — keep)

# 2. SaleDashboardDisabledFeatures — ตอนนี้ zero caller จาก Phase 6 W6.1
# Phase 6 W6.1 ลบไปแล้ว → ✅ confirmed

# 3. featureDefinitions.ts — Phase 6 W6.1 ลบแล้ว → ✅ confirmed
```

**Candidates (ตามที่ user คอนเฟิร์ม):**

| ID | Candidate | Status | Action |
|----|-----------|--------|--------|
| D.1 | `sale_dashboard_disabled_features` table | ✅ already dropped (Phase 6 W6.1 M11 — done) | already done |
| D.2 | `src/modules/v1/featureManagement/` | ✅ already deleted (Phase 6 W6.1) | already done |
| D.3 | `src/constant/featureDefinitions.ts` | ✅ already deleted (Phase 6 W6.1) | already done |
| D.4 | `src/modules/v1/admin/package/` (legacy admin CRUD) | ⚠️ verify | grep before drop |
| D.5 | Other dead code identified in Wave A.4 | TBD | TBD |

**Tasks:**
- D.1 ตรวจสอบ caller ของ candidate D.4 (`src/modules/v1/admin/package/*`) — ถ้า zero caller (no one ref `getPackageById/ByUuid/getPackages` จาก v1 admin) → drop
- D.2 Drop file/folder + run TS check
- D.3 ⚠️ **อย่า drop `const_packages` table** — ยังมี active caller
- D.4 ⚠️ **อย่า drop `comp_companies.add_ons` column** — ยังมี active caller

> 💡 Wave D เปิดประตูสำหรับการ drop ของในอนาคตเมื่อ caller ทยอยลดลง — ไม่ใช่ "drop ทันที"

---

#### Wave E — New Feature Flows on New Schema (LOW risk; per-feature)

**Goals:** Feature ใหม่ที่จะเพิ่มในอนาคต — ใช้ Resolver + comp_packages + comp_company_addons ตรง ๆ ไม่ผ่าน legacy schema

**Pattern reference (build by example):**

| New Feature Pattern | What to use |
|---------------------|-------------|
| Get effective addon quantity for company | `CompCompanyAddons.query().where({companyId, statusType: 'active'})` + expiry filter |
| Get effective package tier for company | `resolveEffectivePackageIdService(companyId, cache)` → comp_packages |
| Check feature gate (e.g. "company has e_signature feature?") | `resolveCompanyEffectiveFeatureIdsService(companyId, cache)` |
| Validate menu access in API | `resolveUserEffectivePermissionService(userUuid, cache)` (Phase 4) |

**Anti-patterns ที่ห้ามใช้ใน flow ใหม่:**
- ❌ `ConstPackages.query()` — ของเก่า, อย่าใช้
- ❌ `company.addOns?.find(...)` — ของเก่า, อ่านจาก `comp_company_addons` แทน
- ❌ Raw query `WHERE add_ons @> '...'` — ใช้ table query แทน

**Deliverables (per feature):**
- Service ใหม่ใช้ pattern ข้างบน
- Unit test stub `comp_company_addons` + `comp_packages` (ไม่ใช่ const_packages)
- Document feature ใน existing module's README

> 💡 Wave E ไม่มี timeline แน่นอน — เริ่มต่อเมื่อมี requirement feature ใหม่. Plan นี้ define "rules of the road" สำหรับ feature ใหม่ทั้งหมด

### 4.2 Risks & Mitigations

> 🔑 **Risk profile ลดลงมากเทียบกับ rewrite version** — ไม่ touch caller เก่า → ไม่มี regression risk บน flow ที่ทำงานอยู่

#### Risk 1: Stripe Metadata Mismatch (LOW-MEDIUM)
**ปัญหา:** Stripe product ใน production บางตัวอาจมี `metadata.packageUuid`/`addonUuid` ที่ไม่ตรงกับ `comp_packages.uuid`/`comp_addons.uuid`
**Impact ต่อ Wave B:** new helper จะ log warning + skip (ไม่ใส่ row ใน comp_company_addons) — flow เดิมยังทำงาน
**Mitigation:**
- Wave A.1: audit production sample
- Wave B.1: helper logs detailed mismatch (productId, metadata, expected uuid)
- Wave C: reconciliation job catches patterns + creates fix queue

#### Risk 2: Dual-Write Helper Crashes Webhook (LOW)
**ปัญหา:** ถ้า new helper throw exception → webhook return 500 → Stripe retry → potential duplicate writes
**Mitigation:**
- ✅ **Strict rule: helper ไม่ throw** (B.3 critical safety rule) — wrap try/catch แล้ว swallow error
- ✅ Idempotency key (B.8) — ใช้ stripeEventId กัน duplicate
- Monitor: Sentry alert ถ้า dual-write fail rate > 0.1%

#### Risk 3: Drift Between JSONB and Table (MEDIUM)
**ปัญหา:** Race condition หรือ helper bug → JSONB กับ table desynchronized
**Mitigation:**
- Wave C reconciliation job — daily compare + alert
- Auto-fix mode (C.4): re-run M7 backfill SQL targeted ที่ company ที่ drift
- Manual escalation ถ้า drift > X% (threshold from Wave A.4)

#### Risk 4: Feature Flag Toggle Edge Case (LOW)
**ปัญหา:** ถ้า flip `ADDON_DUAL_WRITE_ENABLED=false` กลางอากาศ → new code path skip → `comp_company_addons` ขาด data ของช่วงที่ off
**Mitigation:**
- Wave C reconciliation จะ catch + auto-fix
- Document toggle procedure: ก่อน flip on/off → check Wave C dashboard

#### Risk 5: Stripe API Outage During Reconciliation (LOW)
**ปัญหา:** Wave C reconciliation อาจต้อง fetch Stripe subscription data → ถ้า Stripe down → job fail
**Mitigation:**
- Reconciliation ใช้ DB only (compare JSONB vs table — ไม่ต้องเรียก Stripe API)
- ถ้า future spec ต้อง verify against Stripe → exponential backoff + circuit breaker

### 4.3 Migration Strategy

#### 4.3.1 Lifecycle

```
[T0]   Wave A audit → decisions locked (1-2 วัน)
       ↓
[T+1w] Wave B deploy (dual-write enabled, behind flag) — flow เก่ายังเหมือนเดิม
       ↓
[T+2w] Wave C deploy (reconciliation job + dashboards) — confidence building
       ↓
[T+3w] Verify dashboard 1 sprint — drift = 0 ตลอด → confidence ✅
       ↓
[T+4w] Wave D drop verified-unused (low-risk cleanup)
       ↓
[T+∞]  Wave E ทำเมื่อมี new feature requirement
```

#### 4.3.2 Rollback Plan
- **Wave B:** flag `ADDON_DUAL_WRITE_ENABLED=false` → new helper short-circuits → flow กลับไปเหมือน pre-Wave B ทันที
- **Wave C:** disable cron (no impact on data; just observability)
- **Wave D:** drop migration มี down() — restore table/folder ได้ (last-resort)
- **Wave E:** revert per-feature commit

> 💡 **No emergency rollback for legacy flow** — เพราะ Wave B-D ห้าม touch legacy

#### 4.3.3 Data Reconciliation Strategy

**Initial backfill (one-time, after Wave B deploy):**
```sql
-- Re-run M7 backfill targeted ที่ company ที่ JSONB เปลี่ยนหลัง M7 deploy
-- (เพราะ JSONB writes หลัง M7 ก่อน Wave B ยังไม่ mirror)
INSERT INTO comp_company_addons (company_id, addon_id, quantity, purchased_at, status_type, ...)
SELECT cc.id, ca.id, COALESCE((addon_item->>'quantity')::int, 1), cc.updated_at, 'active', ...
FROM comp_companies cc
CROSS JOIN LATERAL jsonb_array_elements(cc.add_ons) AS addon_item
JOIN comp_addons ca ON ca.uuid = (addon_item->>'uuid')::uuid
LEFT JOIN comp_company_addons cca
  ON cca.company_id = cc.id AND cca.addon_id = ca.id AND cca.status_type = 'active'
WHERE cc.add_ons IS NOT NULL
  AND jsonb_array_length(cc.add_ons) > 0
  AND cca.id IS NULL  -- only insert ถ้ายังไม่มี
ON CONFLICT DO NOTHING;
```

**Continuous reconciliation (Wave C):** daily — alert + auto-fix

### 4.4 Test Strategy

#### 4.4.1 Unit Test Coverage
- B.1 helper: stub trx + verify SQL pattern (archive-then-insert)
- B.8 idempotency: 2 calls with same eventId → 1 row only
- C.1 reconciliation: synthetic drift cases → expected diff output

#### 4.4.2 Integration Test
- Stripe webhook simulator (sandbox events)
- Coverage:
  - subscription.created with addons → verify both JSONB + table written
  - subscription.updated (add/remove/qty change) → verify mirror correct
  - subscription.deleted → verify all rows archived
  - Edge: helper throws → verify legacy still succeeds (no rollback)
  - Edge: feature flag off → verify no new writes (legacy only)

#### 4.4.3 E2E Test (Playwright + Stripe sandbox)
- Subscribe new company via Stripe checkout → verify both data paths consistent
- Upgrade tier → verify
- Add addon → verify
- Cancel → verify

#### 4.4.4 Performance Test
- B.1 helper overhead: webhook latency before vs after dual-write — must be <50ms additional
- C.1 reconciliation job runtime: must complete <5min on production-size DB

---

## 5. Open Questions ที่ต้อง Resolve ก่อนเริ่ม

> 🔑 Strategy update 2026-05-07 — questions ที่เกี่ยวกับ refactor caller เก่าถูก **drop** ไปแล้ว (เพราะ strategy ใหม่ห้าม touch caller)

| # | Question | Owner | Blocking Wave |
|---|----------|-------|---------------|
| Q1 | Stripe production: ทุก product มี `metadata.packageUuid` + `metadata.addonUuid` ที่ตรงกับ `comp_packages.uuid` + `comp_addons.uuid` ใช่ไหม? ถ้าไม่ — strategy fallback คืออะไร? | DevOps + Stripe admin | A → blocks B |
| Q4 | Write strategy ของ `comp_company_addons` ใน dual-write helper: archive-then-INSERT-always (Phase 3 W4 pattern) vs UPSERT (revive archived row)? | Backend lead | A → blocks B.1 |
| Q6 | v1 Stripe webhook (`api/v1/admin/payment/stripeWebhook.controller.ts`) มี traffic active ใน production ไหม? — ถ้ามี → ต้อง mirror dual-write ที่ v1 webhook ด้วย (B.6) | DevOps | A → blocks B.6 decision |
| Q11 | Wave C auto-fix mode default on/off? — auto-fix อาจ mask bug ใน Wave B helper | Backend lead | C |
| Q12 | Wave D candidate list: ต้องการ drop `src/modules/v1/admin/package/` (legacy admin CRUD) ไหม? — verify zero caller ก่อน | Backend lead | D |
| Q13 | Reconciliation drift threshold: alert ที่ % เท่าไร? (e.g. > 1% companies drifted) | Backend lead + Ops | C.5 |
| Q14 | Idempotency layer (B.8): ใช้ `stripeEventId` UNIQUE ใน comp_company_addons (extra column) หรือ in-memory Redis dedup หรือ separate table? | Backend lead | B.8 |

**Questions ที่ถูก drop จาก strategy เก่า (สำหรับ traceability):**
- ~~Q2 Stripe API integration scope~~ — out of scope (ไม่ touch Stripe SDK calls)
- ~~Q3 add_ons JSONB caller exact count~~ — ไม่ refactor caller
- ~~Q5 v1 admin /package deprecate or migrate~~ — Wave D candidate, ไม่ blocking
- ~~Q7 dual-write transition period~~ — strategy ใหม่ dual-write ตลอด (no cut-over)
- ~~Q8 const_packages.prices shape~~ — ไม่ touch const_packages caller
- ~~Q9 package_id vs selected_package_uuid canonical~~ — ไม่ refactor caller, ทั้งคู่ยังใช้
- ~~Q10 reconciliation job placement~~ — ตัดสินแล้ว (new file `reconcileAddonsFromJsonb.job.ts`)

---

## 6. References (File Paths)

### 6.1 Foundation Code (Read-only — ใช้ pattern จากตรงนี้)

**Backend (`happywork-backend/`):**

| Purpose | Path |
|---------|------|
| comp_packages model | `src/database/postgresql/models/data/compPackages.model.ts` |
| comp_addons model | `src/database/postgresql/models/data/compAddons.model.ts` |
| comp_features model | `src/database/postgresql/models/data/compFeatures.model.ts` |
| comp_company_features model | `src/database/postgresql/models/data/compCompanyFeatures.model.ts` |
| comp_company_addons model | `src/database/postgresql/models/data/compCompanyAddons.model.ts` |
| Migrations dir | `src/database/postgresql/migrations/data/` |
| Permission Resolver | `src/modules/v2/sale-dashboard/permissionResolver/` |
| Package CRUD module | `src/modules/v2/sale-dashboard/packageCrud/` |
| Addon CRUD module | `src/modules/v2/sale-dashboard/addon/` |
| Feature CRUD module | `src/modules/v2/sale-dashboard/feature/` |
| Company Feature toggle | `src/modules/v2/sale-dashboard/companyFeature/` |
| Sale Dashboard routes | `src/api/v2/sale-dashboard/sale-dashboard.routes.ts` |
| External auth permissions | `src/api/external/auth/permissions.controller.ts` |
| Admin compPermission default (filtered) | `src/api/v1/admin/compPermission/compPermission.controller.ts` |
| `requireSuperAdmin` middleware | `src/middlewares/requireSuperAdmin.middleware.ts` |
| `validateSaleDashboardAccessToken` | `src/middlewares/validateSaleDashboardAccessToken.middleware.ts` |

**Frontend (`happywork-sale-cms/`):**

| Purpose | Path |
|---------|------|
| Package detail view | `src/sections/package-management/package-detail-view.tsx` |
| Package UUID dialog | `src/components/package-management/package-uuid-edit-dialog.tsx` |
| Addon detail view | `src/sections/addon-management/addon-detail-view.tsx` |
| Addon UUID dialog | `src/components/addon-management/addon-uuid-edit-dialog.tsx` |
| Feature detail view | `src/sections/feature-management/feature-detail-view.tsx` |
| Per-company feature tab | `src/sections/client-management/feature-tab/company-features-tab-view.tsx` |
| Redux slices | `src/store/{actions,reducers,sagas,services}/sale-dashboard-*.ts` |
| URL builders | `src/store/services/sale-dashboard.ts` |

### 6.2 Legacy Code (Target ของ Rewrite)

**Backend (`happywork-backend/`):**

| Purpose | Path | Status |
|---------|------|--------|
| ConstPackages model | `src/database/postgresql/models/data/constPackages.model.ts` | DELETE in Wave E |
| const_packages migration (create) | `src/database/postgresql/migrations/data/20250804-1627-const_packages.ts` | preserved (rollback only) |
| Stripe webhook v2 | `src/api/v2/webhooks/stripe-webhook.controller.ts` | REWRITE in Wave D |
| Stripe webhook v1 | `src/api/v1/admin/payment/stripeWebhook.controller.ts` | DEPRECATE? (Q6) |
| paymentProcessing module | `src/modules/v1/admin/paymentProcessing/` | REWRITE in Wave B |
| subscription module | `src/modules/v1/admin/subscription/` | REWRITE in Wave B |
| trialExpiration module | `src/modules/v1/admin/trialExpiration/` | REWRITE in Wave B |
| employees/employeeLimit | `src/modules/v1/admin/employees/employeeLimit.service.ts` | REWRITE in Wave B |
| v1 admin/package | `src/modules/v1/admin/package/` | REWRITE or DEPRECATE (Q5) |
| sale-dashboard customers/leads/quotations | `src/modules/v1/sale-dashboard/{customers,leads,quotations}/` | REWRITE in Wave B |
| syncMaxUsersFromStripe job | `src/jobs/syncMaxUsersFromStripe.job.ts` | REWRITE in Wave D |
| Stripe SDK wrapper | `src/modules/v1/admin/paymentProcessing/stripe.service.ts` | review |

### 6.3 Memory + Doc References

| Purpose | Path |
|---------|------|
| Workspace AI rules | `/Users/ball/Desktop/ai-space/ai-workspace/.ai-memory/rules.md` |
| Active feature memory | `/Users/ball/Desktop/ai-space/ai-workspace/.ai-memory/current.md` |
| Project task list | `/Users/ball/Desktop/ai-space/ai-workspace/.ai-memory/task-list.md` |
| feature-management requirement docs | `/Users/ball/Desktop/ai-space/ai-workspace/document/requirement/feature-management/` |
| feature-management Phase 3 CR notes | `document/requirement/feature-management/cr-phase3.md` |
| feature-management Phase 4 BA notes | `document/requirement/feature-management/ba-validation-phase4.md` |
| feature-management Phase 5 CR notes | `document/requirement/feature-management/cr-phase5.md` |
| **This handoff** | `document/requirement/stripe-sync-rewrite/handoff.md` |

### 6.4 Git State (as of 2026-05-07)

**hw-backend:** branch `ball/feature/feature-management` HEAD = `8c8f608e` (12 commits ahead of `main`)
**hw-sale-cms:** branch `ball/feature/feature-management` HEAD = `829ffb0` (15 commits ahead of `main`)

**Status:** Working tree clean ทั้งคู่. ✅ M9 + M10 + M11 รันบน dev DB แล้ว (2026-05-07) — พร้อม start Wave A

---

## 7. Appendix — JSONB Shape Specs

### 7.1 `comp_companies.add_ons` JSONB (legacy)

**Shape:**
```typescript
type CompanyAddOnsLegacy = Array<{
  uuid: string;       // → matches comp_addons.uuid (post-M4 seed)
  quantity: number;   // >= 1
}>;
```

**Storage:** `jsonb` array
**Nullability:** column nullable; `NULL` interpreted as empty array
**Created/updated by:** Stripe webhook (v1+v2) + manual admin edits (legacy paths)

**Example values found:**
```json
// Empty case:
null

// Single addon:
[{"uuid": "abc-...", "quantity": 1}]

// Multiple addons with quantities:
[
  {"uuid": "abc-123", "quantity": 1},
  {"uuid": "def-456", "quantity": 5}
]
```

**Constraints (informal):**
- ⚠️ ไม่มี `purchased_at` / `expires_at` — subscription expiry semantics ไม่ available
- ⚠️ ไม่มี audit trail (replace whole array)
- ⚠️ ไม่มี FK enforcement — อาจมี orphan UUID ถ้า addon ถูกลบจาก `comp_addons`

### 7.2 `comp_company_addons` Table (new equivalent)

```typescript
type CompCompanyAddonsRow = {
  id: bigint;
  uuid: string;
  companyId: bigint;
  addonId: bigint;             // FK → comp_addons.id
  quantity: number;            // CHECK (quantity >= 1)
  purchasedAt: Date;
  expiresAt: Date | null;       // NULL = never expires
  statusType: 'active' | 'archived';
  // audit columns: created_at, updated_at, archived_at, created_by, ...
};
```

**Equivalence rule (Resolver):**
```typescript
// Old:
const addonsForCompany = company.addOns?.filter(a => a.uuid && a.quantity > 0) ?? [];

// New (equivalent set, with expiry filter):
const addonsForCompany = await CompCompanyAddons.query()
  .where({ companyId: company.id, statusType: 'active' })
  .where(qb => qb.whereNull('expiresAt').orWhere('expiresAt', '>', knex.fn.now()));
// or via resolver if just need feature impact:
const featureIds = await resolveCompanyEffectiveFeatureIdsService(company.id, cache);
```

### 7.3 Stripe Subscription Item Metadata (expected format)

ตามที่ webhook code handle:
```typescript
type StripeSubscriptionItemMetadata = {
  // For package items:
  packageUuid?: string;   // = comp_packages.uuid

  // For addon items:
  addonUuid?: string;     // = comp_addons.uuid
  // quantity comes from subscription_item.quantity (Stripe SDK field)
};
```

**Requirements ของ Stripe product setup:**
- ทุก Stripe product ที่เป็น "package" → metadata.packageUuid = comp_packages.uuid
- ทุก Stripe product ที่เป็น "addon" → metadata.addonUuid = comp_addons.uuid
- ⚠️ ไม่ใช่ทุก subscription item จะมี metadata — webhook fallback strategy (Q1)

### 7.4 ConstPackages.prices JSONB (ต้อง verify)

**Shape (เดา จาก code usage):**
```typescript
type ConstPackagePrices = {
  // possibly:
  thb?: number;
  usd?: number;
  // or:
  monthly?: number;
  yearly?: number;
  // หรือเป็น nested:
  regular?: { thb: number };
  discount?: { thb: number };
};
```

⚠️ **Wave A.2 ต้อง query distinct shapes ใน production** เพื่อ confirm

```sql
SELECT DISTINCT prices FROM const_packages WHERE prices IS NOT NULL LIMIT 50;
SELECT jsonb_object_keys(prices) AS top_keys, COUNT(*) AS row_count
FROM const_packages WHERE prices IS NOT NULL
GROUP BY top_keys;
```

---

## 📝 Note: How to Use This Document

1. **เริ่ม session ใหม่:** อ่าน Section 1 + Section 5 (Open Questions) ก่อน — สรุปสถานะ + scope
2. **Implement wave ใด ๆ:** อ่าน Section 4 (wave detail) + Section 3 (legacy detail) + Section 2 (foundation patterns to mirror)
3. **เจอปัญหา:** check Section 6 (file paths) → grep code จริง — ห้ามอ้างอิงจาก doc โดยไม่ verify
4. **เพิ่ม insight:** อัพเดต doc นี้พร้อม PR — keep single source of truth

**Last update:**
- 2026-05-07 (initial handoff after feature-management Phase 1-7 closeout)
- 2026-05-07 (strategy revision per user decision — Additive/Non-Breaking instead of full Rewrite; preserve all legacy callers, add dual-write only at Stripe webhook, drop verified-unused only)
