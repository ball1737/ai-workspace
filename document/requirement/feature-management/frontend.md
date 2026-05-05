---
feature: feature-management
type: feature-frontend-detail
created: 2026-05-04
updated: 2026-05-04
owner: SA (initial), Frontend Dev (implementation)
status: draft
---

# Feature Management — Frontend Detail

> รายละเอียด frontend ทั้งหมด: pages, components, store
> Reference pattern: `happywork-sale-cms/src/app/dashboard/survey/*` + `src/sections/survey/*`
> (Pattern Next.js App Router แยก page list/create/[id]/edit แทน dialog)

---

## โครงสร้างโดยรวม

```
src/
├── app/dashboard/                              # Routes (App Router)
│   ├── feature-management/
│   │   ├── page.tsx                            # List page
│   │   ├── create/page.tsx                     # Create page
│   │   └── [id]/
│   │       ├── page.tsx                        # Detail page
│   │       └── edit/page.tsx                   # Edit page
│   ├── package-management/
│   │   ├── page.tsx
│   │   ├── create/page.tsx
│   │   └── [id]/
│   │       ├── page.tsx                        # Detail (รวม manage features)
│   │       └── edit/page.tsx
│   ├── addon-management/
│   │   ├── page.tsx
│   │   ├── create/page.tsx
│   │   └── [id]/
│   │       ├── page.tsx
│   │       └── edit/page.tsx
│   └── client-management/[uuid]/               # Existing
│       └── (เพิ่ม tab "Features" ใน detail view)
│
├── sections/                                   # Section views + components
│   ├── feature-management/
│   │   ├── feature-list-view.tsx
│   │   ├── feature-create-view.tsx
│   │   ├── feature-detail-view.tsx
│   │   ├── feature-edit-view.tsx
│   │   ├── feature.enum.ts
│   │   ├── components/
│   │   │   ├── feature-table.tsx
│   │   │   ├── feature-form.tsx                # ใช้ทั้ง create + edit
│   │   │   ├── feature-delete-dialog.tsx
│   │   │   ├── feature-status-option.tsx
│   │   │   └── feature-usage-info.tsx          # แสดง package/addon ที่ใช้
│   │   └── utils/
│   │       └── feature-form.ts                  # Yup schema + default values
│   │
│   ├── package-management/
│   │   ├── package-list-view.tsx
│   │   ├── package-create-view.tsx
│   │   ├── package-detail-view.tsx
│   │   ├── package-edit-view.tsx
│   │   ├── package.enum.ts
│   │   ├── components/
│   │   │   ├── package-table.tsx
│   │   │   ├── package-form.tsx
│   │   │   ├── package-delete-dialog.tsx
│   │   │   ├── package-features-section.tsx    # อยู่ใน detail view
│   │   │   ├── package-features-edit-dialog.tsx
│   │   │   └── package-effective-menus-preview.tsx
│   │   └── utils/
│   │       └── package-form.ts
│   │
│   ├── addon-management/
│   │   ├── addon-list-view.tsx
│   │   ├── addon-create-view.tsx
│   │   ├── addon-detail-view.tsx
│   │   ├── addon-edit-view.tsx
│   │   ├── addon.enum.ts
│   │   ├── components/
│   │   │   ├── addon-table.tsx
│   │   │   ├── addon-form.tsx
│   │   │   ├── addon-delete-dialog.tsx
│   │   │   ├── addon-features-section.tsx
│   │   │   ├── addon-features-edit-dialog.tsx
│   │   │   └── addon-quantifiable-fields.tsx   # is_quantifiable + max_quantity (conditional)
│   │   └── utils/
│   │       └── addon-form.ts
│   │
│   └── client-management/                      # Existing — เพิ่ม tab
│       └── feature-tab/
│           ├── company-features-tab-view.tsx
│           └── components/
│               ├── company-feature-list.tsx
│               ├── company-feature-row.tsx     # 1 row ต่อ feature: name + source badge + toggle
│               ├── company-feature-toggle-confirm-dialog.tsx
│               └── company-feature-source-filter.tsx
│
├── components/                                  # Reusable across modules
│   └── feature-management/
│       ├── menu-keys-select.tsx                # Tree-style multi-select
│       ├── menu-tree-preview.tsx               # Read-only tree preview
│       ├── feature-source-badge.tsx
│       └── multilingual-text-field.tsx         # TH/EN paired field
│
├── store/                                       # Redux
│   ├── actions/sale-dashboard-feature-management.ts
│   ├── actions/sale-dashboard-package-management.ts
│   ├── actions/sale-dashboard-addon-management.ts
│   ├── actions/sale-dashboard-company-features.ts
│   ├── reducers/sale-dashboard-feature-management.ts (และ 3 ตัวคู่ขนาน)
│   ├── sagas/sale-dashboard-feature-management.ts (และ 3 ตัวคู่ขนาน)
│   ├── services/feature-management-request.ts
│   ├── services/package-management-request.ts
│   ├── services/addon-management-request.ts
│   ├── services/company-features-request.ts
│   └── hooks (อยู่ใน respective slice เดียวกัน)
│
├── types/store/
│   ├── sale-dashboard-feature-management.ts
│   ├── sale-dashboard-package-management.ts
│   ├── sale-dashboard-addon-management.ts
│   └── sale-dashboard-company-features.ts
│
├── utils/
│   └── rbac.ts                                 # เพิ่ม resources: features, packages, addons, company_features
│
├── routes/
│   └── paths.ts                                # เพิ่ม path constants
│
├── layouts/dashboard/
│   └── config-navigation.tsx                   # เพิ่มเมนู Master Data section
│
└── locales/langs/
    ├── en.json                                 # เพิ่ม section featureManagement, packageManagement, addonManagement, companyFeatures
    └── th.json
```

