---
feature: { feature-name }
type: feature-frontend-detail
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: SA (initial), Frontend Dev (implementation)
status: draft
---

# {Feature Name} — Frontend Detail

> รายละเอียด frontend ทั้งหมด: pages, components, store, RBAC, i18n
> Reference pattern: `{path/ไปยัง/page หรือ section ที่ใช้เป็นแม่แบบ}`

---

## 0. โครงสร้างโดยรวม (Folder Structure)

> โครงสร้าง folder ทั้งหมดที่ feature นี้จะสร้าง/แก้ไข

```
src/
├── app/{routeRoot}/                # Routes (App Router)
│   └── {feature-name}/
│       ├── page.tsx                # List
│       ├── create/page.tsx         # Create
│       └── [id]/
│           ├── page.tsx            # Detail
│           └── edit/page.tsx       # Edit
│
├── sections/                       # Section views + components
│   └── {feature-name}/
│       ├── {feature}-list-view.tsx
│       ├── {feature}-create-view.tsx
│       ├── {feature}-detail-view.tsx
│       ├── {feature}-edit-view.tsx
│       ├── {feature}.enum.ts
│       ├── components/
│       │   ├── {feature}-table.tsx
│       │   ├── {feature}-form.tsx          # ใช้ทั้ง create + edit
│       │   └── {feature}-delete-dialog.tsx
│       └── utils/
│           └── {feature}-form.ts            # Yup schema + default values
│
├── components/                     # Reusable across modules
│   └── {feature-name}/
│       └── {sharedComponent}.tsx
│
├── store/                          # Redux
│   ├── actions/{store-slice-name}.ts
│   ├── reducers/{store-slice-name}.ts
│   ├── sagas/{store-slice-name}.ts
│   └── services/{feature}-request.ts
│
├── types/store/
│   └── {store-slice-name}.ts
│
├── utils/
│   └── rbac.ts                     # เพิ่ม resource ใหม่
│
├── routes/
│   └── paths.ts                    # เพิ่ม path constants
│
├── layouts/{layoutName}/
│   └── config-navigation.tsx       # เพิ่มเมนูถ้ามี
│
└── locales/langs/
    ├── en.json                     # เพิ่ม section
    └── th.json
```

---

## 1. Pages

> Pages เป็นแค่ wrapper ที่ render view component (Next.js App Router pattern)

### 1.1 {Feature Name} — Page Group A

#### `/{routeRoot}/{feature-name}` (List)

- **File:** `src/app/{routeRoot}/{feature-name}/page.tsx`
- **Section:** `src/sections/{feature-name}/{feature}-list-view.tsx`
- **Component:**
  - `{feature}-table.tsx` — MUI Table with sort/filter/pagination
  - Columns: ...
  - Actions: View / Edit / Delete

#### `/{routeRoot}/{feature-name}/create` (Create)

- **File:** `src/app/{routeRoot}/{feature-name}/create/page.tsx`
- **Section:** `{feature}-create-view.tsx`
- **Component:** `{feature}-form.tsx` (shared with edit)
- **Fields:** ...
- **Validation:** ...
- **Submit:** API POST → redirect to detail page

#### `/{routeRoot}/{feature-name}/[id]` (Detail)

- **File:** `src/app/{routeRoot}/{feature-name}/[id]/page.tsx`
- **Section:** `{feature}-detail-view.tsx`
- **Component:** display fields, actions Edit/Delete

#### `/{routeRoot}/{feature-name}/[id]/edit` (Edit)

- **File:** `src/app/{routeRoot}/{feature-name}/[id]/edit/page.tsx`
- **Section:** `{feature}-edit-view.tsx`
- **Component:** `{feature}-form.tsx` (same as create, with initial values)

### 1.2 {Page Group B} (ถ้ามี)

(ทำเหมือน 1.1 ต่อให้ครบทุกกลุ่มเพจ)

### 1.x ปรับเพจที่มีอยู่แล้ว (ถ้ามี)

- เช่น เพิ่ม tab ใหม่ในเพจที่มีอยู่
- ระบุ file ที่ต้องแก้ + จุดที่เปลี่ยน

---

## 2. Reusable Components

> Component ที่ใช้ข้ามเพจ → อยู่ใน `src/components/{feature-name}/`

### `{sharedComponent}.tsx`

- **Props:** `prop1: type`, `prop2: type`
- **Behavior:** อธิบายสั้นๆ
- **Usage:** ระบุว่าใช้ที่ไหน

---

## 3. Redux Store

> โครงสร้าง slice ตาม pattern ที่ project ใช้อยู่ (action/reducer/saga/service/hook)

### 3.1 Actions (`actions/{store-slice-name}.ts`)

```typescript
export const Actions = {
  get{Resource}sRequest: createAction<GetParams>('GET_RESOURCES_REQUEST'),
  get{Resource}sSuccess: createAction<{ list: Resource[]; pagination: Pagination }>('GET_RESOURCES_SUCCESS'),
  get{Resource}sFailure: createAction<string>('GET_RESOURCES_FAILURE'),
  // ... CRUD actions อื่นๆ
};
```

### 3.2 Reducer (`reducers/{store-slice-name}.ts`)

