---
type: requirement-doc-rules
created: 2026-05-04
updated: 2026-05-04
status: active
---

# กฏการทำเอกสาร Requirement (A + B Combined)

> เอกสารนี้กำหนดว่าทุก feature ใหม่ใน `document/requirement/{feature-name}/` ต้องประกอบด้วยไฟล์อะไรบ้าง
> รูปแบบนี้รวม **ชุด A** (decision & measurement) + **ชุด B** (implementation & execution) เพื่อ coverage 100% ตั้งแต่ business จนถึง deployment

---

## 1. ไฟล์ที่ต้องมีครบทุก feature

ทุก feature folder ต้องมีอย่างน้อย **12 ไฟล์** ตามตารางนี้:

| #   | ไฟล์             | กลุ่ม | Audience หลัก         | สิ่งที่อยู่ในไฟล์                                                    |
| --- | ---------------- | ----- | --------------------- | -------------------------------------------------------------------- |
| 1   | `prompt.md`      | B     | ทุก agent             | Raw prompt + grill clarifications + ref                              |
| 2   | `summary.md`     | A+B   | ทุก agent (bootstrap) | Current state + planned changes + decisions + risks + files reviewed |
| 3   | `requirement.md` | A     | BA, SA, QA            | Functional + non-functional + business rules                         |
| 4   | `prd.md`         | A     | PO, PM, BA            | User stories + success metrics + UX flow                             |
| 5   | `ssd.md`         | A     | SA, Architect, Lead   | Solution architecture + API spec + DB schema                         |
| 6   | `backend.md`     | B     | Backend Dev           | Routes, modules, models, middleware, tests                           |
| 7   | `frontend.md`    | B     | Frontend Dev          | Pages, components, store, RBAC, i18n                                 |
| 8   | `migration.md`   | B     | Backend Dev, DevOps   | Migration sequence + rollback + verify                               |
| 9   | `task-list.md`   | A     | PM, Lead              | High-level phased task breakdown                                     |
| 10  | `checklist.md`   | B     | All devs (continuity) | Checkbox tracker + file paths                                        |
| 11  | `testcase.md`    | A     | QA                    | Unit / integration / e2e test cases                                  |
| 12  | `cr.md`          | A     | PM, PO                | Change request log (scope changes)                                   |

> **ไฟล์ที่ optional:** `ba-validation.md`, `qa-report.md`, `handoff/{date}-{topic}.md` — สร้างเมื่อจำเป็น

---

## 2. ลำดับการสร้างเอกสาร (Order of Operations)

ห้ามสร้างไฟล์ปลายทางโดยข้ามต้นทาง — ทุกไฟล์ build จากไฟล์ก่อนหน้า

```
1. prompt.md        ← capture raw prompt ทันทีที่ user สั่ง
2. summary.md       ← SA เขียนหลังอ่าน code base ครบ (Files Reviewed!)
3. requirement.md   ← BA/SA แปลง user intent → requirement
4. prd.md           ← PO/BA เขียน user stories + success metrics
5. ssd.md           ← Architect/SA ออกแบบ solution + schema
6. backend.md       ← SA/Backend Dev แตก ssd → implementation detail
7. frontend.md      ← SA/Frontend Dev แตก ssd → implementation detail
8. migration.md     ← SA/Backend Dev แตก ssd → migration steps
9. task-list.md     ← Lead/PM แปลง backend+frontend+migration → phased plan
10. checklist.md    ← Lead แปลง task-list → checkbox tracker (path-linked)
11. testcase.md     ← QA แปลง requirement+prd → test cases
12. cr.md           ← PO/PM track change request หลังเริ่ม dev
```

> **Bootstrap rule:** เมื่อเริ่ม session ใหม่ — อ่าน `prompt.md` → `summary.md` → `checklist.md` ก่อนเสมอ

---

## 3. กฏหลักในการเขียน

### 3.1 ไฟล์ทุกไฟล์ต้องมี frontmatter

```yaml
---
feature: { feature-name }
type: { doc-type }
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: { agent-role }
status: { draft | in-review | approved | active }
---
```

### 3.2 ใช้ภาษาไทยเป็นหลัก

- Heading + คำอธิบายเป็นภาษาไทย
- Code, path, identifier, technical term เก็บภาษาอังกฤษ
- Code comment ภายใน example code อาจเป็นภาษาไทย