---

## 1. Pages

### 1.1 Feature Management

#### `/dashboard/feature-management` (List)

- **File:** `src/app/dashboard/feature-management/page.tsx`
- **Section:** `src/sections/feature-management/feature-list-view.tsx`
- **Component:**
  - `feature-table.tsx` — MUI Table with sort/filter/pagination, columns: code, name, menu_keys count, packages using, addons using, status, actions
  - Search box, status filter dropdown, "Create Feature" button → navigate to `/dashboard/feature-management/create`
  - Row action: View → `/dashboard/feature-management/[id]`, Edit → `/dashboard/feature-management/[id]/edit`, Delete → confirm dialog

#### `/dashboard/feature-management/create` (Create)

- **File:** `src/app/dashboard/feature-management/create/page.tsx`
- **Section:** `feature-create-view.tsx`
- **Component:** `feature-form.tsx` (shared with edit)
  - Fields: `code`, `name (TH/EN)`, `description (TH/EN)`, `menuKeys` (multi-select tree), `sortOrder`, `statusType`
  - Validation: code matches `^[a-z0-9_]+$`, code uniqueness (verify on blur via API)
  - Submit → API POST → redirect to detail page

#### `/dashboard/feature-management/[id]` (Detail)

- **File:** `src/app/dashboard/feature-management/[id]/page.tsx`
- **Section:** `feature-detail-view.tsx`
- **Component:**
  - Display: code, name, description, menuKeys (with tree preview), sortOrder, status
  - `feature-usage-info.tsx` — list packages/addons ที่ผูก feature นี้
  - Actions: Edit, Delete

#### `/dashboard/feature-management/[id]/edit` (Edit)

- **File:** `src/app/dashboard/feature-management/[id]/edit/page.tsx`
- **Section:** `feature-edit-view.tsx`
- **Component:** `feature-form.tsx` (same as create, with initial values)

### 1.2 Package Management

#### `/dashboard/package-management` (List)

- ตาราง package พร้อม columns: code, name, billing, price, user_limit, feature count, status

#### `/dashboard/package-management/create`

- Form: code, name, shortName, description, billingInterval, priceAmount, currency, userLimitMin/Max, isRecommend, isContactUs, isActive, sortOrder

#### `/dashboard/package-management/[id]` (Detail)

- ข้อมูล package ทั้งหมด
- **Section "Features in this Package":** `package-features-section.tsx`
  - List feature ทั้งหมดของ package
  - Button "Edit Features" → เปิด `package-features-edit-dialog.tsx` (เลือก feature multi-select)
  - Save → call `PUT /packages/:uuid/features`
