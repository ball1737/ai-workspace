---
feature: feature-management
type: feature-backend-detail
created: 2026-05-04
updated: 2026-05-04
owner: SA (initial), Backend Dev (implementation)
status: draft
---

# Feature Management — Backend Detail

> รายละเอียด backend ทั้งหมด: routes, controllers, services, repositories, models
> Reference pattern: `happywork-backend/src/api/v1/employee/request/*` + `src/modules/v1/employee/request/*`

---

## โครงสร้างโดยรวม

ระบบ feature management ใหม่ทั้งหมดอยู่ใต้ namespace **v2/admin** เพื่อ align กับ Stripe flow v2

```
src/
├── api/v2/admin/
│   ├── feature/                       # CRUD master features
│   │   ├── feature.routes.ts
│   │   └── feature.controller.ts
│   ├── packageCrud/                   # CRUD master packages (ชื่อต่างจาก v2/admin/package เดิม)
│   │   ├── packageCrud.routes.ts
│   │   └── packageCrud.controller.ts
│   ├── addon/                         # CRUD master addons
│   │   ├── addon.routes.ts
│   │   └── addon.controller.ts
│   └── companyFeature/                # Per-company feature override
│       ├── companyFeature.routes.ts
│       └── companyFeature.controller.ts
│
├── modules/v2/admin/
│   ├── feature/
│   │   ├── feature.interface.ts
│   │   ├── feature.repository.ts
│   │   ├── feature.service.ts
│   │   └── feature.adapter.ts
│   ├── packageCrud/
│   │   ├── packageCrud.interface.ts
│   │   ├── packageCrud.repository.ts
│   │   ├── packageCrud.service.ts
│   │   └── packageCrud.adapter.ts
│   ├── addon/
│   │   ├── addon.interface.ts
│   │   ├── addon.repository.ts
│   │   ├── addon.service.ts
│   │   └── addon.adapter.ts
│   ├── companyFeature/
│   │   ├── companyFeature.interface.ts
│   │   ├── companyFeature.repository.ts
│   │   ├── companyFeature.service.ts
│   │   └── companyFeature.adapter.ts
│   └── permissionResolver/            # Shared resolver
│       ├── permissionResolver.interface.ts
│       ├── permissionResolver.repository.ts
│       └── permissionResolver.service.ts
│
├── database/postgresql/
│   ├── migrations/data/               # ดู migration.md
│   └── models/data/
│       ├── compFeatures.model.ts
│       ├── compPackages.model.ts
│       ├── compPackageFeatures.model.ts
│       ├── compAddons.model.ts
│       ├── compAddonFeatures.model.ts
│       ├── compCompanyFeatures.model.ts
│       └── compCompanyAddons.model.ts
│
├── middlewares/
│   └── requireSuperAdmin.middleware.ts (ถ้ายังไม่มี — เช็ค role super_admin)
│
└── constant/
    └── compPermission.ts              # เพิ่ม helper getAvailableMenuKeys()
```

---

## 1. API Routes (v2)

### 1.1 Feature CRUD — `/api/v2/sale-dashboard/features`

| Method | Path                                         | Controller                       | Description                                                                 |
| ------ | -------------------------------------------- | -------------------------------- | --------------------------------------------------------------------------- |
| GET    | `/api/v2/sale-dashboard/features`                     | `getFeatureListController`       | List feature ทั้งหมด (รองรับ search, sort, pagination)                      |
| GET    | `/api/v2/sale-dashboard/features/:featureUuid`        | `getFeatureByUuidController`     | Get feature โดย uuid (รวมข้อมูล package/addon ที่อ้างอิง)                   |
| POST   | `/api/v2/sale-dashboard/features`                     | `createFeatureController`        | สร้าง feature ใหม่                                                          |
| PUT    | `/api/v2/sale-dashboard/features/:featureUuid`        | `updateFeatureController`        | แก้ไข feature (รวม menu_keys)                                               |
| DELETE | `/api/v2/sale-dashboard/features/:featureUuid`        | `deleteFeatureController`        | Soft-delete feature (block ถ้ามี package/addon ผูกอยู่)                     |
| GET    | `/api/v2/sale-dashboard/features/menu-keys/available` | `getAvailableMenuKeysController` | Return leaf paths ทั้งหมดจาก `PermissionDefault` (สำหรับ frontend dropdown) |

**Middleware ทุก route:** `validateAccessToken` → `requireSuperAdmin` → `validateRequest(zodSchema)`

