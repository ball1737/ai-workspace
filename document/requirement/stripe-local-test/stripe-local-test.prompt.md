# Stripe Local Test — Raw Prompt

> Prompt ที่ได้รับจากผู้ใช้

ฉันต้องการจะทำงานที่ต่อกับ stripe
ใช้ context7 อ่าน doc stripe ให้หน่อย แล้วอธิบายขั้นตอนในการรันบน local เพื่อทดสอบ ให้หน่อย

## Clarifications (ระหว่าง plan mode)

- **context7 MCP**: ยังไม่ได้ติดตั้งใน workspace (มีแค่ Google Drive). ผู้ใช้แจ้งจะติดตั้งเอง — guide นี้จึงอ้างอิงเฉพาะ existing internal docs + code; หลัง user ติดตั้ง context7 แล้วสามารถใช้ดึง Stripe official doc มาเสริมได้ใน turn ถัดไป
- **Scope**: อธิบายขั้นตอน run + test ของที่มีอยู่บน local (ไม่ใช่ implement เพิ่ม / ไม่ใช่ stripe-sync-rewrite)
- **Output mode**: เขียน guide ลงไฟล์ตาม global rule §1.2 (3 ไฟล์ summary/checklist/prompt) + สรุปในแชต

## Plan File

`/Users/ball/.claude/plans/stripe-context7-glowing-peacock.md`