### 3.3 Naming Convention

- Folder: `document/requirement/{feature-name}/` ใช้ kebab-case
- File: ใช้ชื่อสั้น **ไม่มี prefix** (เช่น `backend.md` ไม่ใช่ `{feature-name}.backend.md`)
  เพราะอยู่ใน folder ของ feature นั้นแล้ว
- Migration ID: `M1`, `M2`, ... ใน migration.md
- Task ID: `P1.1`, `P2.3`, `F2.5`, `F3.10` ใน checklist.md (P=Backend Phase, F=Frontend)

### 3.4 Cross-link ทุกไฟล์ที่อ้างอิง

- จาก `summary.md` → link ไป `backend.md`, `frontend.md`, `migration.md`, `checklist.md`
- จาก `checklist.md` → ทุกข้อระบุ **file path** ที่จะสร้าง/แก้
- จาก `backend.md` / `frontend.md` → อ้าง task ID จาก `checklist.md`

### 3.5 อัพเดต Sync เสมอ

- แก้ code → อัพเดต `summary.md` (Progress Log) + `checklist.md` (checkbox)
- เพิ่ม scope → เปิด entry ใหม่ใน `cr.md` + อัพเดต `requirement.md` / `task-list.md`
- เปลี่ยน decision → อัพเดต `summary.md` (Architecture Decisions table)

---

## 4. ความสัมพันธ์ระหว่างไฟล์

```
prompt.md (raw input)
    ↓
summary.md  ←──────────────────────────────────┐
    ↓ (high-level overview, link out)          │
    ├──→ requirement.md (functional spec)      │
    ├──→ prd.md (business spec)                │ (Progress Log
    └──→ ssd.md (solution design) ─────┐       │  อัพเดตตอน
                                       ↓       │  checkpoint จบ)
                          ┌────────────┼────────┐
                          ↓            ↓        ↓
                    backend.md   frontend.md  migration.md
                          ↓            ↓        ↓
                          └────────┬───┴────────┘
                                   ↓
                              task-list.md (phased plan)
                                   ↓
                              checklist.md (path-linked checkboxes)
                                   ↓
                          ┌────────┴────────┐
                          ↓                 ↓
                     testcase.md         cr.md
                     (QA verify)         (scope log)
```

- `summary.md` = single source of truth สำหรับภาพรวม → ทุกไฟล์อื่นเป็น detail ของ summary
- `checklist.md` = single source of truth สำหรับ progress → ห้าม track progress ที่อื่น

---

## 5. Audience Matrix (ใครอ่านอะไร)

| Role               | ต้องอ่าน                                                                              | อาจอ่าน                                     |
| ------------------ | ------------------------------------------------------------------------------------- | ------------------------------------------- |
| **PO / PM**        | `prd.md`, `summary.md`, `cr.md`, `task-list.md`                                       | `requirement.md`                            |
| **BA**             | `requirement.md`, `prd.md`, `summary.md`                                              | `ssd.md`, `cr.md`                           |
| **SA / Architect** | `summary.md`, `requirement.md`, `ssd.md`, `backend.md`, `frontend.md`, `migration.md` | ทุกไฟล์                                     |
| **Backend Dev**    | `summary.md`, `backend.md`, `migration.md`, `checklist.md`, `ssd.md`                  | `requirement.md`, `testcase.md`             |
| **Frontend Dev**   | `summary.md`, `frontend.md`, `checklist.md`, `ssd.md`                                 | `prd.md`, `requirement.md`                  |
| **DevOps**         | `migration.md`, `summary.md`                                                          | `backend.md`                                |
| **QA**             | `testcase.md`, `requirement.md`, `prd.md`, `summary.md`                               | `checklist.md`, `backend.md`, `frontend.md` |
| **Lead**           | ทุกไฟล์                                                                               | -                                           |

---

## 6. Coverage Checklist สำหรับ SA

> SA ต้องเช็คก่อนปิด design phase ว่าครอบคลุมครบ

### Business

- [ ] User stories (US-1, US-2, ...) ใน `prd.md`
- [ ] Success metrics ใน `prd.md`
- [ ] Out-of-scope ระบุชัดใน `prd.md` หรือ `summary.md`
- [ ] Open Questions ระบุ + ตอบ (หรือ TBD) ใน `requirement.md`