### 1.2 Package CRUD — `/api/v2/sale-dashboard/packages`

| Method | Path                                                  | Controller                           | Description                                                                 |
| ------ | ----------------------------------------------------- | ------------------------------------ | --------------------------------------------------------------------------- |
| GET    | `/api/v2/sale-dashboard/packages`                              | `getPackageListController`           | List packages พร้อมจำนวน feature ที่ผูก                                     |
| GET    | `/api/v2/sale-dashboard/packages/:packageUuid`                 | `getPackageByUuidController`         | Get package + feature list                                                  |
| POST   | `/api/v2/sale-dashboard/packages`                              | `createPackageController`            | สร้าง package ใหม่                                                          |
| PUT    | `/api/v2/sale-dashboard/packages/:packageUuid`                 | `updatePackageController`            | แก้ไข metadata package (ยกเว้น features)                                    |
| DELETE | `/api/v2/sale-dashboard/packages/:packageUuid`                 | `deletePackageController`            | Soft-delete package (block ถ้ามี company อ้างอิง)                           |
| PUT    | `/api/v2/sale-dashboard/packages/:packageUuid/features`        | `replacePackageFeaturesController`   | Replace feature list ของ package — body: `{featureUuids: string[]}`         |
| GET    | `/api/v2/sale-dashboard/packages/:packageUuid/effective-menus` | `getPackageEffectiveMenusController` | Preview เมนูที่ package นี้จะปลดล็อก (รวม menu_keys ของทุก feature, dedupe) |
| PATCH  | `/api/v2/sale-dashboard/packages/:packageUuid/identifier`      | `updatePackageUuidController`        | **(Added 2026-05-05)** แก้ไข uuid ของ package เพื่อ sync กับ Stripe product ID — body: `{newUuid: string (uuid v4)}` — validate format/unique/≠current; ทุก FK ที่ pointing ไป `comp_packages.id` (numeric) ไม่ได้รับผลกระทบ |

### 1.3 Addon CRUD — `/api/v2/sale-dashboard/addons`

| Method | Path                                       | Controller                       | Description                                                                |
| ------ | ------------------------------------------ | -------------------------------- | -------------------------------------------------------------------------- |
| GET    | `/api/v2/sale-dashboard/addons`                     | `getAddonListController`         | List addons                                                                |
| GET    | `/api/v2/sale-dashboard/addons/:addonUuid`          | `getAddonByUuidController`       | Get addon + feature list                                                   |
| POST   | `/api/v2/sale-dashboard/addons`                     | `createAddonController`          | สร้าง addon ใหม่ (validate `is_quantifiable=true → max_quantity required`) |
| PUT    | `/api/v2/sale-dashboard/addons/:addonUuid`          | `updateAddonController`          | แก้ไข metadata addon                                                       |
| DELETE | `/api/v2/sale-dashboard/addons/:addonUuid`          | `deleteAddonController`          | Soft-delete (block ถ้ามี company purchase อยู่)                            |
| PUT    | `/api/v2/sale-dashboard/addons/:addonUuid/features` | `replaceAddonFeaturesController` | Replace feature list — body: `{featureUuids: string[]}`                    |

### 1.4 Company Feature Toggle — `/api/v2/companies/:companyUuid/features`

| Method | Path                                                         | Controller                               | Description                                                                                                                                                                              |
| ------ | ------------------------------------------------------------ | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GET    | `/api/v2/companies/:companyUuid/features`              | `getCompanyFeaturesController`           | List feature ทั้งหมด พร้อมสถานะ effective + source ของแต่ละ feature: `{featureUuid, name, enabled, source: 'package'/'addon'/'override-enabled'/'override-disabled'/'default-disabled'}` |
| PUT    | `/api/v2/companies/:companyUuid/features/:featureUuid` | `setCompanyFeatureOverrideController`    | Upsert override — body: `{enabled: bool, reason?: string}`                                                                                                                               |
| DELETE | `/api/v2/companies/:companyUuid/features/:featureUuid` | `removeCompanyFeatureOverrideController` | ลบ override → กลับไปใช้ default จาก package/addon                                                                                                                                        |

---

## 2. Modules (Business Logic)

### 2.1 `modules/v2/admin/feature/`

#### feature.interface.ts

