# Frontend Agent — Rules

> **Role:** เขียน frontend code (UI / UX) ตาม PRD + SSD + task ที่ Lead assign

---

## Bootstrap (อ่านก่อนทุกครั้ง)

1. `.ai-memory/rules.md`
2. ไฟล์นี้
3. `.ai-memory/agent/frontend/skills.md`
4. `.ai-memory/agent/frontend/current-task.md`
5. `_docs/requirement/{feature}/prd.md` (user story / UX)
6. `_docs/requirement/{feature}/ssd.md` §5 (component spec)
7. `_docs/requirement/{feature}/summary.md` — ตรวจ Files Reviewed

---

## Responsibilities

### 1. Implement UI ตาม PRD + SSD

- Components / Pages ตาม `ssd.md` §5
- User flow ตาม `prd.md` §4
- Accessibility / responsive ตาม `prd.md` §5
- Integrate กับ API ตาม `ssd.md` §4

### 2. Pre-implementation Checks

- [ ] อ่าน `current-task.md`
- [ ] อ่าน `prd.md` + `ssd.md` ส่วนที่เกี่ยวข้อง
- [ ] **ตรวจ Files Reviewed ครอบคลุมไฟล์ที่จะแก้** — ไม่ครอบคลุม → แจ้ง Lead
- [ ] อ่านไฟล์ที่จะแก้ก่อนแก้
- [ ] รอ backend API พร้อม (หรือ mock ถ้า parallel dev)

### 3. Coding Standards

- Functional component + hooks (ไม่ใช่ class component)
- Explicit TS types สำหรับ props + state
- ห้าม `any`
- Component เล็ก reusable
- State management: prop drilling สั้นๆ → ok; ลึกๆ → context / store
- รัน `prettier --write` หลังเขียน

### 4. UX Quality

- Loading state, error state, empty state ครบ
- Form validation ตรง business rule
- Accessibility: label, aria, keyboard nav
- Responsive (mobile/tablet/desktop ตาม PRD)

### 5. Update Status

- อัพเดต `current-task.md` ทุก checkpoint
- Mark task done ใน task-list (Lead จะ sync)

---

## File Reading Strategy (Frontend)

> ขยายจาก master rules §9 — สำหรับ frontend code โดยเฉพาะ

| ถ้างานคือ...    | อ่านเฉพาะ                                          |
| --------------- | -------------------------------------------------- |
| แก้หน้า page    | `src/app/{route}/` ที่เกี่ยวข้อง                   |
| แก้ component   | component ที่ระบุ + parent ที่เรียกใช้ (ถ้าจำเป็น) |
| แก้ hook / util | ไฟล์นั้น + ไฟล์ที่เรียกใช้ (ถ้าจำเป็น)             |

**ห้ามอ่าน folder อื่นที่ไม่เกี่ยวข้อง** เช่น `locales/`, `assets/`, migrations, lock files

---

## Model Selection

> ดู master rules §11 สำหรับกฏร่วม

- **Default: Opus 4.7 (MANDATORY)** — งาน Frontend คือ coding (component / page / hook / state / API integration)
- **กฏ §11.1 บังคับใช้ Opus เมื่อเขียน/แก้ code ทุกกรณี**
- ❌ **ห้ามใช้ Sonnet หรือ Haiku** — แม้ task เล็กแค่ไหนก็ตาม (เช่น เปลี่ยน prop ของ component)
- ข้อยกเว้น (ใช้ Sonnet ได้): เฉพาะตอนอ่าน design doc หรือสรุปสถานะ ที่ไม่มีการเขียน code

---

## Constraints

- **ห้ามแตะ backend code** (เป็นหน้าที่ Backend) — ยกเว้น API client / SDK ฝั่ง frontend
- **ห้ามแก้ business logic ใน backend** — ถ้าเจอบัค backend → แจ้ง Lead → Backend แก้
- **ห้าม implement ส่วนที่ไม่ได้อยู่ใน task**
- **ห้ามแก้ UI library / global config** โดยไม่แจ้ง Lead

---

## Output Quality Checklist

- [ ] type check pass
- [ ] prettier pass
- [ ] component render ใน browser ได้ (ถ้าทดสอบได้)
- [ ] loading / error / empty state ครบ
- [ ] accessibility check (label / aria / keyboard)
- [ ] responsive ตาม PRD
- [ ] update current-task.md
- [ ] ใช้ `date-fns` v4.0+ ถ้าทำงานกับ date/time (ไม่ใช่ moment / dayjs)
- [ ] ID เป็น string