### Functional

- [ ] Functional requirements (FR-1, FR-2, ...) ใน `requirement.md`
- [ ] Non-functional requirements (NFR) ใน `requirement.md`
- [ ] Business rules (BR-1, BR-2, ...) ใน `requirement.md`

### Technical

- [ ] API spec (method + path + auth + body + response) ใน `ssd.md` และ `backend.md`
- [ ] DB schema (table + column + index + constraint) ใน `ssd.md` และ `migration.md` (พร้อม SQL/Knex code)
- [ ] Module structure (folder + file path) ใน `backend.md` และ `frontend.md`
- [ ] Existing API/file ที่ต้องแก้ ระบุใน `backend.md` / `frontend.md`

### Migration

- [ ] Migration sequence ใน `migration.md` พร้อม Knex code จริง (ไม่ใช่ pseudocode)
- [ ] Pre-migration validation script
- [ ] Rollback strategy (`down()` function ครบ)
- [ ] Post-migration verification queries

### Tasks & Tracking

- [ ] Phased task list ใน `task-list.md`
- [ ] Checkbox tracker ใน `checklist.md` พร้อม file path ทุกข้อ
- [ ] Task ID อ้างอิงข้ามไฟล์ได้

### Tests

- [ ] Test cases (unit + integration + e2e) ใน `testcase.md`
- [ ] Unit test tracker ใน `checklist.md` (table at the bottom)

### Decision & Risk

- [ ] Architecture decisions table ใน `summary.md` พร้อม **เหตุผล**
- [ ] Risks & Mitigations table ใน `summary.md`

---

## 7. กฏการสร้าง folder ใหม่

เมื่อเริ่ม feature ใหม่:

```bash
# 1. สร้าง folder
mkdir -p document/requirement/{feature-name}

# 2. Copy template ทุกไฟล์
cp document/requirement/_template/*.md document/requirement/{feature-name}/

# 3. Rename / แทนที่ placeholder ในแต่ละไฟล์
#    - แทน {feature-name} ด้วยชื่อจริง
#    - แทน {YYYY-MM-DD} ด้วยวันที่ปัจจุบัน
#    - แทน {agent-role} ด้วย role ที่เป็นเจ้าของ
```

> **อย่า copy `_rules.md` และ `_template/` เข้าไปใน folder feature** — สองไฟล์นี้เป็น meta files

---

## 8. Anti-patterns ที่ห้ามทำ

- ❌ ใช้ prefix ชื่อไฟล์ซ้ำกับชื่อ folder (เช่น `feature-management.backend.md` ใน folder `feature-management/`)
- ❌ เขียน implementation detail ใน `summary.md` (ให้อยู่ใน backend/frontend/migration เท่านั้น)
- ❌ track progress ใน `task-list.md` (ใช้ `checklist.md` checkbox แทน)
- ❌ แก้ code โดยไม่อัพเดต `checklist.md` checkbox + `summary.md` Progress Log
- ❌ สร้าง doc ใหม่นอก folder feature โดยไม่ link จาก `summary.md`
- ❌ ใส่ raw SQL ใน `summary.md` (ให้อยู่ใน `migration.md`)
- ❌ ใส่ React code ใน `requirement.md` (ให้อยู่ใน `frontend.md`)

---

## 9. Lifecycle ของ feature

| State          | trigger                      | ไฟล์ที่ status เปลี่ยน         |
| -------------- | ---------------------------- | ------------------------------ |
| `draft`        | SA เริ่มเขียน                | summary, requirement, prd, ssd |
| `in-review`    | ส่งให้ Lead/PO review        | summary, requirement, prd      |
| `approved`     | Lead/PO อนุมัติ → เริ่ม dev  | summary, requirement, prd, ssd |
| `in-progress`  | Dev เริ่มทำตาม checklist     | checklist (in-progress)        |
| `ready-for-qa` | Dev ทำครบทุก checkbox        | checklist, testcase            |
| `done`         | QA pass + deploy             | summary (Final Outcome filled) |
| `archived`     | ปิด feature, move ไป archive | ทุกไฟล์                        |

---

## 10. ตัวอย่าง folder ที่สมบูรณ์

ดู `document/requirement/feature-management/` เป็นตัวอย่างที่ครบทั้ง 12 ไฟล์
