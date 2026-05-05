# Backend Agent — Skills

---

## Core Skills

### 1. API Design & Implementation

- RESTful endpoint design
- Request/response validation (Zod / Joi / Yup)
- Status codes ถูกต้องตาม HTTP semantics
- Error response แบบ consistent

### 2. Database Operations

- Query design (avoid N+1)
- Transaction handling
- Index awareness
- Migration script (up + down)
- ORM usage (Prisma / TypeORM / Objection ฯลฯ)

### 3. Business Logic

- แยก domain logic จาก infrastructure
- Pure function ทำธุรกรรม
- Compose ผ่าน function ไม่ใช่ inheritance

### 4. Authentication & Authorization

- Session / JWT handling
- Role-based / attribute-based access control
- Secure token storage

### 5. Error Handling

```typescript
try {
  const result = await someOperation();
  return result;
} catch (error) {
  logger.error('เกิดข้อผิดพลาด:', {
    operation: 'someOperation',
    error: error instanceof Error ? error.message : 'ไม่ทราบสาเหตุ',
    stack: error instanceof Error ? error.stack : undefined,
  });
  await cleanupResources();
  throw error;
}
```

### 6. Testing

- Unit test ด้วย Jest / Vitest
- Mock dependency (อย่า mock DB ถ้าเป็น integration test)
- Coverage โฟกัส branch ที่สำคัญ ไม่ต้อง 100%

---

## Tool Preferences

| Task | Tool |
|------|------|
| อ่านไฟล์ก่อนแก้ | Read |
| แก้ไฟล์ | Edit (preferred) / Write (สร้างใหม่) |
| ค้น API ที่ existing | Grep |
| รัน test | Bash |
| รัน prettier | Bash |

---

## Anti-patterns

- ❌ ใช้ `any` type
- ❌ Silent catch (catch แล้วไม่ทำอะไร)
- ❌ N+1 query (loop call DB)
- ❌ ใช้ class + this เมื่อ function ทำได้
- ❌ Mock DB ใน integration test
- ❌ Skip migration script เมื่อแก้ schema
- ❌ Hardcode secret / config — ใช้ env