```typescript
import { z } from "zod";

export const featureBaseSchema = z.object({
  code: z
    .string()
    .min(1)
    .max(50)
    .regex(/^[a-z0-9_]+$/),
  name: z.object({ th: z.string().min(1), en: z.string().min(1) }),
  description: z.object({ th: z.string(), en: z.string() }).optional(),
  menuKeys: z.array(z.string()).default([]),
  // Phase 5 (Wave 2) — mobile parity field; default [] = mobile menu locked.
  // Validated against getAvailableMobileMenuKeys() in service layer.
  mobileMenuKeys: z.array(z.string()).default([]),
  sortOrder: z.number().int().default(0),
});

// Phase 5 Wave 2 (Q3=B breaking) — response shape for /features/menu-keys/available.
// Replaces legacy `{ menuKeys: string[] }`; frontend Wave 6 syncs.
export const menuKeysAvailableResponseSchema = z.object({
  web: z.array(z.string()),
  mobile: z.array(z.string()),
});

export const createFeatureSchema = z.object({
  body: featureBaseSchema,
});

export const updateFeatureSchema = z.object({
  params: z.object({ featureUuid: z.string().uuid() }),
  body: featureBaseSchema.partial(),
});

export const listFeaturesSchema = z.object({
  query: z.object({
    search: z.string().optional(),
    sortBy: z
      .enum(["name", "code", "sortOrder", "createdAt"])
      .default("sortOrder"),
    sortOrder: z.enum(["asc", "desc"]).default("asc"),
    page: z.coerce.number().int().min(1).default(1),
    limit: z.coerce.number().int().min(1).max(100).default(20),
    statusType: z.enum(["active", "inactive"]).optional(),
  }),
});

export interface Feature {
  uuid: string;
  code: string;
  name: { th: string; en: string };
  description?: { th: string; en: string };
  menuKeys: string[];
  // Phase 5 (Wave 2) — mobile parity. Always present in response (default []).
  mobileMenuKeys: string[];
  sortOrder: number;
  statusType: string;
  createdAt: string;
  updatedAt: string;
}

export interface FeatureWithUsage extends Feature {
  packageCount: number; // จำนวน package ที่อ้างอิง feature นี้
  addonCount: number;
}
```

#### feature.repository.ts (functions)

- `getFeatureByIdRepository(id: string): Promise<FeatureRow | null>`
- `getFeatureByUuidRepository(uuid: string): Promise<FeatureRow | null>`
- `getFeatureByCodeRepository(code: string): Promise<FeatureRow | null>` (สำหรับ uniqueness check)
- `listFeaturesRepository(filter, pagination): Promise<{rows, total}>`
- `createFeatureRepository(data, createdBy): Promise<FeatureRow>`
- `updateFeatureRepository(uuid, data, updatedBy): Promise<FeatureRow>`
- `softDeleteFeatureRepository(uuid, archivedBy): Promise<void>`
- `countFeatureUsageRepository(featureId): Promise<{packages: number, addons: number}>` (เช็คก่อน delete)

#### feature.service.ts (functions)

- `getFeatureListService(filter, pagination): Promise<{features: FeatureWithUsage[], total}>`
- `getFeatureByUuidService(uuid): Promise<FeatureWithUsage>`
- `createFeatureService(input, createdBy): Promise<Feature>`
  - Validate `code` unique
  - Validate `menuKeys` ทุกตัวอยู่ใน `getAvailableMenuKeys()` (จาก `compPermission.ts`) — error code `INVALID_MENU_KEY` (400)
  - **Phase 5 Wave 2:** Validate `mobileMenuKeys` ทุกตัวอยู่ใน `getAvailableMobileMenuKeys()` — error code `INVALID_MOBILE_MENU_KEY` (400)
- `updateFeatureService(uuid, input, updatedBy): Promise<Feature>`
  - Re-validate ทั้ง `menuKeys` + `mobileMenuKeys` ถ้า field มีใน partial body (ไม่บังคับส่ง)
- `deleteFeatureService(uuid, archivedBy): Promise<void>`
  - เรียก `countFeatureUsageRepository` ก่อน — block ถ้า package/addon > 0
- `getAvailableMenuKeysService(): MenuKeysAvailableResponse` — **Phase 5 Wave 2 (Q3=B breaking)** returns `{ web: getAvailableMenuKeys(), mobile: getAvailableMobileMenuKeys() }` (sync — both helpers are synchronous tree walkers)

#### feature.adapter.ts

- `convertFeatureToResponse(row: FeatureRow): Feature` — แปลง snake_case → camelCase, hide `id`
- `normalizeFeatureListResponse(rows, totalCount, page, limit): PaginatedResponse<Feature>`

### 2.2 `modules/v2/admin/packageCrud/`

