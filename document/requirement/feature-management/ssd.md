---
feature: feature-management
type: system-specification-document
created: 2026-05-04
updated: 2026-05-04
owner: SA
status: draft
---

# System Specification Document — Feature Management

> มุมมองทาง technical — ตอบคำถาม "ระบบต้องทำอะไร, ทำยังไง, มี API/DB/component อะไรบ้าง"

---

## 1. Architecture Overview

```
┌─────────────────────┐         ┌─────────────────────┐
│ happywork-sale-cms  │         │ happywork-app       │
│ (Super Admin CMS)   │         │ (End user app)      │
│                     │         │                     │
│ - Feature CRUD UI   │         │ - Renders menus     │
│ - Package CRUD UI   │         │   from /permissions │
│ - Addon CRUD UI     │         │                     │
│ - Company features  │         │                     │
│   tab               │         │                     │
└─────────┬───────────┘         └─────────┬───────────┘
          │                               │
          │ HTTPS (v2 admin)              │ HTTPS (external/auth)
          │                               │
          ▼                               ▼
┌──────────────────────────────────────────────────────┐
│                  happywork-backend                   │
│                                                      │
│  ┌─────────────────────────────────────────────┐    │
│  │ /api/v2/{features,packages,addons,    │    │
│  │   companies/:uuid/features}                 │    │
│  │                                             │    │
│  │  Controllers → Services → Repositories      │    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
│  ┌─────────────────────────────────────────────┐    │
│  │ permissionResolver (shared service)         │    │
│  │                                             │    │
│  │  resolveCompanyEffectiveMenuKeysService()   │    │
│  │  filterPermissionByMenuKeysService()        │    │
│  │  resolveUserEffectivePermissionService()    │    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
│  ┌─────────────────────────────────────────────┐    │
│  │ Existing /api/external/auth/v1/permissions  │    │
│  │   → wraps with resolver in Phase 4          │    │
│  └─────────────────────────────────────────────┘    │
└──────────────────────────┬───────────────────────────┘
                           │ Knex / Objection.js
                           ▼
        ┌──────────────────────────────────────┐
        │            PostgreSQL                │
        │                                      │
        │  comp_features                       │
        │  comp_packages                       │
        │  comp_package_features               │
        │  comp_addons                         │
        │  comp_addon_features                 │
        │  comp_company_features (override)    │
        │  comp_company_addons                 │
        │                                      │
        │  comp_companies (FK migrated)        │
        │  comp_permission (existing)          │
        │  comp_permission_mapping (existing)  │
        └──────────────────────────────────────┘
```

## 2. Affected Modules / Services

| Module / Service | Path | Type of change | Note |
|------------------|------|----------------|------|
| `compFeatures` model | `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts` | new | Objection.js model |
| `compPackages` model | `compPackages.model.ts` | new | |
| `compPackageFeatures` model | `compPackageFeatures.model.ts` | new | Link table |
| `compAddons` model | `compAddons.model.ts` | new | |
| `compAddonFeatures` model | `compAddonFeatures.model.ts` | new | Link table |
| `compCompanyFeatures` model | `compCompanyFeatures.model.ts` | new | Per-company override |
| `compCompanyAddons` model | `compCompanyAddons.model.ts` | new | |
| `compCompanies` model | `compCompanies.model.ts` | modify | FK relation update |
| `feature` module | `src/modules/v2/admin/feature/` | new | CRUD master |
| `feature` routes | `src/api/v2/feature/` | new | |
| `packageCrud` module | `src/modules/v2/admin/packageCrud/` | new | CRUD master (different from existing v2/admin/package which is Stripe sync) |
| `packageCrud` routes | `src/api/v2/packageCrud/` | new | |
| `addon` module | `src/modules/v2/admin/addon/` | new | CRUD master |
| `addon` routes | `src/api/v2/addon/` | new | |
| `companyFeature` module | `src/modules/v2/admin/companyFeature/` | new | Per-company override |
| `companyFeature` routes | `src/api/v2/companyFeature/` | new | |
| `permissionResolver` module | `src/modules/v2/admin/permissionResolver/` | new | Shared resolver (used by Phase 4 callers) |
| `requireSuperAdmin` middleware | `src/middlewares/requireSuperAdmin.middleware.ts` | new (if not exists) | RBAC for super_admin only |
| `compPermission.ts` constant | `src/constant/compPermission.ts` | modify | Add `getAvailableMenuKeys()` helper |
| `permissions.controller.ts` | `src/api/external/auth/permissions.controller.ts` | modify (Phase 4) | Wrap with resolver |
| `permissions.service.ts` | `src/modules/v1/externalAuth/permissions.service.ts` | modify (Phase 4) | |
| `compPermission` admin module | `src/modules/v1/admin/compPermission/*` | modify (Phase 4) | List filter |
| `compPermissionMapping` admin module | `src/modules/v1/admin/compPermissionMapping/*` | modify (Phase 4) | Save validation |
| Frontend: feature-management section | `happywork-sale-cms/src/sections/feature-management/` | new | List/Create/Detail/Edit views + components |
| Frontend: package-management section | `src/sections/package-management/` | new | |
| Frontend: addon-management section | `src/sections/addon-management/` | new | |
| Frontend: client-management feature-tab | `src/sections/client-management/feature-tab/` | new | Tab in existing client detail |
| Frontend: app routes | `src/app/dashboard/{feature,package,addon}-management/` | new | Next.js App Router pages |
| Frontend: shared components | `src/components/feature-management/` | new | Reusable: menu-keys-select, multilingual-text-field, feature-source-badge |
| Frontend: Redux store | `src/store/{actions,reducers,sagas,services}/sale-dashboard-{feature,package,addon}-management.ts` + `sale-dashboard-company-features.ts` | new | 4 new slices |
| Frontend: types | `src/types/store/sale-dashboard-{feature,package,addon}-management.ts` + `sale-dashboard-company-features.ts` | new | |
| Frontend: rbac | `src/utils/rbac.ts` | modify | Add resources |
| Frontend: navigation | `src/layouts/dashboard/config-navigation.tsx` | modify | Add Master Data section |
| Frontend: paths | `src/routes/paths.ts` | modify | Add path constants |
| Frontend: i18n | `src/locales/langs/{en,th}.json` | modify | Add translation keys |

