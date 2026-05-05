---
feature: { feature-name }
type: feature-backend-detail
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: SA (initial), Backend Dev (implementation)
status: draft
---

# {Feature Name} — Backend Detail

> รายละเอียด backend ทั้งหมด: routes, controllers, services, repositories, models
> Reference pattern: `{path/ไปยัง/module ที่ใช้เป็นแม่แบบ}`

---

## 0. โครงสร้างโดยรวม (Folder Structure)

> โครงสร้าง folder ทั้งหมดที่ feature นี้จะสร้าง/แก้ไข — สำหรับ dev ใช้เป็น checklist เปิดไฟล์

```
src/
├── api/{version}/{namespace}/
│   └── {moduleName}/
│       ├── {moduleName}.routes.ts
│       └── {moduleName}.controller.ts
│
├── modules/{version}/{namespace}/
│   └── {moduleName}/
│       ├── {moduleName}.interface.ts
│       ├── {moduleName}.repository.ts
│       ├── {moduleName}.service.ts
│       └── {moduleName}.adapter.ts
│
├── database/postgresql/
│   ├── migrations/data/        # ดู migration.md
│   └── models/data/
│       └── {newModelName}.model.ts
│
├── middlewares/
│   └── {newMiddlewareName}.middleware.ts
│
└── constant/
    └── {fileName}.ts           # ถ้ามี helper ใหม่
```

---

## 1. API Routes

> ระบุทุก endpoint ที่ feature นี้จะ expose

### 1.1 {Module A} — `/api/{version}/{namespace}/{resource}`

| Method | Path                | Controller          | Description       |
| ------ | ------------------- | ------------------- | ----------------- |
| GET    | `/{resource}`       | `getXxxxController` | List ...          |
| GET    | `/{resource}/:uuid` | `getXxxxByUuid...`  | Get by uuid       |
| POST   | `/{resource}`       | `createXxx...`      | สร้างใหม่         |
| PUT    | `/{resource}/:uuid` | `updateXxx...`      | แก้ไข             |
| DELETE | `/{resource}/:uuid` | `deleteXxx...`      | Soft-delete หรือ… |

**Middleware ทุก route:** `validateAccessToken` → `{authMiddleware}` → `validateRequest(zodSchema)`

### 1.2 {Module B} — `/api/{version}/{namespace}/{resource2}`

(เหมือน 1.1 — list ทุก endpoint)

---

## 2. Modules (Business Logic)

> 1 module = 1 sub-folder ใน `src/modules/{version}/{namespace}/`
> ทุก module มี `interface.ts`, `repository.ts`, `service.ts`, `adapter.ts` (ตามที่จำเป็น)

### 2.1 `modules/{version}/{namespace}/{moduleName}/`

#### `{moduleName}.interface.ts`

```typescript
import { z } from "zod";

export const {moduleName}BaseSchema = z.object({
  // ...
});

export const create{ModuleName}Schema = z.object({
  body: {moduleName}BaseSchema,
});

export const update{ModuleName}Schema = z.object({
  params: z.object({ {entity}Uuid: z.string().uuid() }),
  body: {moduleName}BaseSchema.partial(),
});

export interface {ModuleName} {
  uuid: string;
  // ...
}
```

#### `{moduleName}.repository.ts` (functions)

- `get{ModuleName}ByIdRepository(id: string): Promise<{ModuleName}Row | null>`
- `get{ModuleName}ByUuidRepository(uuid: string): Promise<{ModuleName}Row | null>`
- `list{ModuleName}sRepository(filter, pagination): Promise<{rows, total}>`
- `create{ModuleName}Repository(data, createdBy): Promise<{ModuleName}Row>`
- `update{ModuleName}Repository(uuid, data, updatedBy): Promise<{ModuleName}Row>`
- `softDelete{ModuleName}Repository(uuid, archivedBy): Promise<void>`

#### `{moduleName}.service.ts` (functions)

- `get{ModuleName}ListService(filter, pagination)`
- `get{ModuleName}ByUuidService(uuid)`
- `create{ModuleName}Service(input, createdBy)` — validate uniqueness, business rules
- `update{ModuleName}Service(uuid, input, updatedBy)`
- `delete{ModuleName}Service(uuid, archivedBy)`