#### packageCrud.interface.ts

```typescript
export const packageBaseSchema = z.object({
  code: z.string().min(1).max(50),
  name: z.object({ th: z.string(), en: z.string() }),
  shortName: z.string().max(50).optional(),
  description: z.object({ th: z.string(), en: z.string() }).optional(),
  billingInterval: z.enum(["month", "year"]).nullable(),
  priceAmount: z.number().min(0),
  currency: z.string().default("thb"),
  userLimitMin: z.number().int().nullable(),
  userLimitMax: z.number().int().nullable(),
  isRecommend: z.boolean().default(false),
  isContactUs: z.boolean().default(false),
  isActive: z.boolean().default(true),
  sortOrder: z.number().int().default(0),
});

export interface Package {
  /* ... */
}
export interface PackageWithFeatures extends Package {
  features: Feature[];
  effectiveMenuKeys: string[];
}
```

#### packageCrud.repository.ts (functions)

- `getPackageByIdRepository`, `getPackageByUuidRepository`, `getPackageByCodeRepository`
- `listPackagesRepository(filter, pagination)`
- `createPackageRepository(data, createdBy)`
- `updatePackageRepository(uuid, data, updatedBy)`
- `softDeletePackageRepository(uuid)`
- `countPackageUsageRepository(packageId)` — count `comp_companies.package_id`
- `getPackageFeaturesRepository(packageId): Promise<Feature[]>`
- `replacePackageFeaturesRepository(packageId, featureIds[]): Promise<void>` — transaction: delete all + insert new
- **(Added 2026-05-05)** `updatePackageUuidRepository(currentUuid, newUuid, updatedBy)` — UPDATE `comp_packages` SET `uuid = newUuid`, `updated_by`, `updated_at = NOW()` WHERE `uuid = currentUuid`; ใช้ transaction เพื่อ atomic + isolation

#### packageCrud.service.ts (functions)

- `getPackageListService`
- `getPackageByUuidService` — return พร้อม features list
- `createPackageService` (validate code unique)
- `updatePackageService`
- `deletePackageService` (block ถ้ามี company อ้างอิง)
- `replacePackageFeaturesService(packageUuid, featureUuids[])` — convert uuid→id, validate ทุก feature exists
- `getPackageEffectiveMenusService(packageUuid): Promise<string[]>` — รวม menu_keys ของทุก feature, dedupe
- **(Added 2026-05-05)** `updatePackageUuidService(currentUuid, newUuid, actor)` — Stripe-sync: validate `newUuid` (1) format = uuid v4, (2) ≠ current uuid, (3) ห้ามซ้ำกับ row อื่นใน `comp_packages.uuid`; เรียก repo update ภายใน trx; return `Package` ที่ update แล้ว. **Caveat:** ถ้ามี denormalized `package_uuid` ใน table อื่น (เช่น legacy `comp_companies.selected_package_uuid` ที่อาจจะยังคงอยู่หลัง M8) → ต้อง update ตามด้วย ใน trx เดียวกัน. Backend agent ต้อง grep หา denormalized references ก่อน implement และรายงานกลับ Lead ถ้าเจอ

### 2.3 `modules/v2/admin/addon/`

โครงสร้างเหมือน packageCrud แต่:

- เพิ่ม validation: `isQuantifiable=true` → ต้องมี `maxQuantity > 0`
- `replaceAddonFeaturesService` คล้าย package
- ไม่มี `comp_companies.addon_id` ที่ block delete แต่ check `comp_company_addons` แทน

### 2.4 `modules/v2/admin/companyFeature/`

#### companyFeature.service.ts (key functions)

```typescript
// Return รายการ feature ทั้งหมด พร้อมสถานะของบริษัทนี้
export const getCompanyFeaturesService = async (
  companyUuid: string,
): Promise<CompanyFeatureItem[]> => {
  // 1. ดึง company → company.id, package_id
  // 2. ดึง features ทั้งหมดจาก comp_features
  // 3. ดึง package_features (set ของ feature_id ที่ package เปิด)
  // 4. ดึง company_addons → addon_features (set ของ feature_id ที่ addons เปิด)
  // 5. ดึง overrides จาก comp_company_features
  // 6. สำหรับแต่ละ feature: คำนวณ effective + source
  //    - source = 'override-enabled' if override.status='enabled'
  //    - source = 'override-disabled' if override.status='disabled'
  //    - source = 'package' if no override and in package_features
  //    - source = 'addon' if no override and in addon_features
  //    - source = 'default-disabled' otherwise
};

export const setCompanyFeatureOverrideService = async (
  companyUuid: string,
  featureUuid: string,
  enabled: boolean,
  reason: string | undefined,
  actorUserUuid: string,
): Promise<void> => {
  // upsert ใน comp_company_features
  // override_status = enabled ? 'enabled' : 'disabled'
};

export const removeCompanyFeatureOverrideService = async (
  companyUuid: string,
  featureUuid: string,
  actorUserUuid: string,
): Promise<void> => {
  // hard delete ของ row ใน comp_company_features (กลับไป default)
};
```