## 3. Data Model

### 3.1 New Tables / Collections

```sql
-- 3.1.1 comp_features (master)
CREATE TABLE comp_features (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,                -- via tableTemplate
  uuid UUID NOT NULL DEFAULT uuid_generate_v4() UNIQUE,
  status_type VARCHAR(25) NOT NULL DEFAULT 'active',
  -- audit fields from tableTemplate
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  archived_at TIMESTAMPTZ NULL,
  created_by VARCHAR(50) NULL,
  updated_by VARCHAR(50) NULL,
  archived_by VARCHAR(50) NULL,
  -- domain fields
  code VARCHAR(50) NOT NULL UNIQUE,
  name JSONB NOT NULL,                                  -- {th, en}
  description JSONB NOT NULL DEFAULT '{}',
  menu_keys JSONB NOT NULL DEFAULT '[]',                -- string[] of leaf paths
  sort_order INT NOT NULL DEFAULT 0
);
CREATE INDEX idx_comp_features_status_type ON comp_features (status_type);
CREATE INDEX idx_comp_features_name_gin ON comp_features USING GIN (name);

-- 3.1.2 comp_packages (master, replaces const_packages logically)
-- Note: 7 seed records (Seed + LITE/CORE/PRO × 2 billing intervals).
-- UNIQUE constraint = composite (code, billing_interval) เพราะ packageMasterData reuse code 'lite'/'core'/'pro' ระหว่าง month + year
CREATE TABLE comp_packages (
  -- tableTemplate fields (id, uuid, audit, status_type)
  code VARCHAR(50) NOT NULL,                            -- 'seed','lite','core','pro' (เป็น tier; reused across billing intervals)
  name JSONB NOT NULL,
  short_name VARCHAR(50),
  description JSONB DEFAULT '{}',
  billing_interval VARCHAR(20) NULL,                    -- 'month'|'year'|NULL
  price_amount DECIMAL(12,2) DEFAULT 0,
  currency VARCHAR(3) DEFAULT 'thb',
  user_limit_min INT NULL,
  user_limit_max INT NULL,
  is_recommend BOOLEAN DEFAULT false,
  is_contact_us BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  stripe_product_id VARCHAR(255) NULL,
  stripe_price_id VARCHAR(255) NULL,
  sort_order INT DEFAULT 0
);
CREATE INDEX idx_comp_packages_status_type ON comp_packages (status_type);
CREATE INDEX idx_comp_packages_is_active ON comp_packages (is_active);
CREATE INDEX idx_comp_packages_name_gin ON comp_packages USING GIN (name);
CREATE UNIQUE INDEX uq_comp_packages_code_billing_interval ON comp_packages (code, billing_interval);

-- 3.1.3 comp_package_features (N:M)
CREATE TABLE comp_package_features (
  package_id BIGINT NOT NULL REFERENCES comp_packages(id) ON DELETE CASCADE,
  feature_id BIGINT NOT NULL REFERENCES comp_features(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  created_by VARCHAR(50),
  PRIMARY KEY (package_id, feature_id)
);
CREATE INDEX idx_comp_package_features_feature_id ON comp_package_features (feature_id);

-- 3.1.4 comp_addons (master)
-- Note: 9 seed records (HappyBeacon + E_SLIP/EVALUATION/HAPPY_PM/E_SIGNATURE × 2 billing intervals).
-- UNIQUE constraint = composite (code, billing_interval) เพราะ addonsMasterData reuse code ระหว่าง month + year
CREATE TABLE comp_addons (
  -- tableTemplate fields
  code VARCHAR(50) NOT NULL,                            -- PackageAddonsEnumCode value (reused across billing intervals)
  name JSONB NOT NULL,
  short_name VARCHAR(50),
  description JSONB DEFAULT '{}',
  billing_interval VARCHAR(20) NULL,                    -- 'month'|'year'|NULL (one-time)
  price_amount DECIMAL(12,2) DEFAULT 0,
  currency VARCHAR(3) DEFAULT 'thb',
  is_quantifiable BOOLEAN DEFAULT false,                -- e.g., HappyBeacon=true
  max_quantity INT NULL,                                -- required if is_quantifiable=true
  is_recommend BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  stripe_product_id VARCHAR(255) NULL,
  stripe_price_id VARCHAR(255) NULL,
  sort_order INT DEFAULT 0
);
CREATE INDEX idx_comp_addons_status_type ON comp_addons (status_type);
CREATE INDEX idx_comp_addons_is_active ON comp_addons (is_active);
CREATE INDEX idx_comp_addons_name_gin ON comp_addons USING GIN (name);
CREATE UNIQUE INDEX uq_comp_addons_code_billing_interval ON comp_addons (code, billing_interval);

-- 3.1.5 comp_addon_features (N:M)
CREATE TABLE comp_addon_features (
  addon_id BIGINT NOT NULL REFERENCES comp_addons(id) ON DELETE CASCADE,
  feature_id BIGINT NOT NULL REFERENCES comp_features(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (addon_id, feature_id)
);
CREATE INDEX idx_comp_addon_features_feature_id ON comp_addon_features (feature_id);

-- 3.1.6 comp_company_features (DELTA override per company)
-- Q4=B: feature_id FK ใช้ RESTRICT (ไม่ใช่ CASCADE) เพื่อ keep audit trail
-- soft-delete feature → row override ยังอยู่ → Resolver filter status_type='active' เอง
CREATE TABLE comp_company_features (
  -- tableTemplate fields (id, uuid, audit, status_type)
  company_id BIGINT NOT NULL REFERENCES comp_companies(id) ON DELETE CASCADE,
  feature_id BIGINT NOT NULL REFERENCES comp_features(id) ON DELETE RESTRICT,  -- Q4=B
  override_status VARCHAR(20) NOT NULL,                 -- 'enabled'|'disabled'
  reason TEXT NULL,
  UNIQUE (company_id, feature_id)
);
CREATE INDEX idx_comp_company_features_company_id ON comp_company_features (company_id);
CREATE INDEX idx_comp_company_features_feature_id ON comp_company_features (feature_id);
CREATE INDEX idx_comp_company_features_override_status ON comp_company_features (override_status);

-- 3.1.7 comp_company_addons (per-company addons purchased)
CREATE TABLE comp_company_addons (
  -- tableTemplate fields
  company_id BIGINT NOT NULL REFERENCES comp_companies(id) ON DELETE CASCADE,
  addon_id BIGINT NOT NULL REFERENCES comp_addons(id) ON DELETE CASCADE,
  quantity INT NOT NULL DEFAULT 1,
  purchased_at TIMESTAMPTZ NOT NULL,
  expires_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_comp_company_addons_company_id ON comp_company_addons (company_id);
CREATE INDEX idx_comp_company_addons_addon_id ON comp_company_addons (addon_id);
```

