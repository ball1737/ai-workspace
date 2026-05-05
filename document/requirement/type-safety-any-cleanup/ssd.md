---
feature: type-safety-any-cleanup
type: system-specification-document
created: 2026-04-29
updated: 2026-04-29
owner: SA
status: in-progress
---

# System Specification Document — Type Safety Any Cleanup

## 1. Architecture Overview

`mof-backend/src` จะคง architecture เดิม Controller → Service → Repository → DB แต่เพิ่ม type contracts ให้ชัดขึ้นระหว่าง layer

## 2. Affected Modules / Services

| Module / Service | Path | Type of change | Note |
|------------------|------|----------------|------|
| PostgreSQL models | `src/database/postgresql/models` | modify | Add one exported model type per class |
| Shared typings | `src/typings`, `src/types` | modify/new | Replace `any` with reusable unknown-safe types |
| Middleware | `src/middlewares` | modify | Typed request/response/auth/upload contracts |
| API routes/controllers | `src/api/v1` | modify | Remove route/controller `as any` casts |
| Repositories | `src/modules/v1` | modify | Add specific row interfaces for raw/join results |
| Utilities | `src/utils` | modify | Replace `any` in helpers/http/error/openapi utilities |

## 3. Data Model

No DB schema changes.

### 3.3 Entities / Domain Models

Every model class exports one type:

```typescript
import type { ModelObject } from "objection";

export class SystemUser extends BaseModel {
  // model fields
}

export type SystemUserType = ModelObject<SystemUser>;
```

## 4. API Specification

No API behavior or response shape changes.

## 5. Implementation Details

- Add shared JSON/object types where code currently uses `Record<string, any>`
- Use `unknown` for error/input surfaces that require runtime narrowing
- Use local row interfaces for Knex result rows with aliases
- Replace route-level `as any` with a typed handler helper or compatible `RequestHandler`
- Enable `@typescript-eslint/no-explicit-any` only after production source cleanup passes

## 6. Error Handling

No runtime error behavior changes. Existing `handleError` and response handler behavior remains.

## 7. Testing / Verification

- `npm run lint -- --no-cache`
- `npx tsc --noEmit --pretty false`
- Targeted Jest suites for auth/documentLibrary/task/typed request utilities where possible
- Final full Jest with `DISABLE_RUNTIME_OPENAPI=true`; baseline existing failures must be reported separately