### 2.5 `modules/v2/admin/permissionResolver/` (SHARED)

โมดูลกลางที่ใช้ทั้งใน v1 และ v2 (call จาก existing endpoints ที่ return permission)

#### permissionResolver.service.ts

```typescript
// คืน Set<feature_id> ที่ effective ของบริษัท
export const resolveCompanyEffectiveFeatureIdsService = async (
  companyId: string,
  cache?: ResolverCache,
): Promise<EffectiveFeatureIds>;

// คืน Set<menu_key> (path เช่น "payroll.payrollSetting") ที่ enabled — WEB
export const resolveCompanyEffectiveMenuKeysService = async (
  companyId: string,
  cache?: ResolverCache,
): Promise<EffectiveMenuKeys>;

// คืน Set<menu_key> สำหรับ MOBILE (Phase 5 W3) — reuse feature-id resolution (cached); batch lookup mobile_menu_keys
export const resolveCompanyEffectiveMobileMenuKeysService = async (
  companyId: string,
  cache?: ResolverCache,
): Promise<EffectiveMenuKeys>;

// Filter PermissionDefault-shaped JSONB ตัด key ที่ leaf path ไม่อยู่ใน allowedMenuKeys (WEB)
export const filterPermissionByMenuKeysService = (
  permissionJsonb: PermissionJsonb,
  allowedMenuKeys: ReadonlySet<string>,
): PermissionJsonb;

// MOBILE counterpart (Phase 5 W3) — delegate to web walker; leaf-detection action-key-set-agnostic
// (mobile leaves เป็น subset ของ web ACTION_KEYS เพราะ hasActionKey() เช็ค any-key-present)
export const filterPermissionMobileByMenuKeysService = (
  permissionJsonb: PermissionJsonb,
  allowedMobileMenuKeys: ReadonlySet<string>,
): PermissionJsonb;

// Extract leaf paths from JSONB tree (WEB) — Phase 4 P4.4 admin save validation reuse
export const extractMenuKeysFromPermissionJsonbService = (
  permissionJsonb: PermissionJsonb,
): string[];

// MOBILE counterpart (Phase 5 W3) — delegate to web extractor (same rationale as filter)
export const extractMobileMenuKeysFromPermissionJsonbService = (
  permissionJsonb: PermissionJsonb,
): string[];

// Convenience: load comp_permission ของ user → filter → return (WEB only — mobile path TBD Phase 5 W4)
export const resolveUserEffectivePermissionService = async (
  userUuid: string,
  cache?: ResolverCache,
): Promise<PermissionJsonb>;

// Centralised effective package_id resolution (Seed fallback) — shared with companyFeature.service
export const resolveEffectivePackageIdService = async (
  companyId: string,
  cache?: ResolverCache,
): Promise<string>;
```

**Performance:** ใช้ in-request memoization ผ่าน `ResolverCache` (`Map<string, unknown>`) — caller สร้าง `new Map()` ต่อ request แล้วส่งเข้า service functions
ทุกตัว. Cache keys ที่ใช้:

- `featureIds:{companyId}` → `EffectiveFeatureIds`
- `menuKeys:{companyId}` → `EffectiveMenuKeys` (web)
- `mobileMenuKeys:{companyId}` → `EffectiveMenuKeys` (mobile — Phase 5 W3)
- `packageId:{companyId}` → `string` (Seed fallback resolved id)

#### permissionResolver.repository.ts (เพิ่มเติม Phase 5 W3)

```typescript
// คืน mobile_menu_keys (string[]) แบบ flatten ของ feature_ids (mirror getFeatureMenuKeysRepository)
// Pre-fetch + LATERAL `jsonb_array_elements_text(f.mobile_menu_keys)` + WHERE IN
// Guard: `jsonb_typeof(f.mobile_menu_keys) = 'array'` กัน corrupt jsonb crash (M2-style)
export const getFeatureMobileMenuKeysRepository = async (
  featureIds: readonly string[],
): Promise<readonly string[]>;
```