### 3.2 Modified Tables

| Table | Change | Migration script |
|-------|--------|------------------|
| `comp_companies` | Drop FK `package_id` → `const_packages.id`; Recreate FK → `comp_packages.id`; Backfill `package_id` ค่าใหม่ผ่าน `selected_package_uuid` lookup; default Seed สำหรับ legacy | `M8: *-alter-comp_companies-package-id-fk.ts` |

### 3.3 Entities / Domain Models

```typescript
// Backend interface (ใน module .interface.ts)
export interface Feature {
  readonly uuid: string;
  readonly code: string;
  readonly name: { th: string; en: string };
  readonly description?: { th: string; en: string };
  readonly menuKeys: string[];
  readonly sortOrder: number;
  readonly statusType: string;
  readonly createdAt: string;
  readonly updatedAt: string;
}

export interface FeatureWithUsage extends Feature {
  readonly packageCount: number;
  readonly addonCount: number;
}

export interface Package {
  readonly uuid: string;
  readonly code: string;
  readonly name: { th: string; en: string };
  readonly shortName?: string;
  readonly description?: { th: string; en: string };
  readonly billingInterval: 'month' | 'year' | null;
  readonly priceAmount: number;
  readonly currency: string;
  readonly userLimitMin?: number;
  readonly userLimitMax?: number;
  readonly isRecommend: boolean;
  readonly isContactUs: boolean;
  readonly isActive: boolean;
  readonly stripeProductId?: string;
  readonly stripePriceId?: string;
  readonly sortOrder: number;
}

export interface PackageWithFeatures extends Package {
  readonly features: Feature[];
  readonly effectiveMenuKeys: string[];          // dedupe union of features' menu_keys
}

export interface Addon {
  readonly uuid: string;
  readonly code: string;
  readonly name: { th: string; en: string };
  readonly billingInterval: 'month' | 'year' | null;
  readonly priceAmount: number;
  readonly isQuantifiable: boolean;
  readonly maxQuantity?: number;
  // ... etc
}

export interface CompanyFeatureItem {
  readonly featureUuid: string;
  readonly featureCode: string;
  readonly name: { th: string; en: string };
  readonly enabled: boolean;
  readonly source: 'package' | 'addon' | 'override-enabled' | 'override-disabled' | 'default-disabled';
  readonly overrideReason?: string;
}
```

