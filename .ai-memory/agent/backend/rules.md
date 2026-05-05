# Backend Agent — Rules

> **Role:** เขียน backend code ตาม SSD + task ที่ Lead assign

---

## Bootstrap (อ่านก่อนทุกครั้ง)

1. `.ai-memory/rules.md`
2. ไฟล์นี้
3. `.ai-memory/agent/backend/skills.md`
4. `.ai-memory/agent/backend/current-task.md`
5. `_docs/requirement/{feature}/ssd.md` (สำหรับ task ที่กำลังทำ)
6. `_docs/requirement/{feature}/summary.md` — ตรวจ Files Reviewed (สำหรับงานระยะยาว)

---

## Responsibilities

### 1. Implement Backend ตาม SSD

- API endpoints ตาม `ssd.md` §4
- Data model / DB schema ตาม §3
- Business logic ตาม `prd.md` §6
- Error handling ตาม §7

### 2. Pre-implementation Checks

ก่อนเริ่มเขียน code ทุก task:

- [ ] อ่าน `current-task.md` รู้ scope
- [ ] อ่าน `ssd.md` ส่วนที่เกี่ยวข้อง
- [ ] **ตรวจ "Files Reviewed" ใน summary.md ครอบคลุมไฟล์ที่จะแก้ไหม** — ไม่ครอบคลุม → แจ้ง Lead
- [ ] อ่านไฟล์ที่จะแก้ก่อนแก้ (Read tool)
- [ ] พิจารณา test plan (จะเขียน unit test เลยหรือให้ QA เขียน)

### 3. Coding Standards

ตาม master rules ข้อ 5:

- Functional + modular
- Explicit TS types, ห้าม `any`
- Pure function, immutable
- Error handling: try/catch + log + cleanup + rethrow
- ห้ามใช้ `logger.debug()`
- รัน `prettier --write` หลังเขียน

### 4. Update Document on Code Change

ถ้าระหว่าง dev พบว่าต้องเปลี่ยนจาก SSD:

- ระบุการเปลี่ยนใน `current-task.md`
- รายงาน Lead → Lead จะตัดสินว่าต้องเปิด CR ไหม
- **ห้ามเปลี่ยน SSD เอง**

### 5. Update Status

- อัพเดต `current-task.md` ทุก checkpoint
- Mark task `done` ใน `_docs/requirement/{feature}/task-list.md` (Lead จะ sync ให้)

---

## Backend Architecture (Default Pattern)

> **หมายเหตุ:** กฏชุดนี้เป็น **default pattern** ของ workspace อิงจาก stack Express.js + Objection.js + Knex + Zod
> ถ้าโปรเจคใดใช้ stack ต่าง agent ต้องแจ้ง Lead เพื่อปรับ rules ของ project นั้นเป็นการเฉพาะ

### Project Structure

```
src/
├── api/v1/                       # kebab-case folder (ตรงกับ route path)
│   └── {module-name}/
│       ├── {moduleName}.controller.ts
│       └── {moduleName}.routes.ts
├── modules/v1/                   # camelCase folder
│   └── {moduleName}/
│       ├── {moduleName}.repository.ts
│       ├── {moduleName}.service.ts
│       └── {moduleName}.interface.ts
├── configs/
├── database/
├── middlewares/
├── utils/
├── schemas/
├── constants/
└── typings/
```

### Layer Responsibilities

| Layer          | หน้าที่                                         | ข้อห้าม                          |
| -------------- | ----------------------------------------------- | -------------------------------- |
| **Controller** | Data prep ก่อน/หลัง service + return response   | ❌ ห้ามมี query / business logic |
| **Service**    | Orchestrate repository + business rules         | ❌ ห้ามมี query                  |
| **Repository** | Query เท่านั้น — รับ params ยืดหยุ่นเพื่อ reuse | ❌ ห้ามมี business logic         |
| **Interface**  | Type / Interface ใช้ใน service + repository     | —                                |

### Repository File Placement (by Main Model)

- เมื่อสร้าง function ใน repository → **ดู model หลักของ query** แล้วสร้างใน folder ที่ชื่อตรงกับ model นั้น
- ไม่มี folder ตรงกับ model → **สร้าง folder ใหม่**
- **ห้ามวาง function ใน folder ที่เรียกใช้** (แต่ไม่ตรงกับ model หลัก)

---

## API Response Standard

### Response Shape (ใช้รูปแบบเดียวทั้งโปรเจค)

```typescript
{
  code: number;
  msg: string;
  data: T | null;
  error?: object;
}
```

### Rules

- ใช้ `res.success()` สำหรับ success response เท่านั้น
- ใช้ `handleError()` สำหรับ error handling เท่านั้น
- **ห้ามใช้ `res.json()` โดยตรงใน controller**

---

## Database (Objection.js / Knex)

### Query Rules

- ใช้ `.query()` สำหรับ SELECT / INSERT / UPDATE / DELETE ทั่วไป
- ใช้ `.knex()` สำหรับ aggregation (COUNT / SUM / AVG + GROUP BY)
- เมื่อใช้ `.knex()` → ต้อง `as` alias เป็น **camelCase** (ผลลัพธ์ raw จะเป็น snake_case)
- โปรเจคใช้ `knexSnakeCaseMappers()` แล้ว → **ไม่ต้อง manual map**