- **Section "Effective Menus Preview":** `package-effective-menus-preview.tsx`
  - แสดง menu tree ที่ package นี้ปลดล็อก (รวม menu_keys ของทุก feature) — read-only

#### `/dashboard/package-management/[id]/edit`

- Form แก้ metadata เฉพาะ (ไม่รวม features — features จัดการที่ detail)

### 1.3 Addon Management

#### `/dashboard/addon-management` (List)

- ตาราง addon: code, name, billing (month/year/oneTime), price, is_quantifiable, max_quantity, feature count, status

#### `/dashboard/addon-management/create`

- Form ใช้ `addon-form.tsx` ซึ่งรวม `addon-quantifiable-fields.tsx`:
  - Switch `isQuantifiable` — ถ้าเปิด → แสดง field `maxQuantity` (required)
  - ถ้าปิด → hide `maxQuantity`, set NULL

#### `/dashboard/addon-management/[id]` (Detail)

- ข้อมูล addon
- **Section "Features Unlocked":** `addon-features-section.tsx`
  - List feature ที่ addon ปลดล็อก
  - Button "Edit Features" → `addon-features-edit-dialog.tsx`

#### `/dashboard/addon-management/[id]/edit`

- Form แก้ metadata

### 1.4 Client Detail — Features Tab

#### Existing: `/dashboard/client-management/[uuid]`

- เพิ่ม tab ใหม่ "Features"
- **Section:** `src/sections/client-management/feature-tab/company-features-tab-view.tsx`
- **Component:**
  - `company-feature-source-filter.tsx` — filter dropdown: All, From Package, From Addon, Override Enabled, Override Disabled, Default Disabled
  - `company-feature-list.tsx` — list ทุก feature ของระบบ
  - `company-feature-row.tsx` — แต่ละ row:
    - Feature name + description
    - Source badge: `feature-source-badge.tsx` (Package / Addon / Override / Default)
    - Toggle switch (current state)
    - "Reset to default" button (ถ้ามี override) → call DELETE override
  - `company-feature-toggle-confirm-dialog.tsx` — เปิดเมื่อ toggle, รับ `reason` (optional)

---

## 2. Reusable Components

### `src/components/feature-management/menu-keys-select.tsx`

- **Props:** `value: string[]`, `onChange: (value: string[]) => void`, `availableKeys: string[]`, `disabled?: boolean`
- **Behavior:**
  - Fetch available keys จาก API `/features/menu-keys/available` (cached)
  - Render เป็น tree (group by main category, leaf เป็น checkbox)
  - Multi-select tree-style
  - Search box
- **Usage:** Feature form, package detail (read-only mode)

### `src/components/feature-management/menu-tree-preview.tsx`

- **Props:** `menuKeys: string[]`, `availableKeys: string[]`
- **Behavior:** Render tree ของ menu structure แบบ readonly (highlight ที่ enabled)
- **Usage:** Package detail "Effective Menus", Addon detail "Effective Menus"

### `src/components/feature-management/feature-source-badge.tsx`

- **Props:** `source: 'package' | 'addon' | 'override-enabled' | 'override-disabled' | 'default-disabled'`
- **Behavior:** Render Chip MUI สีต่างกันตาม source

### `src/components/feature-management/multilingual-text-field.tsx`

- **Props:** `name: string` (สำหรับ react-hook-form), `label: string`, `multiline?: boolean`
- **Behavior:** Render TextField คู่ TH/EN พร้อม flag/label, register ผ่าน react-hook-form
- **Usage:** ทุก form ที่มี name/description multilingual

---

## 3. Redux Store

> **Q7 Resolution (2026-05-05):** ชื่อไฟล์ + reducer slot + hook + saga watcher ของ Master Data slices ใช้ตาม spec ด้านล่าง (`sale-dashboard-{feature,package,addon}-management`, `useSaleDashboard{Feature,Package,Addon}Management`) — legacy slice เดิมที่ใช้ชื่อนี้ถูก rename เป็น `sale-dashboard-package-config-feature` ใน Wave Q7 (pure rename, no logic change). 4 importer views ของ legacy (`package-config-feature-view`, `edit-package-dialog`, `lead-information-tab`, `customer-information-tab`) update slot key + namespace ref แล้ว
>
> **Q8 Resolution (2026-05-05):** `AddonBillingInterval` + `PackageBillingInterval` ใช้ค่า `'month' | 'year'` (string literal union, nullable) — match backend B3 Zod `z.enum(['month','year']).nullable()`. `null` = one-time billing. Form select 3 options: One-time (form value `""` → wire `null`), Monthly (`"month"`), Yearly (`"year"`). i18n key `addonManagement.billing.one_time` คงไว้เป็น display label ของ null (semantic preservation)