## 4. API Specification

### 4.1 Feature CRUD

#### `GET /api/v2/sale-dashboard/features`
- Description: List features with usage info
- Auth required: yes (super_admin only)
- Query: `?search=&sortBy=&sortOrder=&page=&limit=&statusType=`
- Response (200):
  ```json
  {
    "code": 0,
    "msg": "success",
    "data": {
      "list": [{ "uuid": "...", "code": "payroll", "name": {"th":"เงินเดือน","en":"Payroll"}, "menuKeys": ["payroll.payrollSetting"], "packageCount": 2, "addonCount": 0, "sortOrder": 4, "statusType": "active" }],
      "pagination": { "page": 1, "limit": 20, "total": 14 }
    }
  }
  ```
- Errors: 401 unauthorized, 403 forbidden (not super_admin)

#### `GET /api/v2/sale-dashboard/features/:featureUuid`
- Response: Feature + usage details

#### `POST /api/v2/sale-dashboard/features`
- Auth: super_admin
- Request body:
  ```json
  {
    "code": "payroll",
    "name": { "th": "เงินเดือน", "en": "Payroll" },
    "description": { "th": "...", "en": "..." },
    "menuKeys": ["payroll.payrollSetting", "payroll.payrollProcess"],
    "sortOrder": 4
  }
  ```
- Response (201): Feature
- Errors: 400 (invalid code/menu_keys, duplicate code), 401, 403

#### `PUT /api/v2/sale-dashboard/features/:featureUuid`
- Body: partial Feature
- Response (200): Feature

#### `DELETE /api/v2/sale-dashboard/features/:featureUuid`
- Soft-delete (set status_type='archived')
- Errors: 400 (still in use by packages/addons)

#### `GET /api/v2/sale-dashboard/features/menu-keys/available`
- Response: list of leaf paths from PermissionDefault
  ```json
  { "code": 0, "data": { "menuKeys": ["dashboard", "payroll.payrollSetting", "timeAttendance.attendance", ...] } }
  ```

### 4.2 Package CRUD

| Method | Path | Description |
|---|---|---|
| GET | `/api/v2/sale-dashboard/packages` | List packages |
| GET | `/api/v2/sale-dashboard/packages/:packageUuid` | Get + features list |
| POST | `/api/v2/sale-dashboard/packages` | Create |
| PUT | `/api/v2/sale-dashboard/packages/:packageUuid` | Update metadata |
| DELETE | `/api/v2/sale-dashboard/packages/:packageUuid` | Soft-delete |
| PUT | `/api/v2/sale-dashboard/packages/:packageUuid/features` | Replace feature list — body: `{featureUuids: string[]}` |
| GET | `/api/v2/sale-dashboard/packages/:packageUuid/effective-menus` | Preview unlocked menus |
| PATCH | `/api/v2/sale-dashboard/packages/:packageUuid/identifier` | **(Added 2026-05-05)** Update package uuid เพื่อ sync กับ Stripe product ID — body: `{newUuid: string (uuid v4)}` — validate unique + ≠ current; FK ที่ pointing ไป `comp_packages.id` ไม่ได้รับผลกระทบ |

### 4.3 Addon CRUD

| Method | Path | Description |
|---|---|---|
| GET | `/api/v2/sale-dashboard/addons` | List |
| GET | `/api/v2/sale-dashboard/addons/:addonUuid` | Get |
| POST | `/api/v2/sale-dashboard/addons` | Create (validate is_quantifiable→max_quantity) |
| PUT | `/api/v2/sale-dashboard/addons/:addonUuid` | Update |
| DELETE | `/api/v2/sale-dashboard/addons/:addonUuid` | Soft-delete |
| PUT | `/api/v2/sale-dashboard/addons/:addonUuid/features` | Replace features |

### 4.4 Company Feature Toggle

#### `GET /api/v2/companies/:companyUuid/features`
- Response: list ของ feature ทั้งหมด พร้อม `enabled` + `source`
  ```json
  {
    "code": 0,
    "data": {
      "list": [
        { "featureUuid": "...", "featureCode": "payroll", "name": {...}, "enabled": true, "source": "package" },
        { "featureUuid": "...", "featureCode": "advanced_reporting", "name": {...}, "enabled": true, "source": "override-enabled", "overrideReason": "Trial extension" },
        { "featureUuid": "...", "featureCode": "ai_chat", "name": {...}, "enabled": false, "source": "default-disabled" }
      ]
    }
  }
  ```

