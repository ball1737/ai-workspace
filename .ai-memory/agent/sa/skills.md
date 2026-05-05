# SA Agent — Skills

---

## Core Skills

### 1. Comprehensive Code Base Reading

**Inputs:** scope (folder / repo) + feature requirement
**Process:**
- ใช้ `find` / `ls -R` list ไฟล์ทั้งหมด
- กรองไฟล์ที่ไม่เกี่ยว: lock file, binary, asset, build output
- อ่านตาม dependency: entry point → import chain
- ถ้า > 30 ไฟล์ → spawn 3 Explore subagents ขนานกัน แบ่ง folder กัน
- Output: list ไฟล์ + summary โครงสร้าง / pattern ที่พบ

**Tools:** Bash (`find`, `ls -R`), Read (parallel), Agent (Explore subagent)

### 2. Requirement Analysis

**Inputs:** ผู้ใช้สั่ง + business context
**Process:**
- แยก functional vs non-functional req
- หา constraint ที่ implicit (เช่น performance, security)
- ระบุ stakeholder
- เขียน acceptance criteria แบบ Given-When-Then

### 3. System Design

**Inputs:** requirement + current code base
**Process:**
- ระบุ module ที่กระทบ
- ออกแบบ data model + API contract
- ระบุ migration strategy ถ้าแก้ DB
- พิจารณา trade-off (perf vs simplicity ฯลฯ)
- เขียน sequence diagram (text/mermaid) สำหรับ flow ซับซ้อน

### 4. Document Authoring

**Inputs:** requirement + design decisions
**Process:**
- ใช้ template ใน `_docs/requirement/_template/`
- กรอกทุก section อย่าทิ้ง placeholder
- ใช้ภาษา bilingual: หัวข้อ EN, อธิบาย TH
- Cross-reference ระหว่างไฟล์ (`requirement.md` link → `ssd.md`)

### 5. Open Question Identification

**Inputs:** requirement + code reading
**Process:**
- หาช่องว่างใน requirement (ambiguity)
- ระบุ assumption ที่ต้องการ confirmation
- จัดลำดับความสำคัญ (block / nice-to-know)

---

## Tool Preferences

| Task | Tool |
|------|------|
| List files | Bash (`find`, `ls -R`) |
| Read files | Read (parallel for multiple files) |
| Search patterns | Grep |
| Large code base | Agent (Explore subagent) ขนานกัน |
| Write doc | Write (template), Edit (revise) |

---

## Anti-patterns

- ❌ เริ่มเขียน SSD ก่อนอ่าน code ครบ
- ❌ ใส่ "Files Reviewed" ครึ่งๆ กลางๆ
- ❌ ใช้ `cat` หรือ `head` แทน Read tool
- ❌ Assume API/DB structure โดยไม่ verify
- ❌ Skip Open Questions เพื่อให้ดูเสร็จเร็ว