#### `{moduleName}.adapter.ts`

- `convert{ModuleName}ToResponse(row)` — แปลง snake_case → camelCase, hide `id`
- `normalize{ModuleName}ListResponse(rows, totalCount, page, limit)`

### 2.2 `modules/{version}/{namespace}/{moduleName2}/`

(เหมือน 2.1 ต่อให้ครบทุก module)

### 2.x Shared Module (ถ้ามี)

> เช่น resolver, validator, helper ที่ใช้ข้าม module

---

## 3. Database Models (Objection.js)

> ทุก model define: `tableName`, `idColumn`, `jsonSchema`, `relationMappings`, TS interface

### `{newModelName}.model.ts`

```typescript
export class {NewModel} extends Model {
  static tableName = '{table_name}';
  static idColumn = 'id';

  id!: string;
  uuid!: string;
  // ... fields ตาม schema

  static relationMappings = {
    {relationName}: {
      relation: Model.{RelationType},
      modelClass: () => {OtherModel},
      join: {
        from: '{table}.{column}',
        to: '{otherTable}.{column}',
      },
    },
  };
}
```

(ทำซ้ำสำหรับทุก model ที่ feature นี้สร้างใหม่)

---

## 4. Updates ของ Existing API (ถ้ามี)

> List ไฟล์ที่ feature นี้ต้องไปแก้ + สิ่งที่ต้องเปลี่ยน

| File              | สิ่งที่ต้องทำ                                |
| ----------------- | -------------------------------------------- |
| `path/to/file.ts` | เพิ่ม call ไปยัง service ใหม่ / เปลี่ยน flow |

### หลักการ

> เขียนหลักคิดของ change เพื่อให้ดู rationale ได้

---

## 5. Authorization Middleware

> ถ้า feature ต้องการ middleware ใหม่

### `{newMiddlewareName}.middleware.ts`

```typescript
import { Request, Response, NextFunction } from "express";
import { AppError } from "@/utils/error";

export const {
  middlewareName,
} = (req: Request, _res: Response, next: NextFunction) => {
  // logic
  next();
};
```

ใช้ใน routes ทั้งหมดของ feature นี้

---

## 6. Constants Update (ถ้ามี)

### `src/constant/{fileName}.ts`

เพิ่ม helper / enum / config:

```typescript
export const {
  newHelper,
} = () => {
  // ...
};
```

---

## 7. Route Registration

อัพเดต `src/api/{version}/{namespace}/index.routes.ts` (หรือ root routes):

```typescript
import { moduleARouter } from "@/api/{version}/{namespace}/{moduleA}/{moduleA}.routes";

router.use("/{resource}", moduleARouter);
```

---

## 8. Reference Pattern

> module ที่ใช้เป็นแม่แบบ (ระบุ path ชัดเจน + อธิบาย pattern ที่ adapt มา)

โครงสร้างแบบ `{reference-module-path}`:

- `{moduleName}.interface.ts` — Zod schemas + TS types + enums
- `{moduleName}.repository.ts` — DB operations (Objection.js queries)
- `{moduleName}.service.ts` — business logic, orchestrate repositories
- `{moduleName}.controller.ts` — handle req/res, validate, call service, format response
- `{moduleName}.routes.ts` — Express Router with middleware chain

**Pattern ที่ apply กับ feature นี้:**

- _(สรุปว่าใช้ pattern เดิม + จุดที่ต่างเพราะอะไร)_

---

## 9. Testing

> Test ที่ backend ต้องเขียน — ใน checklist.md จะมี ID อ้างอิงกลับมาที่ section นี้

### Unit Tests

- Service-level: mock repository, ทดสอบ logic
  - `{serviceName}` — ทดสอบเคส X, Y, Z

### Integration Tests

- ใช้ test DB จริง (Knex migrations apply)
- ครอบคลุม: CRUD flow, edge case, transaction

### Regression Tests (ถ้า feature แก้ existing API)

- ทดสอบ API เดิมว่าไม่กระทบ

---

## 10. Open Questions / TBD

> รายการคำถามที่ยังไม่ตอบ — ต้องเคลียร์ก่อน implement

| #   | คำถาม | ตอบแล้ว? | คำตอบ |
| --- | ----- | -------- | ----- |
| 1   |       |          |       |