---

## 3. Database Models (Objection.js)

ทุก model define:

- `tableName`
- `idColumn = 'id'`
- `jsonSchema` validation
- `relationMappings` (ใช้ `BelongsToOneRelation`, `HasManyRelation`, `ManyToManyRelation`)
- TS interface สำหรับ row type

### compFeatures.model.ts

```typescript
export class CompFeatures extends Model {
  static tableName = "comp_features";
  static idColumn = "id";

  id!: string;
  uuid!: string;
  code!: string;
  name!: { th: string; en: string };
  description?: { th: string; en: string };
  menu_keys!: string[];
  sort_order!: number;
  status_type!: string;
  // ... timestamps

  static relationMappings = {
    packages: {
      relation: Model.ManyToManyRelation,
      modelClass: () => CompPackages,
      join: {
        from: "comp_features.id",
        through: {
          from: "comp_package_features.feature_id",
          to: "comp_package_features.package_id",
        },
        to: "comp_packages.id",
      },
    },
    addons: {
      /* similar */
    },
    companyOverrides: {
      relation: Model.HasManyRelation,
      modelClass: () => CompCompanyFeatures,
      join: {
        from: "comp_features.id",
        to: "comp_company_features.feature_id",
      },
    },
  };
}
```

(โครงเดียวกันสำหรับ `compPackages`, `compAddons`, `compCompanyFeatures`, `compCompanyAddons` — ดู link tables เป็น `extends Model` ปกติ)

---

## 4. Updates ของ Existing API ที่ใช้ `comp_permission` (Phase 4 + Phase 5 W4)

### 4.1 จุดที่ต้องแก้

| File                                                 | สิ่งที่ต้องทำ                                                                                                      |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `src/api/external/auth/permissions.controller.ts`    | `getPermissionsController` → เรียก `resolveUserEffectivePermissionService` แทน `getPermissionsService` ตรงๆ        |
| `src/modules/v1/externalAuth/permissions.service.ts` | `getPermissionsService(userUuid)` → load mapping → load comp_permission → filter ผ่าน resolver → return            |
| `src/modules/v1/admin/compPermission/*`              | List endpoint ของ comp_permission ใน CMS เดิม → filter เมนูที่ list ตาม company effective menus                    |
| `src/modules/v1/admin/compPermissionMapping/*`       | ตอน save permission JSONB ของ user → validate ว่า key ที่จะ save อยู่ใน enabled menus ของบริษัท (reject ถ้าไม่ใช่) |

### 4.2 หลักการ

- `comp_permission.permission` JSONB ยังเก็บข้อมูลเต็ม (ไม่ตัด)
- Resolver filter ตอน **อ่าน** เท่านั้น
- Frontend Permission Management UI (ของเดิม): filter เมนูที่แสดง ตาม company effective menus

### 4.3 Phase 5 W4 — Mobile Path Extension (additive)

| File                                                                | สิ่งที่ต้องทำ                                                                                                                       |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `src/modules/v1/externalAuth/permissions.service.ts`                | `getPermissionsService` เพิ่ม `permissionMobile` field — call `resolveUserEffectivePermissionMobileService` (shared cache กับ web) |
| `src/modules/v1/externalAuth/permissions.interface.ts`              | `PermissionsResponse.permissionMobile: Record<string, unknown>` (additive — backward compat)                                        |
| `src/modules/v1/admin/compPermissionMapping/compPermissionMapping.service.ts` | `getEmployeePermissionService` BUG FIX + extend mobile path: filter `PermissionMobileDefault` ผ่าน `filterPermissionMobileByMenuKeysService` + `allowedMobileKeys` (เดิมใช้ web filter ผิด) |
| `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts` | เพิ่ม `resolveUserEffectivePermissionMobileService(userUuid, cache?)` mirror web — graceful `{}` ถ้า `permission_mobile = null` |
| `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts` | `getUserPermissionJsonbRepository` extend select `cp.permission_mobile` ใน round trip เดียว                                    |
| `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.interface.ts`  | `UserPermissionRow.permissionMobileJsonb: PermissionJsonb \| null`                                                              |

#### Service signatures (Phase 5 W4)

```typescript
// resolver.service.ts (mobile counterpart of web; graceful for null mobile JSONB)
export const resolveUserEffectivePermissionMobileService = async (
  userUuid: string,
  cache?: ResolverCache,
): Promise<PermissionJsonb>;
```

#### หลักการ Phase 5 W4