โครงสร้างคล้าย `sale-dashboard-admin-users.ts` ที่มีอยู่แล้ว

### 3.1 Actions (`actions/sale-dashboard-feature-management.ts`)

```typescript
export const Actions = {
  sdGetFeaturesRequest: createAction<GetFeaturesParams>('SD_GET_FEATURES_REQUEST'),
  sdGetFeaturesSuccess: createAction<{ list: Feature[]; pagination: Pagination }>('SD_GET_FEATURES_SUCCESS'),
  sdGetFeaturesFailure: createAction<string>('SD_GET_FEATURES_FAILURE'),

  sdGetFeatureByUuidRequest: createAction<{ uuid: string }>(...),
  sdGetFeatureByUuidSuccess: createAction<Feature>(...),

  sdCreateFeatureRequest: createAction<CreateFeaturePayload>(...),
  sdUpdateFeatureRequest: createAction<{ uuid: string; data: UpdateFeaturePayload }>(...),
  sdDeleteFeatureRequest: createAction<{ uuid: string }>(...),

  sdGetAvailableMenuKeysRequest: createAction<void>(...),
  sdGetAvailableMenuKeysSuccess: createAction<string[]>(...),
};
```

### 3.2 Reducer (`reducers/sale-dashboard-feature-management.ts`)

```typescript
interface State {
  list: Feature[];
  pagination: Pagination;
  current: Feature | null;
  availableMenuKeys: string[];
  isLoading: boolean;
  isSubmitting: boolean;
  error: string | null;
}
```

### 3.3 Saga (`sagas/sale-dashboard-feature-management.ts`)

- `takeLatest(sdGetFeaturesRequest, ...)` → call service → dispatch success/failure
- `takeLatest(sdGetFeatureByUuidRequest, ...)`
- `takeLatest(sdCreateFeatureRequest, ...)` → call service → notify success → navigate to detail
- ... etc

### 3.4 Service (`services/feature-management-request.ts`)

```typescript
export const featureManagementRequest = {
  getList: (params) =>
    axios.get(`${HW_ENDPOINT}/api/v2/admin/features`, { params }),
  getByUuid: (uuid) =>
    axios.get(`${HW_ENDPOINT}/api/v2/admin/features/${uuid}`),
  create: (data) => axios.post(`${HW_ENDPOINT}/api/v2/admin/features`, data),
  update: (uuid, data) =>
    axios.put(`${HW_ENDPOINT}/api/v2/admin/features/${uuid}`, data),
  delete: (uuid) =>
    axios.delete(`${HW_ENDPOINT}/api/v2/admin/features/${uuid}`),
  getAvailableMenuKeys: () =>
    axios.get(`${HW_ENDPOINT}/api/v2/admin/features/menu-keys/available`),
};
```

### 3.5 Hook (สามารถอยู่ในไฟล์ reducer หรือแยก)

```typescript
export const useSaleDashboardFeatureManagement = () => {
  const dispatch = useDispatch();
  const state = useSelector(
    (s: TStore.RootState) => s.saleDashboardFeatureManagement,
  );

  return {
    ...state,
    getFeatures: (params) => dispatch(Actions.sdGetFeaturesRequest(params)),
    getFeatureByUuid: (uuid) =>
      dispatch(Actions.sdGetFeatureByUuidRequest({ uuid })),
    createFeature: (data) => dispatch(Actions.sdCreateFeatureRequest(data)),
    updateFeature: (uuid, data) =>
      dispatch(Actions.sdUpdateFeatureRequest({ uuid, data })),
    deleteFeature: (uuid) => dispatch(Actions.sdDeleteFeatureRequest({ uuid })),
    getAvailableMenuKeys: () =>
      dispatch(Actions.sdGetAvailableMenuKeysRequest()),
  };
};
```

