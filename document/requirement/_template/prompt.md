---
feature: { feature-name }
type: feature-raw-prompt
created: { YYYY-MM-DD }
updated: { YYYY-MM-DD }
owner: SA (capture)
status: draft
---

# {Feature Name} — Raw Prompt

> Prompt ที่ได้รับจากผู้ใช้ (preserve ตามต้นฉบับ — ห้ามแก้ไขเนื้อหา)

---

## Original Prompt

> วาง raw prompt ที่ผู้ใช้ส่งมาตรงนี้ (ไม่ตัด ไม่เพิ่ม)

```
{ raw prompt ที่ได้รับมา }
```

---

## Follow-up Clarifications

> ถ้ามี grill session / ถาม-ตอบเพิ่ม ระหว่างวิเคราะห์ ให้บันทึกคำถาม + คำตอบที่นี่

| #   | คำถาม | คำตอบ | ที่มา (session date) |
| --- | ----- | ----- | -------------------- |
| 1   |       |       |                      |

---

## Reference Patterns (ที่ user ระบุให้ใช้)

> ถ้า user ระบุ pattern หรือ module ที่ให้ใช้เป็นแม่แบบ ให้บันทึกที่นี่

- Backend reference: `path/ไปยัง/module ที่ใช้เป็นแม่แบบ`
- Frontend reference: `path/ไปยัง/page หรือ section ที่ใช้เป็นแม่แบบ`
- Other: ...

---

## Constraints / Out-of-Scope (ที่ user ระบุ)

> ข้อจำกัด หรือสิ่งที่ user บอกว่าไม่ต้องทำ

- ...