#### `PUT /api/v2/companies/:companyUuid/features/:featureUuid`
- Body:
  ```json
  { "enabled": true, "reason": "Trial extension by sales" }
  ```
- Action: upsert row ใน `comp_company_features` (override_status = enabled/disabled)
- Response (200): updated CompanyFeatureItem

#### `DELETE /api/v2/companies/:companyUuid/features/:featureUuid`
- Action: hard delete row ใน `comp_company_features` → กลับไป default จาก package/addon
- Response (200): updated CompanyFeatureItem

### 4.5 Modified Existing Endpoint (Phase 4)

#### `GET /api/external/auth/v1/permissions/:userUuid`
- Existing endpoint
- Phase 4 change: ภายใน `getPermissionsService` → call `resolveUserEffectivePermissionService` ที่ filter JSONB ก่อนตอบ
- Response: permission JSONB เหมือนเดิม แต่ key ที่ feature ปิดถูกตัดออก

## 5. Frontend Components / Pages

| Component / Page | Path | Purpose | Props / Notes |
|-----------|------|---------|-------|
| **Pages** |
| FeatureListPage | `app/dashboard/feature-management/page.tsx` | List + search | wrapper render `<FeatureListView />` |
| FeatureCreatePage | `.../create/page.tsx` | Create form | render `<FeatureCreateView />` |
| FeatureDetailPage | `.../[id]/page.tsx` | View | render `<FeatureDetailView />` |
| FeatureEditPage | `.../[id]/edit/page.tsx` | Edit form | render `<FeatureEditView />` |
| (PackageManagement/AddonManagement: เหมือน pattern เดียวกัน) | | | |
| **Section Views** |
| FeatureListView | `sections/feature-management/feature-list-view.tsx` | Page-level orchestration | calls Redux hook |
| FeatureCreateView | `feature-create-view.tsx` | Wraps form | navigates on success |
| FeatureDetailView | `feature-detail-view.tsx` | Show + usage info | |
| FeatureEditView | `feature-edit-view.tsx` | Wraps form with initial values | |
| PackageDetailView | `sections/package-management/package-detail-view.tsx` | Detail + manage features section | |
| (etc.) | | | |
| **Components (per section)** |
| FeatureTable | `feature-management/components/feature-table.tsx` | MUI Table | props: `data`, `pagination`, `onSort`, etc |
| FeatureForm | `feature-management/components/feature-form.tsx` | Shared create/edit form | props: `defaultValues?`, `onSubmit` |
| FeatureDeleteDialog | `feature-management/components/feature-delete-dialog.tsx` | Confirm + show usage warnings | |
| FeatureUsageInfo | `feature-management/components/feature-usage-info.tsx` | Show packages/addons referencing | |
| PackageFeaturesSection | `package-management/components/package-features-section.tsx` | List package features + edit button | |
| PackageFeaturesEditDialog | `package-management/components/package-features-edit-dialog.tsx` | Multi-select features to assign | |
| PackageEffectiveMenusPreview | `package-management/components/package-effective-menus-preview.tsx` | Read-only menu tree | |
| AddonQuantifiableFields | `addon-management/components/addon-quantifiable-fields.tsx` | Conditional max_quantity | |
| AddonFeaturesSection | `addon-management/components/addon-features-section.tsx` | List addon features + edit | |
| CompanyFeaturesTabView | `client-management/feature-tab/company-features-tab-view.tsx` | Tab content | new tab in client detail |
| CompanyFeatureRow | `feature-tab/components/company-feature-row.tsx` | Single feature row with toggle + source badge | |
| CompanyFeatureToggleConfirmDialog | `feature-tab/components/company-feature-toggle-confirm-dialog.tsx` | Confirm + reason input | |
| CompanyFeatureSourceFilter | `feature-tab/components/company-feature-source-filter.tsx` | Dropdown filter | |
| **Reusable Components (cross-module)** |
| MenuKeysSelect | `components/feature-management/menu-keys-select.tsx` | Multi-select tree | props: `value`, `onChange`, `availableKeys` |
| MenuTreePreview | `components/feature-management/menu-tree-preview.tsx` | Read-only tree | |
| FeatureSourceBadge | `components/feature-management/feature-source-badge.tsx` | Colored chip | |
| MultilingualTextField | `components/feature-management/multilingual-text-field.tsx` | TH/EN paired field | |

## 6. Sequence / Flow

### 6.1 Create Feature Flow

```
[Super Admin: click Create Feature button]
      ↓
[CMS] Navigate to /dashboard/feature-management/create
      ↓
[CMS] FeatureCreateView renders FeatureForm
      ↓
[Super Admin] Fill code, name (TH/EN), select menu_keys from MenuKeysSelect (fetched via /features/menu-keys/available)
      ↓
[Super Admin] Submit
      ↓
[CMS] Saga dispatches sdCreateFeatureRequest → service.create() → POST /api/v2/sale-dashboard/features
      ↓
[Backend] requireSuperAdmin middleware → validateRequest (Zod) → controller → service
      ↓
[Service] feature.service.createFeatureService:
  - validate code unique (repository)
  - validate menu_keys ∈ getAvailableMenuKeys() (constant)
  - createFeatureRepository → INSERT into comp_features
  - return Feature
      ↓
[Backend] res.success(feature, 0, 'created')
      ↓
[CMS] Saga success → navigate to /dashboard/feature-management/[id]
```