ทำ pattern เดียวกันสำหรับ:

- `useSaleDashboardPackageManagement()` (CRUD + replaceFeatures)
- `useSaleDashboardAddonManagement()` (CRUD + replaceFeatures)
- `useSaleDashboardCompanyFeatures()` (getCompanyFeatures, setOverride, removeOverride)

---

## 4. RBAC Update

### `src/utils/rbac.ts`

> **Decision Q6 (2026-05-05):** Resource สำหรับ Master Data Package Management ใช้ชื่อ `master_packages` (ไม่ใช่ `packages`) เพราะ `packages` มีอยู่แล้วใน `rbac.ts` (pre-existing) ถูกใช้โดย `src/sections/package-config-feature/package-config-feature-view.tsx` — ห้ามแตะ existing `packages` เพื่อกัน break UI ที่ admin/manager/viewer ใช้อยู่

เพิ่ม resources และ permissions:

```typescript
export type Resource =
  | "admin_users" | "customers" | "leads" | /* existing — รวม `packages` ที่มีอยู่แล้ว */
  | "features"           // new (Phase 2)
  | "master_packages"    // new (Phase 2) — Master Data Package Management
  | "addons"             // new (Phase 2)
  | "company_features";  // new (Phase 2)

export const PERMISSIONS: Record<Role, Record<Resource, PermissionAction[]>> = {
  super_admin: {
    // ... existing (รวม `packages` เดิม)
    features: ["view", "create", "edit", "delete", "manage"],
    master_packages: ["view", "create", "edit", "delete", "manage"],
    addons: ["view", "create", "edit", "delete", "manage"],
    company_features: ["view", "edit", "manage"],
  },
  admin: {
    // ไม่ให้สิทธิ์ — Master Data เฉพาะ super_admin
    features: [],
    master_packages: [],
    addons: [],
    company_features: [],
  },
  // ... manager, viewer ก็ไม่ให้สิทธิ์ (`[]` ทั้ง 4 resources)
};
```

> **Implementation note:** Wave F1 (2026-05-05) เพิ่ม `features`, `addons`, `company_features` ครบแล้ว แต่ **ยังไม่ได้เพิ่ม `master_packages`** เพราะยังไม่ได้ตัดสิน Q6 — Wave ที่จะ implement Page B (Package Management UI) ต้องเพิ่ม `master_packages` resource เข้า `rbac.ts` ก่อน + ใน components ที่เรียกใช้ใช้ key `"master_packages"` (ไม่ใช่ `"packages"`)

---

## 5. Navigation Menu Update

### `src/layouts/dashboard/config-navigation.tsx`

เพิ่ม section "Master Data" (super_admin only):

```typescript
const isSuperAdmin = user?.role === "super_admin";

// ภายใน data array:
...(isSuperAdmin
  ? [
      {
        subheader: t("menu.master_data"),
        items: [
          {
            title: t("menu.feature_management"),
            path: paths.dashboard.featureManagement.root,
            icon: ICONS.feature,
          },
          {
            title: t("menu.package_management"),
            path: paths.dashboard.packageManagement.root,
            icon: ICONS.package,
          },
          {
            title: t("menu.addon_management"),
            path: paths.dashboard.addonManagement.root,
            icon: ICONS.addon,
          },
        ],
      },
    ]
  : []),
```

### `src/routes/paths.ts`

```typescript
export const paths = {
  dashboard: {
    // ... existing
    featureManagement: {
      root: "/dashboard/feature-management",
      create: "/dashboard/feature-management/create",
      detail: (id: string) => `/dashboard/feature-management/${id}`,
      edit: (id: string) => `/dashboard/feature-management/${id}/edit`,
    },
    packageManagement: {
      root: "/dashboard/package-management",
      create: "/dashboard/package-management/create",
      detail: (id: string) => `/dashboard/package-management/${id}`,
      edit: (id: string) => `/dashboard/package-management/${id}/edit`,
    },
    addonManagement: {
      root: "/dashboard/addon-management",
      create: "/dashboard/addon-management/create",
      detail: (id: string) => `/dashboard/addon-management/${id}`,
      edit: (id: string) => `/dashboard/addon-management/${id}/edit`,
    },
  },
};
```