```typescript
interface State {
  list: Resource[];
  pagination: Pagination;
  current: Resource | null;
  isLoading: boolean;
  isSubmitting: boolean;
  error: string | null;
}
```

### 3.3 Saga (`sagas/{store-slice-name}.ts`)

- `takeLatest(getResourcesRequest, ...)` → call service → dispatch success/failure
- `takeLatest(createResourceRequest, ...)` → call service → notify success → navigate to detail
- ... etc

### 3.4 Service (`services/{feature}-request.ts`)

```typescript
export const { feature }Request = {
  getList: (params) =>
    axios.get(`${API_ENDPOINT}/api/{version}/{path}`, { params }),
  getByUuid: (uuid) => axios.get(`${API_ENDPOINT}/api/{version}/{path}/${uuid}`),
  create: (data) => axios.post(`${API_ENDPOINT}/api/{version}/{path}`, data),
  update: (uuid, data) =>
    axios.put(`${API_ENDPOINT}/api/{version}/{path}/${uuid}`, data),
  delete: (uuid) => axios.delete(`${API_ENDPOINT}/api/{version}/{path}/${uuid}`),
};
```

### 3.5 Hook

```typescript
export const use{Feature} = () => {
  const dispatch = useDispatch();
  const state = useSelector((s: TStore.RootState) => s.{sliceName});

  return {
    ...state,
    get{Resource}s: (params) => dispatch(Actions.get{Resource}sRequest(params)),
    // ... action helpers
  };
};
```

(ทำซ้ำสำหรับทุก slice ที่ feature นี้ต้องการ)

---

## 4. RBAC Update

### `src/utils/rbac.ts`

เพิ่ม resources และ permissions:

```typescript
export type Resource =
  | "existing_resource_1"
  // ...
  | "{new_resource}";

export const PERMISSIONS: Record<Role, Record<Resource, PermissionAction[]>> = {
  super_admin: {
    // ...
    {new_resource}: ["view", "create", "edit", "delete", "manage"],
  },
  admin: {
    {new_resource}: [], // หรือ permission ที่เหมาะสม
  },
  // ...
};
```

---

## 5. Navigation Menu Update (ถ้ามี)

### `src/layouts/{layoutName}/config-navigation.tsx`

```typescript
{
  subheader: t("menu.{section_name}"),
  items: [
    {
      title: t("menu.{feature_name}"),
      path: paths.{routeRoot}.{featureName}.root,
      icon: ICONS.{iconName},
      // ใส่ conditional ถ้าจำกัดสิทธิ์ (เช่น super_admin only)
    },
  ],
},
```

### `src/routes/paths.ts`

```typescript
export const paths = {
  {routeRoot}: {
    // ...
    {featureName}: {
      root: "/{routeRoot}/{feature-name}",
      create: "/{routeRoot}/{feature-name}/create",
      detail: (id: string) => `/{routeRoot}/{feature-name}/${id}`,
      edit: (id: string) => `/{routeRoot}/{feature-name}/${id}/edit`,
    },
  },
};
```

---

## 6. i18n

### `src/locales/langs/en.json` (และ th.json)

```json
{
  "menu": {
    "{section_name}": "...",
    "{feature_name}": "..."
  },
  "{featureKey}": {
    "list_title": "...",
    "create_title": "...",
    "edit_title": "...",
    "form": {
      "field1": "...",
      "field1_helper": "..."
    },
    "validation": {}
  }
}
```

---

## 7. Reference Pattern

> Page/section ที่ใช้เป็นแม่แบบ — อ้าง path ชัดเจน

### Pattern จาก `{reference-section-path}`:

- `{ref}-list-view.tsx` — list view component หลัก, page.tsx แค่ render view
- `{ref}-create-view.tsx`, `{ref}-detail-view.tsx`, `{ref}-edit-view.tsx`
- `components/` — sub-components ของ section
- `utils/{ref}-form.ts` — Yup schema + default values
- `{ref}.enum.ts` — enum/constants

### Page wrapping pattern:

```typescript
// page.tsx
import { {Feature}ListView } from "@/sections/{feature-name}/{feature}-list-view";

export default function {Feature}Page() {
  return <{Feature}ListView />;
}
```

---

## 8. Form Library

ใช้ {React Hook Form + Yup / หรือไลบรารี่ที่ project ใช้อยู่}:

- `FormProvider` wrap form
- `RHFTextField`, `RHFSelect`, `RHFSwitch` เป็น hook-form controllers
- Yup schema validation
- `useForm({ resolver: yupResolver(schema), defaultValues })`

---

## 9. UAT Test Checklist (สรุปสั้น — รายละเอียดอยู่ใน testcase.md / checklist.md)

- เปิดหน้าเป็น role ที่มีสิทธิ์ → เห็นเมนู / เพจ
- เปิดหน้าเป็น role ที่ไม่มีสิทธิ์ → ไม่เห็น / 403
- CRUD ครบ (create / read / update / delete)
- Validation ครบ (required, format, uniqueness)
- Edge case (empty list, network error, conflict)

---

## 10. Open Questions / TBD

| #   | คำถาม | ตอบแล้ว? | คำตอบ |
| --- | ----- | -------- | ----- |
| 1   |       |          |       |