### 6.2 Resolve User Permission Flow (Phase 4)

```
[End user: login HappyWork app, request /external/auth/v1/permissions]
      ↓
[Backend] permissions.controller.getPermissionsController(userUuid)
      ↓
[Service] permissions.service.getPermissionsService(userUuid):
  - load comp_permission_mapping by userUuid → permissionId
  - load comp_permission by permissionId → permissionJsonb
  - load user → companyId
  - call resolveCompanyEffectiveMenuKeysService(companyId)
      ↓
[Resolver] permissionResolver.service.resolveCompanyEffectiveMenuKeysService:
  - load company → package_id (cached per request)
  - load package_features by package_id → featureIds (filter: feature.status_type='active')
  - load ACTIVE company_addons by company_id → addonIds
      → SQL: WHERE company_id=? AND (expires_at IS NULL OR expires_at > NOW())   -- Q3=A
  - load addon_features by addonIds → more featureIds (filter: feature.status_type='active')
  - load comp_company_features by company_id → overrides
      (filter: override.feature.status_type='active')                              -- Q4=B
  - effective featureIds = union − overrides[disabled] + overrides[enabled]
  - load features by ids → menu_keys → flatten + dedupe
  - return Set<string>
      ↓
[Resolver] filterPermissionByMenuKeysService(permissionJsonb, allowedMenuKeys):
  - recursive walk JSONB tree
  - leaf check: if path in allowedMenuKeys → keep, else drop
  - return filtered JSONB
      ↓
[Backend] res.success(filteredPermission)
      ↓
[End user app] Render menus from filtered JSONB
```

### 6.3 Toggle Company Feature Flow

```
[Super Admin: client detail → Features tab → toggle switch]
      ↓
[CMS] CompanyFeatureRow opens CompanyFeatureToggleConfirmDialog
      ↓
[Super Admin] Enter optional reason → Confirm
      ↓
[CMS] Saga: PUT /api/v2/companies/:companyUuid/features/:featureUuid
      ↓
[Backend] companyFeature.service.setCompanyFeatureOverrideService:
  - validate company exists, feature exists
  - upsert comp_company_features (company_id, feature_id, override_status, reason)
  - return updated CompanyFeatureItem
      ↓
[CMS] Saga success → reducer updates state → row re-renders with new badge
```

## 7. Error Handling

| Scenario | Handling | Log level |
|----------|---------|-----------|
| Feature code already exists | 400 + AppError 'CODE_EXISTS' | warn |
| Menu key not in PermissionDefault | 400 + AppError 'INVALID_MENU_KEY' | warn |
| Feature delete blocked (in use) | 400 + AppError 'FEATURE_IN_USE' (พร้อม count) | info |
| Package delete blocked (companies using) | 400 + AppError 'PACKAGE_IN_USE' | info |
| Addon: is_quantifiable=true ขาด max_quantity | 400 + AppError 'MAX_QUANTITY_REQUIRED' | warn |
| Resolver: company package_id is NULL | Default to Seed package, log warn | warn |
| Resolver: comp_permission JSONB malformed | 500 + AppError 'PERMISSION_DATA_CORRUPTED' | error |
| Backfill M8: company.selected_package_uuid not in comp_packages | Migration aborts with error → fix data → retry | fatal |
| RBAC fail (admin/manager call admin endpoint) | 403 + AppError 'FORBIDDEN' | warn |
| Concurrent toggle (race condition) | DB UNIQUE constraint catches, retry with 409 | warn |

ทุก error log ด้วย `logger.error('operation', { error: ..., stack: ..., context })`

## 8. Performance Considerations

### Expected load
- Master data CRUD: low (super_admin only, < 100 calls/day)
- Permission resolver: high (every authenticated request from end users) — must be fast

### Caching strategy
- **In-request memoization**: `resolveCompanyEffectiveMenuKeysService` cache result per (companyId, requestId)
- ไม่ใช้ shared cache (Redis) ในรอบนี้ — ถ้า profile แล้ว resolver ช้า ค่อยเพิ่ม L2 cache phase ถัดไป

### N+1 query risks
- Risk #1: ตอน list ของ company หลายตัวพร้อมกัน เรียก resolver ในลูป → ใช้ batch ด้วย DataLoader pattern (อนาคต) หรือ batch query `WHERE company_id IN (...)`
- Risk #2: `getCompanyFeaturesService` ต้อง pre-fetch package_features + addon_features + overrides ในไม่กี่ query ไม่ลูป

