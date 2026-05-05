---
feature: {feature-name}
type: system-specification-document
created: {YYYY-MM-DD}
updated: {YYYY-MM-DD}
owner: SA
status: draft
---

# System Specification Document — {Feature Name}

> มุมมองทาง technical — ตอบคำถาม "ระบบต้องทำอะไร, ทำยังไง, มี API/DB/component อะไรบ้าง"

---

## 1. Architecture Overview

แสดง architecture diagram (text หรือ mermaid) ของส่วนที่กระทบ

```
[Frontend] ── HTTP ──> [Backend API] ── SQL ──> [DB]
```

## 2. Affected Modules / Services

| Module / Service | Path | Type of change | Note |
|------------------|------|----------------|------|
|                  |      | new / modify   |      |

## 3. Data Model

### 3.1 New Tables / Collections

```sql
-- ตัวอย่าง
CREATE TABLE example (
  id UUID PRIMARY KEY,
  ...
);
```

### 3.2 Modified Tables

| Table | Change | Migration script |
|-------|--------|------------------|
|       |        |                  |

### 3.3 Entities / Domain Models

```typescript
type Example = {
  readonly id: string;
  // ...
};
```

## 4. API Specification

### POST /api/v1/{resource}

- Description:
- Auth required: yes / no
- Request body:
  ```json
  {}
  ```
- Response (200):
  ```json
  {}
  ```
- Errors:
  | Code | Reason |
  |------|--------|

## 5. Frontend Components / Pages

| Component | Path | Purpose | Props |
|-----------|------|---------|-------|
|           |      |         |       |

## 6. Sequence / Flow

อธิบาย flow ของ data ระหว่าง component (text หรือ mermaid sequence diagram)

```
User → Frontend → Backend → DB → Backend → Frontend → User
```

## 7. Error Handling

| Scenario | Handling | Log level |
|----------|---------|-----------|
|          |         | error     |

## 8. Performance Considerations

- Expected load:
- Caching strategy:
- N+1 query risks:
- Index requirements:

## 9. Security Considerations

- Auth / authorization:
- Input validation:
- Sensitive data handling:
- Rate limiting:

## 10. Migration / Rollout Plan

- Backward compat:
- Migration steps:
- Rollback plan:
- Feature flag:

## 11. Testing Strategy

- Unit tests:
- Integration tests:
- E2E tests:
- Manual QA:

## 12. References

- Related code paths (full list — ดู `summary.md` "Files Reviewed"):
- External docs / RFCs:
