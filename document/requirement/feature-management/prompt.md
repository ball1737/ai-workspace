---
feature: feature-management
type: feature-raw-prompt
created: 2026-05-04
updated: 2026-05-04
owner: SA (capture)
status: draft
---

# Feature Management — Raw Prompt

> Prompt ที่ได้รับจากผู้ใช้ (preserve ตามต้นฉบับ)

---

backend - happywork-backend
frontend - happywork-sale-cms
ฉันต้องการเริ่มวางแผนงานระยะยาว 1 งาน เป็น feature จัดการ feature ของ package และลูกค้า
รายละเอียดตามนี้

- ตอนนี้ระบบมีการจัดการ การแสดงเมนูด้วย comp_permission ซึ่ง base ของเมนูมาจากตัวแปร PermissionDefault ใน happywork-backend/src/constant/compPermission.ts ซึ่งเป็นตัวแปร constant อยู่ทำให้ ไม่ยืดหยุ่น หรือปรับเปลี่ยนไปตามแต่ละ comp_company ได้
- PermissionDefault ถือเป็น เมนูที่เปิดให้ใช้งานในระบบ ให้อิงจากตัวแปรนี้เป้นหลัก
- ฉันต้องการให้ปรับปรุงส่วนนี้โดยให้แต่ละ comp_company มี เมนู ที่จะจัดการใน comp_permission ต่างกันในแต่ละ comp_company โดยขึ้นอยู่กับว่าถูกเปิดให้ใช้ feature ไหนบ้าง
- module ฉันต้องการมีหลักๆ 3 อย่างคือ คือ
  - เมนูจัดการ feature โดย เป็นหน้า CRUD ว่าในระบบมี feature อะไรบ้าง โดยประกอบด้วยชื่อ feature , รายละเอียด , เมนูที่จะเห็นเมื่อมีสิทธิ์ใช้ feature นี้ (อิงจาก ตัวแปร PermissionDefault เป็นหลัก)
  - เมนูจัดการ package สำหรับจัดการ package โดยเป็นหน้า CRUD เหมือนกัน (ยังไม่มี table เก็บ สร้างไฟล์ migrate ให้ด้วย โดยอิง field ต่างๆที่ happywork-backend/\_docs/requirement/subscription/subscription-package-summary.md ) และจัดการว่าแต่ละ package จะมี feature อะไรบ้าง (พร้อมทั้งแสดงให้เห้นว่า แต่ละ feature มีเมนูอะไรบ้าง พอรวมทุก feature ที่จะได้ใช้ จะเห้นเมนูทั้งหมดเป้นยังไง)
  - เมนูจัดการ feature ของแต่ละ comp_company โดยแต่ละ comp_company จะมี feature ที่ใช้ได้ตาม package ของ comp_company นั้นๆ เป็น feature เริ่มต้นที่เปิดให้ โดยในหน้าจะ list แสดง feature ทั้งหมด และบอกว่า feature ไหนถูกเปิดใช้งานบ้าง ซึ่งสามารถเปิดปิด เพิ่มให้ได้ (migrate table ใหม่มาเก็บการ map data ส่วนนี้) ซึ่ง feature ที่เปิดปิดนี้เองจะไปแสดงเมนูตามที่ถูก config ไว้ในหน้าจัดการ permission ของแต่ละ comp_company
- จากสิ่งที่เพิ่มมาทำให้ต้องกลับไปแก้ api ทั้งหมดที่เกี่ยวข้องกับ comp_permission / comp_permission_mapping ให้ดึงจากตัว mapping ไปแทน

ช่วยวิเคราะห์และออกแบบงานทั้งหมดทั้งส่วน backend , frontend แบบละเอียดออกมาเป้นไฟล์ให้หน่อย

- ส่วนของ backend ให้วิเคราะห์และ plan งานออกมาให้ละเอียดแบบ จะทำกี่ route แต่ละ route รายละเอีดยังไง มีกี่ controller กี่ service กี่ repository แล้วแต่ตัวอย่างต้องทำอะไรบ้าง list รายละเอียดพร้อม task ทั้งหมดออกมา (ย่อยให้ลึกที่สุด)
- ส่วนของ frontend ให้วิเคราะห์และ plan งานออกมาให้ละเอียดแบบ ว่าจะมีต้องมีกี่หน้า แล้วแต่ละหน้าทำกี่ component (component ที่สามารถ reuse ได้ให้ทำแยกไว้)
- แบ่งงานตามแต่ละหน้าที่ agent ด้วย
- ทำ task list ออกมาให้ละเอียดที่สุด (เพื่อที่ให้เมื่อเปลี่ยน session แล้วก็ยังสามารถทำงานต่อเนื่องได้)

สร้างเอกสารไว้ที่ \_docs/requirement/feature-management

---

## Follow-up clarifications (จาก /grill-me session)

| #   | คำถาม                          | คำตอบ                                            |
| --- | ------------------------------ | ------------------------------------------------ |
| 1   | นิยาม feature ↔ menu           | D — feature ผูก leaf path แบบ mixed (main + sub) |
| 2   | Feature ↔ comp_permission      | C — filter ที่ runtime, JSONB เดิมไม่แตะ         |
| 3   | Package → Feature → Company    | B — Delta/Override                               |
| 4   | Strategy รวม                   | A — Greenfield + Deprecate                       |
| 5   | Package table                  | B — สร้างใหม่ + migrate FK                       |
| 6.1 | Addon                          | A1 — แยก table                                   |
| 6.3 | comp_companies.package_id      | B1 — migrate FK เก็บชื่อ column เดิม             |
| 6.4 | Action gating                  | Yes (visibility only)                            |
| 7.1 | Migration scope ของ v1 callers | C — Keep const_packages alive                    |
| 7.2 | API version                    | v2                                               |
| 7.2 | CMS menu                       | Option I (เมนูใหม่แยก + tab ใน client detail)    |
| 7.3 | Cleanup ของ v1                 | แยก phase สุดท้าย                                |

## Reference patterns (ที่ user ระบุให้ใช้)

- Backend CRUD reference: `happywork-backend/src/modules/v1/employee/request/*` + `src/api/v1/employee/request/*`
- Frontend page reference: `happywork-sale-cms/src/app/dashboard/survey/*` + `src/sections/survey/*` (pattern Next.js App Router แยก page list/create/[id]/edit)