### Index requirements
- `comp_features.code` UNIQUE
- `comp_packages.code` UNIQUE
- `comp_addons.code` UNIQUE
- `comp_package_features (feature_id)` — สำหรับ "feature ใช้ใน package ไหน"
- `comp_addon_features (feature_id)` — เช่นเดียวกัน
- `comp_company_features (company_id, feature_id)` UNIQUE — primary lookup
- `comp_company_features (override_status)` — filter ใน some queries
- `comp_company_addons (company_id, addon_id)` — primary lookup
- GIN index บน `name` JSONB ของทุก master table — รองรับ search

### Decision rule: Subquery vs Pre-fetch
- Resolver มี join ระหว่าง package_features + addon_features + overrides + features → 3-way join เล็ก, dataset เล็ก (per company) → **JOIN/subquery ดีกว่า**
- ถ้าต้อง resolve user หลายร้อยคนพร้อมกัน → **pre-fetch with `WHERE company_id IN (...)` batch** ตาม pattern DataLoader

## 9. Security Considerations

### Auth / Authorization
- ทุก endpoint v2/admin: `validateAccessToken` → `requireSuperAdmin`
- `requireSuperAdmin` middleware เช็ค `req.user.role === 'super_admin'`; ถ้าไม่ใช่ → 403
- Frontend: RBAC matrix ใน `rbac.ts` — admin/manager/viewer ทั้งหมด no access

### Input validation
- ใช้ Zod schema ทุก request body/params/query
- `code` regex: `^[a-z0-9_]+$`
- `menu_keys` whitelist จาก PermissionDefault (server-side enforcement)
- Multilingual fields ต้องมี `th` + `en` (Zod ตรวจ)

### Sensitive data handling
- Stripe IDs (stripe_product_id, stripe_price_id) เก็บ plain (เป็น public ID จาก Stripe)
- ไม่มี data sensitive อื่นใน feature/package/addon

### Rate limiting
- Master CRUD: ใช้ rate limit ของ admin API ปัจจุบัน (เช่น 100 req/min/user)
- Permission API (Phase 4): ปัจจุบันมี rate limit อยู่แล้ว — resolver ไม่เพิ่ม layer

### Audit
- ทุก row บันทึก `created_by`, `updated_by`, `archived_by` (uuid ของ super_admin)
- `comp_company_features` มี `reason` field สำหรับ override audit

## 10. Migration / Rollout Plan

### Resolved Decisions (from requirement.md §9, 2026-05-04)

| ID | Question | Decision | Impact on this SSD |
|----|----------|----------|---------------------|
| Q1 | Seed `comp_package_features` mapping ครั้งแรก | **C — Cartesian (every package × every feature)** | M3 migration ต้อง seed Cartesian; admin ค่อยเข้ามาปิดเฉพาะที่ขายไม่ครบ |
| Q2 | เปลี่ยน package → override จัดการยังไง | **A — Keep override** | ไม่เปลี่ยน schema; documented behavior |
| Q3 | Addon expired (`expires_at < NOW()`) → auto-disable? | **A — Auto-disable** | Resolver §6.2 ต้อง filter `expires_at IS NULL OR expires_at > NOW()` ตอน load `comp_company_addons` |
| Q4 | Soft-delete feature → override row จัดการยังไง | **B — FK no-CASCADE + Resolver skip archived** | M6 FK `feature_id` = `RESTRICT` (ไม่ใช่ CASCADE); Resolver filter `feature.status_type='active'` |
| Q5 | `comp_companies.add_ons` jsonb format | **Resolved by code review** | format `Array<{uuid, quantity}>` — M7 SQL พร้อมใช้ |
| Q6 | RBAC resource name สำหรับ Master Data Package Management (collision กับ `packages` ที่มีอยู่แล้ว) | **A — ใช้ resource name ใหม่ `master_packages`** (resolved 2026-05-05) | `frontend.md §4` ปรับ example + components ของ Page B ใช้ key `"master_packages"`; existing `packages` ใน `rbac.ts` คงเดิม (ไม่ break `package-config-feature` view) |
| Q7 | Slice file naming collision: legacy `sale-dashboard-feature-management` ถูกใช้โดย 4 production views อยู่ก่อนแล้ว ทำให้ F3 Master Data slices ไปชนชื่อ | **B (rename-only) — Rename legacy slice → `sale-dashboard-package-config-feature` (ตาม section folder); F3 Master Data ใช้ชื่อ `sale-dashboard-{feature,package,addon}-management` ตาม spec เดิม; pure rename ห้าม refactor logic** (resolved 2026-05-05) | F3 hooks/reducer slots/saga watchers/types ของ Master Data ใช้ชื่อตาม spec; legacy slice + 4 importer views update ใช้ namespace `SaleDashboardPackageConfigFeature` แทน |
| Q8 | Frontend `AddonBillingInterval` ใน F3 slice draft ใช้ `'monthly'\|'yearly'\|'one_time'` ไม่ตรงกับ backend B3 Zod `z.enum(['month','year']).nullable()` → addon CRUD จะ fail at runtime | **Rename frontend value: `monthly→month`, `yearly→year`, drop `one_time` (semantic ย้ายไป null)** (resolved 2026-05-05) | Form select 3 options: One-time (value=`""` → wire `null`), Monthly (`"month"`), Yearly (`"year"`); mappers convert null↔""; i18n keys `addonManagement.billing.{month,year,one_time}` (one_time = null display label); Package side ถูกต้องอยู่แล้ว (F5 fix) |