---

## 6. i18n

### `src/locales/langs/en.json` (และ th.json)

เพิ่ม section ใหม่:

```json
{
  "menu": {
    "master_data": "Master Data",
    "feature_management": "Feature Management",
    "package_management": "Package Management",
    "addon_management": "Addon Management"
  },
  "featureManagement": {
    "list_title": "Features",
    "create_title": "Create Feature",
    "edit_title": "Edit Feature",
    "form": {
      "code": "Code",
      "code_helper": "lowercase letters, numbers, underscore only",
      "name_th": "Name (Thai)",
      "name_en": "Name (English)",
      "description_th": "Description (Thai)",
      "description_en": "Description (English)",
      "menu_keys": "Menus Unlocked",
      "menu_keys_helper": "Select menus that this feature unlocks",
      "sort_order": "Sort Order"
    },
    "delete_warning": "This feature is used by {{packageCount}} packages and {{addonCount}} addons. Remove the references first.",
    "validation": { /* ... */ }
  },
  "packageManagement": { /* similar */ },
  "addonManagement": {
    /* ... */ ,
    "is_quantifiable": "Allow Quantity Selection",
    "max_quantity": "Max Quantity"
  },
  "companyFeatures": {
    "tab_title": "Features",
    "source": {
      "package": "From Package",
      "addon": "From Addon",
      "override_enabled": "Override (Enabled)",
      "override_disabled": "Override (Disabled)",
      "default_disabled": "Default (Disabled)"
    },
    "toggle_confirm_title": "Toggle Feature",
    "toggle_confirm_reason_label": "Reason (optional)",
    "reset_to_default": "Reset to Default"
  }
}
```

---

## 7. Reference Pattern (จาก survey module)

### Pattern จาก `src/sections/survey/`:

- `survey-list-view.tsx` — list view เป็น component หลัก, page.tsx แค่ render view
- `survey-create-view.tsx`, `survey-detail-view.tsx`, `survey-edit-view.tsx` แยกชัด
- `components/` — sub-components ของ section นั้น
- `utils/survey-form.ts` — Yup schema + default values สำหรับ form
- `survey.enum.ts` — enum/constants ของ section

### Apply pattern กับ feature-management:

- `feature-list-view.tsx` ↔ `survey-list-view.tsx`
- `feature-create-view.tsx`, `feature-edit-view.tsx`, `feature-detail-view.tsx`
- `components/feature-form.tsx` ↔ Survey ไม่มี shared form แต่เราแยกได้
- `utils/feature-form.ts` — Yup schema
- `feature.enum.ts` — enum/constants

### Page wrapping pattern (จาก `src/app/dashboard/survey/page.tsx`):

```typescript
// page.tsx
import { SurveyListView } from "@/sections/survey/survey-list-view";

export default function SurveyPage() {
  return <SurveyListView />;
}
```

ใช้ pattern เดียวกัน — `page.tsx` เป็นแค่ wrapper, logic อยู่ใน `*-view.tsx`

---

## 8. Form Library

ใช้ React Hook Form + Yup (ตาม pattern survey + admin-users):

- `FormProvider` wrap form
- `RHFTextField`, `RHFSelect`, `RHFSwitch` เป็น hook-form controllers
- Yup schema validation
- `useForm({ resolver: yupResolver(schema), defaultValues })`

---

## 9. UAT Test Checklist (ดูเพิ่มใน checklist.md)

- เปิดด้วย super_admin → เห็นเมนู Master Data + 3 sub-menu
- เปิดด้วย admin/manager → ไม่เห็นเมนู
- CRUD feature ครบ (create/read/update/delete)
- Delete feature ที่มี package ผูก → block พร้อม error message
- CRUD package + manage features (replace)
- CRUD addon + manage features + is_quantifiable conditional field
- Open client detail → tab Features → toggle → re-open → state ตรง
- Toggle feature → save reason → reload → reason ปรากฏ
- Reset to default → row กลับไปใช้ default จาก package
- เพิ่ม addon ให้ company → feature ที่ addon ปลด unlock เพิ่มเข้ามาใน list → source = "From Addon"
