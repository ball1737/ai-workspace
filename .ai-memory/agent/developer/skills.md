# Developer Agent — Skills

> Skills ครอบคลุมทั้ง 2 tracks: backend + frontend — เลือกใช้ตาม Active Track ของ task

---

## Backend Track Skills

### 1. API Design & Implementation

- RESTful endpoint design
- Request/response validation (Zod / Joi / Yup)
- Status codes ถูกต้องตาม HTTP semantics
- Error response แบบ consistent
- OpenAPI registration (Zod → OpenAPI)

### 2. Database Operations

- Query design (avoid N+1)
- Transaction handling
- Index awareness
- Migration script (up + down)
- ORM usage (Prisma / TypeORM / Objection.js / Knex)
- Aggregation: `.knex()` + camelCase alias

### 3. Business Logic

- แยก domain logic จาก infrastructure
- Pure function ทำธุรกรรม
- Compose ผ่าน function ไม่ใช่ inheritance
- Service orchestrate repository — **ห้าม** มี query ใน service

### 4. Authentication & Authorization

- Session / JWT handling
- Role-based / attribute-based access control
- Secure token storage
- Middleware composition (e.g., `validateAccessToken → requireSuperAdmin`)

### 5. Error Handling

```typescript
try {
  const result = await someOperation();
  return result;
} catch (error) {
  logger.error("เกิดข้อผิดพลาด:", {
    operation: "someOperation",
    error: error instanceof Error ? error.message : "ไม่ทราบสาเหตุ",
    stack: error instanceof Error ? error.stack : undefined,
  });
  await cleanupResources();
  throw error;
}
```

### 6. Backend Testing

- Unit test ด้วย Jest / Vitest
- Mock dependency (อย่า mock DB ถ้าเป็น integration test)
- Coverage โฟกัส branch ที่สำคัญ ไม่ต้อง 100%

---

## Frontend Track Skills

### 1. Component Development

- Functional React / Vue / Svelte components
- Props typing แบบ strict
- Composition over inheritance
- Reusable atomic components

### 2. State Management

- Local state (`useState`) สำหรับ scope สั้น
- Context สำหรับ cross-component scope กลาง
- External store (Zustand / Redux / Pinia) สำหรับ scope ใหญ่
- Server state (React Query / SWR / TanStack Query)
- Redux saga: action → request → reducer

### 3. Routing

- File-based routing (Next.js App Router, Nuxt)
- Dynamic routes + catch-all
- Loading / error boundary
- Route-level guards (RBAC)

### 4. Styling

- CSS modules / Tailwind / Styled-components / MUI ตาม project convention
- Design token (color, spacing, font) ใช้ตาม system
- Responsive: mobile-first

### 5. Form Handling

- Validation library (Zod / Yup / React Hook Form)
- Server-side error display (surface backend rejection)
- Optimistic update เมื่อเหมาะสม

### 6. Accessibility

- Semantic HTML
- ARIA labels
- Keyboard navigation
- Color contrast

### 7. Performance

- Memoization (`useMemo`, `useCallback`) เมื่อจำเป็น (ห้าม premature)
- Code splitting / lazy load
- Image optimization
- Avoid unnecessary re-render

### 8. Internationalization (i18n)

- Locale files: en + th (หรือตาม project) — keys ต้อง parity
- Display label แยกจาก wire value (label ใน i18n, value ใน enum constant)
- Pluralization + interpolation ตาม library convention

---

## Cross-track Skills

### File Reading Memory (per master §15)

- ก่อน Read tool ทุกครั้ง → check `current-task.md §Files Read Memory`
- หลัง Read → append entry: path, timestamp, takeaways (1-3 sentences)
- หลัง Edit → mark `Edited: yes` + refresh takeaways
- ห้าม re-read ไฟล์ที่อยู่ใน memory + ยัง valid

### Track Selection (เริ่ม task)

- อ่าน task spec → ตัดสิน track (backend / frontend / both)
- บันทึกใน `current-task.md §Active Track`
- ถ้า both → backend ก่อน เสมอ

---

## Tool Preferences

| Task | Tool |
|------|------|
| อ่านไฟล์ก่อนแก้ | Read (หลัง check Files Read Memory) |
| แก้ไฟล์ | Edit (preferred) / Write (สร้างใหม่) |
| ค้น API / pattern ที่ existing | Grep / Glob |
| รัน test / prettier / type-check | Bash |
| Smoke test ใน browser (frontend) | Bash (start dev server) + manual verify |

---

## Anti-patterns (รวม backend + frontend)

### Backend
- ❌ ใช้ `any` type
- ❌ Silent catch (catch แล้วไม่ทำอะไร)
- ❌ N+1 query (loop call DB)
- ❌ ใช้ `class` + `this` เมื่อ function ทำได้
- ❌ Mock DB ใน integration test
- ❌ Skip migration script เมื่อแก้ schema
- ❌ Hardcode secret / config — ใช้ env ผ่าน `config/`
- ❌ ใช้ `res.json()` แทน `res.success()` / `handleError()`
- ❌ Response ส่ง `id` ภายใน (ต้องส่งแค่ `uuid`)

### Frontend
- ❌ Class component ในของใหม่
- ❌ `any` type ใน props/state
- ❌ Inline business logic (ควรใส่ใน hook / util)
- ❌ Skip loading / error state
- ❌ Skip accessibility
- ❌ Premature memoization
- ❌ Direct DOM manipulation (ใช้ ref ถ้าจำเป็นจริง)
- ❌ Hardcode wire value ใน UI (ใช้ enum constant)
- ❌ i18n key drift (en มี key แต่ th ไม่มี หรือกลับกัน)

### Cross-cutting
- ❌ Re-read ไฟล์ที่อยู่ใน Files Read Memory + valid (เปลือง token)
- ❌ ใช้ Reporting Format เต็มในทุก task (ใช้เฉพาะ trigger ตาม master §16)
- ❌ ทำงาน frontend ก่อน backend ในงาน `both` track (API contract ยังไม่ stable)
- ❌ ใช้ `moment.js` / `dayjs` แทน `date-fns` v4.0+
- ❌ ใช้ `number` สำหรับ ID