### Backward compat
- `comp_permission`/`comp_permission_mapping` JSONB structure ไม่เปลี่ยน
- `const_packages` table ไม่ลบ — v1 caller ยังใช้ได้
- `featureManagement` v1 module ยังทำงาน — Phase 5 ค่อย deprecate
- `featureDefinitions.ts` constant ยังอยู่

### Migration steps (8 migrations)
ดูรายละเอียดใน `migration` section ของ Phase Plan ภายใน task-list.md และไฟล์ migration code:
1. **M1** `*-create-comp_features.ts` (+ seed)
2. **M2** `*-create-comp_packages.ts` (+ seed จาก packageMasterData, **preserve UUID**)
3. **M3** `*-create-comp_package_features.ts`
4. **M4** `*-create-comp_addons.ts` (+ seed จาก addonsMasterData)
5. **M5** `*-create-comp_addon_features.ts`
6. **M6** `*-create-comp_company_features.ts`
7. **M7** `*-create-comp_company_addons.ts` (+ migrate จาก `comp_companies.add_ons`)
8. **M8** `*-alter-comp_companies-package-id-fk.ts` (backfill + recreate FK)

### Rollback plan
- แต่ละ migration มี `down()` function (DROP TABLE หรือ revert FK)
- ก่อน apply M8 production: เก็บ snapshot ของ `comp_companies.package_id` (เพื่อ restore manually)
- ถ้า Phase 4 (resolver integration) เจอ regression → revert deploy → resolver ไม่ activate → permission API ทำงานเหมือนเดิม

### Feature flag
- ไม่ใช้ feature flag — ใช้ phased deploy:
  - P1-3 deploy ก่อน (DB + CRUD UI + override UI + resolver service)
  - P4 deploy แยก (เปลี่ยน existing permission API ให้ใช้ resolver) — กดได้ทันทีที่พร้อม

### Pre-deploy checklist (สำหรับ Phase 4)
- [ ] ทุก feature ใน `comp_features` มี `menu_keys` ครบ (admin map ผ่าน CMS แล้ว)
- [ ] ทุก package ใน `comp_packages` มี `comp_package_features` mapping ครบ
- [ ] Run script เปรียบเทียบ effective menus ของ company (sample 100 ราย) กับ expected — ตรงทุกคน
- [ ] Integration test ผ่าน 100%

## 11. Testing Strategy

### Unit tests
- **Backend services**:
  - `feature.service.spec.ts` — create/update/delete with validation
  - `packageCrud.service.spec.ts` — replaceFeatures, effectiveMenus
  - `addon.service.spec.ts` — is_quantifiable validation
  - `companyFeature.service.spec.ts` — getCompanyFeaturesService source mapping
  - `permissionResolver.service.spec.ts` — delta logic, addon union, override apply, filter JSONB
- **Frontend**: ใช้ existing pattern (ถ้ามี Jest + Testing Library) — ทดสอบ form validation, RBAC visibility

### Integration tests
- Backend with test DB:
  - CRUD flow ครบ (create feature → create package → assign features → company override → resolve)
  - RBAC: admin token call admin endpoint → 403
  - Migration M8 dry-run บน sample data

### E2E tests
- Manual ใน staging:
  - Login super_admin → CRUD ครบ flow
  - Login admin → ไม่เห็นเมนู
  - Toggle feature ของ company → end user app ของ company นั้นเห็น/ไม่เห็นเมนู

### Manual QA
- Phase 4 critical: regression ทุก permission-dependent endpoint
- ดู `testcase.md` สำหรับ test case รายข้อ

## 12. References

- Related code paths (full list — ดู `summary.md` "Files Reviewed"):
  - `happywork-backend/src/constant/compPermission.ts`
  - `happywork-backend/src/database/postgresql/migrations/data/`
  - `happywork-backend/src/modules/v1/employee/request/*` — backend pattern reference
  - `happywork-backend/src/api/v1/employee/request/*` — routes pattern reference
  - `happywork-backend/src/modules/v2/admin/package/*` — Stripe sync (อย่าแก้)
  - `happywork-sale-cms/src/app/dashboard/survey/*` — frontend pattern reference
  - `happywork-sale-cms/src/sections/survey/*`
- External docs / RFCs:
  - [`subscription-package-summary.md`](../subscription/subscription-package-summary.md) — package schema
  - `AI-RULES-CORE.md`, `AI-RULES-BACKEND.md`, `AI-RULES-MULTILINGUAL.md`
  - `PERMISSION_GUIDE.md`, `PERMISSION_IMPLEMENTATION.md` (sale-cms repo)