### Connection Pool

- **ห้ามเปิด DB connection ใหม่ภายใน loop**
- ใช้ `WHERE IN` สำหรับ batch queries
- ใช้ transaction สำหรับ multiple dependent queries
- ใช้ `p-limit` เพื่อจำกัด concurrency

### Constants & Status

```typescript
// แนะนำ: ใช้ enum constants
.where('status_type', StatusTypeEnumCode.ACTIVE)

// หลีกเลี่ยง: hardcode ตัวเลข
.where('status', 1)
```

---

## Schema Validation (Zod)

- ทุก module ต้องมี Zod schemas ใน `src/schemas/`
- ทุก schema ต้อง register OpenAPI
- Response schema **ห้ามรวม `id` field** — ใช้เฉพาะ `uuid`

---

## Environment Variables

- เข้าถึงผ่าน `config/` เท่านั้น
- **ห้ามใช้ `process.env` โดยตรง** ใน business code

---

## Multilingual Data Design

> **Workspace นี้เปิดทั้ง 2 pattern** ให้แต่ละโปรเจคเลือกตอนเริ่ม
> **ภายในโปรเจคเดียวกันห้ามผสม pattern**

### Patterns ที่อนุญาต

**1. JSONB Pattern** (เหมาะ schema ง่าย, locale น้อย):

```sql
name JSONB  -- {"th": "ข้อความ", "en": "Text"}
```

**2. Separate Table Pattern** (เหมาะ schema ซับซ้อน, locale เยอะ):

```sql
products(id, ...)
product_locales(product_id, language_code, name, description)
```

### Rules ที่ใช้กับทุกโปรเจค (ไม่ขึ้นกับ pattern)

- การตัดสินใจ pattern ต้องบันทึกใน `_docs/requirement/{feature}/ssd.md` หรือ project-level decision doc
- ❌ ห้าม duplicate columns เช่น `name_th`, `name_en`
- ❌ ห้าม hardcode default language ใน database
- ❌ ห้าม derive display text แบบ implicit
- ❌ ห้าม nested JSONB structures ที่ซับซ้อน
- Language selection / fallback / default → จัดการที่ **application layer** เสมอ

---

## File Reading Strategy (Backend)

> ขยายจาก master rules §9 — สำหรับ backend code โดยเฉพาะ

| ถ้างานคือ...       | อ่านเฉพาะ                                                          |
| ------------------ | ------------------------------------------------------------------ |
| แก้ business logic | `*.service.ts` + `*.interface.ts` ของ module นั้น                  |
| แก้ query          | `*.repository.ts` + `*.interface.ts` ของ module นั้น               |
| แก้ API endpoint   | `*.controller.ts` + `*.routes.ts` + `*.service.ts` ของ module นั้น |
| แก้ validation     | `src/schemas/{module}/` ที่เกี่ยวข้อง                              |
| ดู data model      | `src/database/postgresql/models/` เฉพาะ model ที่ใช้               |

**ห้ามอ่าน module อื่นที่ไม่เกี่ยวข้อง** — ยกเว้น shared module ที่ถูก import โดยตรง

---

## Model Selection

> ดู master rules §11 สำหรับกฏร่วม

- **Default: Opus 4.7 (MANDATORY)** — งาน Backend คือ coding (controller / service / repository / business logic)
- **กฏ §11.1 บังคับใช้ Opus เมื่อเขียน/แก้ code ทุกกรณี**
- ❌ **ห้ามใช้ Sonnet หรือ Haiku** — แม้ task เล็กแค่ไหนก็ตาม (เช่น แก้ 1 บรรทัดใน service)
- ข้อยกเว้น (ใช้ Sonnet ได้): เฉพาะตอนอ่าน document หรือสรุปสถานะ ที่ไม่มีการเขียน code

---

## Constraints

- **ห้ามแตะ frontend code** (เป็นหน้าที่ Frontend)
- **ห้ามแตะ infra/deploy config** (เป็นหน้าที่ DevOps) — เว้น Dockerfile ของ service ตัวเอง
- **ห้าม implement ส่วนที่ไม่ได้อยู่ใน task** ของตัวเอง
- **ห้ามแก้ไฟล์โดยไม่อ่านก่อน**
- **ห้ามแก้ DB schema โดยไม่มี migration script**

---

## Output Quality Checklist

ก่อน mark task done:

- [ ] code ผ่าน type check
- [ ] code format ด้วย prettier แล้ว
- [ ] error handling ครบทุก path
- [ ] ไม่มี `console.log` / `logger.debug` ค้าง
- [ ] ถ้ามี API ใหม่ → ตรง spec ใน ssd.md
- [ ] unit test (ถ้าเขียนเอง) pass
- [ ] update `current-task.md` แล้ว
- [ ] response ใช้ `res.success()` / `handleError()` (ไม่ใช่ `res.json()` ตรงๆ)
- [ ] ID เป็น string, response มี `uuid` (ไม่มี `id` ภายใน)
- [ ] Zod schema ลงทะเบียน OpenAPI แล้ว
- [ ] ไม่มี DB connection ใหม่ใน loop
- [ ] ไม่ใช้ `process.env` โดยตรง — ใช้ผ่าน `config/`
- [ ] ใช้ `date-fns` v4.0+ ถ้าทำงานกับ date/time
