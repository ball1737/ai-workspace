---
feature: feature-management
type: change-request-log
created: 2026-05-04
updated: 2026-05-04
---

# Change Request Log — Feature Management

> บันทึกการเปลี่ยนแปลง requirement / scope ที่เกิดขึ้นระหว่างทำ feature
> ทุก change ต้องมี CR record ที่นี่ และ SA ต้องอัพเดต requirement / prd / ssd ตามด้วย

---

## CR Format

```
### CR-NNN: {Short Title}
- Date: YYYY-MM-DD
- Requested by: {ผู้ร้องขอ}
- Type: scope-change / requirement-change / design-change
- Reason: เหตุผล
- Impact: ผลกระทบต่อเอกสาร / code / timeline
- Decision: approved / rejected / deferred
- Updated docs: list ของไฟล์ที่อัพเดตหลัง approve
```

---

## Change Requests

### CR-001: เปลี่ยน reference pattern ของ backend และ frontend
- Date: 2026-05-04
- Requested by: User
- Type: design-change
- Reason: User กำหนดให้ใช้ reference ที่เหมาะสมกว่า
  - Backend: `happywork-backend/src/modules/v1/admin/expenseTypes/*` → `happywork-backend/src/modules/v1/employee/request/*`
  - Frontend: `happywork-sale-cms/src/sections/admin-users/*` → `happywork-sale-cms/src/app/dashboard/survey/*` (App Router pattern แยก page list/create/[id]/edit)
- Impact: 
  - Frontend pattern เปลี่ยนจาก dialog-based CRUD → page-based CRUD (มี create/edit เป็นหน้าแยก)
  - Section structure ใช้ `*-list-view`, `*-create-view`, `*-detail-view`, `*-edit-view` + `components/` + `utils/` + `*.enum.ts`
  - `feature-management.frontend.md` ปรับ page structure ใหม่
- Decision: approved
- Updated docs: 
  - `feature-management.backend.md` (พ่วงเอกสารชุดเดิม), 
  - `feature-management.frontend.md`, 
  - `feature-management.checklist.md`,
  - `feature-management.prompt.md` (เพิ่ม follow-up clarifications)

### CR-002: ปรับเอกสารใน workspace `_docs/requirement/feature-management/` ให้ตรง template
- Date: 2026-05-04
- Requested by: User
- Type: design-change (documentation)
- Reason: เอกสารชุดที่ workspace ต้องตรง template (`requirement.md`, `prd.md`, `ssd.md`, `summary.md`, `task-list.md`, `testcase.md`, `cr.md`) เพื่อ align กับมาตรฐาน workspace
- Impact:
  - ลบไฟล์เดิม 6 ไฟล์ (feature-management.{summary,backend,frontend,migration,checklist,prompt}.md)
  - สร้างไฟล์ใหม่ 7 ไฟล์ตาม template พร้อม frontmatter YAML
  - เนื้อหาเหมือนเดิมแต่จัดโครงใหม่ตาม template
- Decision: approved
- Updated docs: 
  - `_docs/requirement/feature-management/{requirement,prd,ssd,summary,task-list,testcase,cr}.md` (ใหม่)
  - เอกสารใน `happywork-backend/_docs/requirement/feature-management/` ยังคงรูปแบบเดิม (ไม่กระทบ)

---

## Statistics

| Status | Count |
|--------|-------|
| Approved | 2 |
| Rejected | 0 |
| Deferred | 0 |
| Total | 2 |