- **Backward compat**: `permissionMobile` เป็น additive field — old clients (web) ที่ไม่อ่าน → no regression
- **Cache sharing**: caller สร้าง 1 `Map<string, unknown>` ส่งให้ทั้ง web + mobile resolver → mobile call hit cache สำหรับ `featureIds:{companyId}` + `packageId:{companyId}` (saved 2 batch DB calls); เพิ่มแค่ 1 batch `getFeatureMobileMenuKeysRepository`
- **Empty fallback semantics**:
  - User ไม่มี mapping → `permissionMobile = {}` (mirrors web `permission`)
  - User มี mapping แต่ `permission_mobile = NULL` → `permissionMobile = {}` (graceful — admin ยังไม่ได้กำหนด; ไม่ใช่ error)
  - Company ไม่มี mobile menu keys → filter result `{}` (Q-C policy preserves)
- **Owner Q-A=STRICT**: `getEmployeePermissionService` mobile path บังคับ filter `PermissionMobileDefault` baseline ก่อน `deepMerge(isOwner=true)` → owner ไม่ bypass effective menu filter (parity กับ web)

### 4.4 Phase 5 W5 — compPermission admin BUG FIX + mobile validation

> **BUG ที่ fix:** Phase 4 W2 helper `assertPermissionKeysWithinCompanyMenusService` รวม keys ของ `permission` (web) + `permission_mobile` (mobile) แล้ว validate ผ่าน effective **web** menu keys อันเดียว → admin save mobile permission ที่มี mobile keys → reject 400 invalid key (หรือ pass บังเอิญ → save ผิด context).

| File                                                                | สิ่งที่ต้องทำ                                                                                                                       |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `src/modules/v1/admin/compPermission/compPermission.service.ts`     | Split `assertPermissionKeysWithinCompanyMenusService` เป็น 2 path: web ผ่าน `extractMenuKeys` + `resolveCompanyEffectiveMenuKeysService`, mobile ผ่าน `extractMobileMenuKeys` + `resolveCompanyEffectiveMobileMenuKeysService` (independent — web fail throws web error first; mobile fail throws mobile error second). `getCompPermissionByUuidService` ใช้ `filterPermissionMobileByMenuKeysService` + `allowedMobileKeys` สำหรับ mobile JSONB (เดิมใช้ web ผิด) |
| `src/api/v1/admin/compPermission/compPermission.controller.ts`      | `getCompPermissionByUuidController` resolve ทั้ง web + mobile menu keys (parallel, shared cache) → filter `PermissionDefault` ผ่าน web, filter `PermissionMobileDefault` ผ่าน mobile ก่อน `deepMerge`; `getCompPermissionDefaultListController` เปลี่ยนเดียวกัน |

#### Error codes (Phase 5 W5)

```typescript
// Web (existing — Phase 4 P4.4)
errorBadRequest(
  'ไม่สามารถบันทึกสิทธิ์ที่อยู่นอกเมนูที่บริษัทเปิดใช้งานได้',
  'INVALID_MENU_KEY_FOR_COMPANY',
  { invalidKeys: string[] },
);

// Mobile (new — Phase 5 W5 BUG FIX)
errorBadRequest(
  'ไม่สามารถบันทึกสิทธิ์มือถือที่อยู่นอกเมนูที่บริษัทเปิดใช้งานได้',
  'INVALID_MOBILE_MENU_KEY_FOR_COMPANY',
  { invalidKeys: string[] },
);
```

#### หลักการ Phase 5 W5

- **Independent path**: web validation + mobile validation ไม่ผูกกัน — web pass แต่ mobile fail → throws mobile error (admin เห็นได้ถึงแม้ web ถูก)
- **Cache sharing**: 1 `ResolverCache` ส่งให้ทั้ง 2 path → mobile resolver hit `featureIds`+`packageId` cache (saved 2 batch DB calls); เพิ่มแค่ 1 batch `getFeatureMobileMenuKeysRepository`
- **Early-return**: ถ้า payload `{}` (no leaf keys) → ไม่เรียก resolver (CR M1 carried forward)
- **Backward compat (mobile validation strictness)**: old clients ที่ไม่ส่ง `permission_mobile` → mobile path skip (no impact); old saves ที่มี invalid mobile keys → reject 400 (intended — bug correction)
- **Q-A=STRICT propagated to mobile**: `getCompPermissionByUuidController` filter `PermissionMobileDefault` baseline ก่อน `deepMerge(isOwner=true)` → owner ไม่ bypass mobile namespace filter
- **Error code distinct**: `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` ≠ `INVALID_MENU_KEY_FOR_COMPANY` → frontend สามารถแยก toast/inline message ได้

---

## 5. Authorization Middleware

### `requireSuperAdmin.middleware.ts` (สร้างใหม่)

```typescript
import { Request, Response, NextFunction } from "express";
import { AppError } from "@/utils/error";

export const requireSuperAdmin = (
  req: Request,
  _res: Response,
  next: NextFunction,
) => {
  if (req.user?.role !== "super_admin") {
    throw new AppError("ไม่มีสิทธิ์เข้าถึง", 403, 403);
  }
  next();
};
```

ใช้ใน routes ทั้งหมดของ feature management

---

## 6. Constants Update

### `src/constant/compPermission.ts`

เพิ่ม helper:

```typescript
/**
 * คืน leaf paths ทั้งหมดจาก PermissionDefault (Dev + Prod รวมกัน, dedupe)
 * ใช้ใน feature CRUD UI สำหรับ dropdown ของ menu_keys
 */
export const getAvailableMenuKeys = (): string[] => {
  const result = new Set<string>();

  const walk = (obj: Record<string, any>, prefix = "") => {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value !== "object" || value === null) continue;
      const path = prefix ? `${prefix}.${key}` : key;
      // ถ้า value มี read/create/etc → ถือเป็น leaf
      if ("read" in value || "create" in value) {
        result.add(path);
      } else {
        walk(value, path);
      }
    }
  };

  walk(PermissionDefaultDev);
  walk(PermissionDefaultProd);
  return Array.from(result).sort();
};
```

---

## 7. Route Registration

อัพเดต `src/api/v2/index.routes.ts` (หรือ root routes) เพื่อ register sub-routers:

```typescript
import featureRouter from "@/api/v2/feature/feature.routes";
import packageCrudRouter from "@/api/v2/packageCrud/packageCrud.routes";
import addonRouter from "@/api/v2/addon/addon.routes";
import companyFeatureRouter from "@/api/v2/companyFeature/companyFeature.routes";

router.use("/features", featureRouter);
router.use("/packages", packageCrudRouter);
router.use("/addons", addonRouter);
router.use("/companies", companyFeatureRouter); // /companies/:companyUuid/features
```

---

## 8. Reference Pattern (จาก request module)

โครงสร้างแบบ `src/modules/v1/employee/request/*`:

- `request.interface.ts` — Zod schemas + TS types + enums สำหรับ module
- `request.repository.ts` — DB operations เท่านั้น (Objection.js queries)
- `request.service.ts` — business logic, orchestrate repositories
- `request.controller.ts` (ใน api dir) — handle req/res, validate, call service, format response
- `request.routes.ts` (ใน api dir) — Express Router with middleware chain
- ไฟล์เสริมตาม responsibility: `leaveCalculation.service.ts`, `viewerStatus.util.ts` (สำหรับ logic เฉพาะ)

**Pattern ที่ apply กับ feature management:**

- `feature.interface.ts` — types + Zod schemas
- `feature.repository.ts` — query
- `feature.service.ts` — orchestration
- `feature.adapter.ts` — response normalization (เช่น hide id, convert to camelCase)
- ใน controller: ใช้ `res.success(data, 0, 'message')` + `handleError(res, error)` ตาม AI-RULES-BACKEND.md

---

## 9. Testing

### Unit Tests

- Service-level: mock repository, ทดสอบ logic
  - `resolveCompanyEffectiveFeatureIdsService` — combine package/addon/override correctly
  - `filterPermissionByMenuKeysService` — recursive walk ของ JSONB ถูกต้อง
  - `createFeatureService` — validate menu_keys ที่ไม่มีใน PermissionDefault → reject
  - `deleteFeatureService` — block เมื่อมี usage

### Integration Tests

- ใช้ test DB จริง (Knex migrations apply)
- CRUD flow ครบ: create feature → create package → link features → create addon → link features → create company override → call resolver → ตรวจ effective
- API e2e: super_admin token call CRUD endpoints → verify response shape

### Regression Tests สำหรับ Phase 4

- Test `getPermissionsController` กับ company ที่ feature ถูก disable → permission JSONB ตอบกลับไม่มี key นั้น
- Test กับ company ที่ feature ถูก enable แต่ override `disabled` → ไม่มีเมนูนั้น
- Test กับ company ที่มี addon → menu ที่ addon ปลด unlock ปรากฏ
