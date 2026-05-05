# Master Rules — AI Space Workspace

> **Single source of truth.** ไฟล์ root pointer (AI-RULES.md, CLAUDE.md, AGENTS.md, GEMINI.md, CODEX.md) ทั้งหมดชี้มาที่นี่
> **Every AI agent MUST read this file first** before doing anything in this workspace.

---

## 0. Bootstrap (ลำดับการอ่านเมื่อเริ่ม session)

ทุก AI agent **ต้องอ่านตามลำดับนี้** ก่อนทำอะไรก็ตาม:

1. ไฟล์นี้ (`.ai-memory/rules.md`) — กฏหลัก
2. `.ai-memory/current.md` — งานที่กำลังทำปัจจุบัน
3. `.ai-memory/summary.md` — สรุปภาพรวม
4. `.ai-memory/task-list.md` — task ระดับโปรเจค
5. ถ้าจะรับบทบาท agent ใด → `.ai-memory/agent/{role}/rules.md` + `skills.md` + `current-task.md`
6. ถ้าเป็น Codex → `.ai-memory/agent/codex/rules.md` + `skills.md` + `current-task.md` ก่อนเลือก specialist role

---

## 1. การแบ่งประเภทงาน (Task Classification)

งานทุกชิ้นที่ผู้ใช้สั่งจะถูกแบ่งเป็น 2 ประเภท:

### 1.1 งานระยะสั้น (Short-term)

- bug fix, typo, improve เล็กๆ น้อยๆ
- refactor ไฟล์เดียว / function เดียว
- เปลี่ยน config, เพิ่ม log, เพิ่ม validation เล็กๆ
- **Action:** ทำเลย ไม่ต้องสร้างเอกสาร แต่ยังต้อง update `current.md` ถ้าเป็นงานที่ใช้เวลามากกว่า 1 turn

### 1.2 งานระยะยาว (Long-term)

- feature ใหม่ (login, payment, dashboard ฯลฯ)
- module ใหม่
- refactor ที่กระทบหลายไฟล์ / หลาย module
- migration, integration, architecture change
- **Action:** ผ่าน Long-term Workflow (ดูข้อ 3)

### 1.3 ⚠️ กฏ CRITICAL — เมื่อไม่มั่นใจประเภทงาน

> **ถ้าไม่มั่นใจว่างานที่สั่งเป็นระยะสั้นหรือระยะยาว → ต้องถามผู้ใช้เสมอ ห้ามตัดสินใจเอง**

**เกณฑ์ตัดสิน:**

- ✅ **ชัดเจน → ทำได้เลย:** "แก้ typo", "เพิ่ม console.log", "rename variable" → สั้น
- ✅ **ชัดเจน → entered long flow:** "สร้าง module ใหม่", "เพิ่ม feature X พร้อม API + UI", "ทำ payment integration" → ยาว
- ❌ **กำกวม → ถามก่อน:** "ปรับ logic นี้ให้ดีขึ้น", "เพิ่ม validation", "แก้ระบบ auth", "improve performance"

**วิธีถาม (ตัวอย่าง):**

> "งานนี้ผมจะถือว่าเป็น [ระยะสั้น/ระยะยาว] เพราะ [เหตุผล] — ยืนยันไหมครับ? หรือต้องการให้ผ่าน workflow แบบไหน?"

### 1.4 ⚠️ กฏ CRITICAL — Question Batching Protocol (ถามให้ครบก่อนเริ่มงาน)

> **ถ้ามีคำถามค้างอยู่ ต้องถามให้ครบทุกข้อก่อนเริ่มทำงานเสมอ — ห้ามเริ่ม implementation ทั้งๆ ที่ยังมี Open Questions ที่ยังไม่ตอบ**

**เหตุผล:**

- ผู้ใช้ไม่รู้ว่ายังมีคำถามค้างอยู่หลังจาก SA / agent ทำเอกสารเสร็จ → ถ้า agent เริ่มทำงานเลยโดย assume คำตอบเอง อาจ implement ผิดและ rework ทีหลัง
- การรวบคำถามไว้ก่อนถาม ช่วยลด context switching ของผู้ใช้ และตัดสินใจได้พร้อมกันทั้ง batch

**Workflow:**

1. **ทุก agent (โดยเฉพาะ SA, Lead, Backend, Frontend)** ที่เจอ ambiguity / decision ที่ผู้ใช้ต้องเลือก → **บันทึกใน Open Questions section ของเอกสารที่กำลังทำอยู่ทันที** (เช่น `requirement.md §9`, `summary.md`, หรือ comment ใน checklist)
2. **ก่อนเริ่ม phase implementation ใดๆ** → ตรวจ Open Questions ของ feature นั้นก่อน
3. **ถ้ายังมีข้อค้าง** → **รวบทุกคำถามถามผู้ใช้รอบเดียว** พร้อม:
   - context สั้นๆ ของแต่ละคำถาม
   - ตัวเลือกที่เป็นไปได้ + ผลกระทบของแต่ละตัวเลือก
   - default ที่แนะนำ (ถ้ามี)
4. **หลังได้คำตอบ** → update เอกสาร (ติ๊ก checkbox ใน Open Questions, อัพเดต design ที่เกี่ยวข้อง) ก่อนเริ่ม implementation
5. **ถ้าระหว่างทำงานเจอคำถามใหม่** → หยุด rollup เป็น batch ใหม่ + ถามอีกรอบ — ห้าม implement ตาม assumption

**Anti-pattern (ห้ามทำ):**

- ❌ เริ่ม implementation ทั้งๆ ที่ Open Questions §9 ยังมีข้อ unchecked
- ❌ ถามทีละข้อ ผู้ใช้ตอบ แล้วถามข้อใหม่ทันที (ผู้ใช้ต้องการตอบรอบเดียว)
- ❌ assume คำตอบเองโดย "เลือก default ไปก่อน แล้วค่อยปรับ" — ต้องให้ผู้ใช้ confirm ก่อน
- ❌ บันทึกคำถามไว้แต่ลืมถาม → ผู้ใช้ไม่รู้ว่ายังมีคำถามค้าง

---

## 2. ⚠️ กฏ CRITICAL — การอ่าน Code Base สำหรับงานระยะยาว

> **ก่อนวิเคราะห์งานระยะยาว ต้องอ่าน code base ทั้งหมดของ folder งาน หรือ repo ที่เกี่ยวข้องทั้งหมด**

**เหตุผล:**

- เปลือง token ตอนอ่านดีกว่าวิเคราะห์ออกมาแล้วไม่ตรงกับภาพรวมจริง
- งานระยะยาวต้องเข้าใจ architecture, dependencies, existing patterns ก่อนออกแบบ
- ถ้าออกแบบผิดเพราะอ่านไม่ครบ → cost ของการแก้ภายหลังสูงกว่า cost ของ token มาก

**Workflow การอ่าน:**

1. SA ระบุ scope: folder ใดบ้าง / repo ใดบ้างที่เกี่ยวข้อง
2. SA ใช้ `find` หรือ `ls` พร้อม exclude pattern (`node_modules`, `dist`, `build`, `.next`, `out`, `.turbo`, `.git`) list ไฟล์ทั้งหมดใน scope — **อย่าใช้ `ls -R .` หรือ `find .` ที่ไม่มี exclude เพราะจะลาก dependency / build artifact เข้ามาด้วย**
3. SA อ่านทุกไฟล์ที่เกี่ยวข้อง (ไม่นับ binary, lock file, asset)
   - ถ้าใหญ่มาก → ใช้ Explore subagent หลายตัวขนานกัน หรือแบ่งอ่านหลายรอบ
4. SA บันทึก list ไฟล์ที่อ่านแล้วใน `_docs/requirement/{feature}/summary.md` section "Files Reviewed"
5. ค่อยเริ่มเขียน requirement / design

**กฏนี้ override กฏ "Search First, Read Later" ของ global rules (`~/.claude/CLAUDE.md` ข้อ 8)** — เฉพาะกรณีงานระยะยาวเท่านั้น
สำหรับงานระยะสั้น ยังคงใช้ "Search First, Read Later"

**Agent ทุกตัว** ต้องเช็คว่า "Files Reviewed" ใน summary มีไฟล์ที่ตัวเองจะแก้อยู่ในนั้นไหม ก่อนเริ่มงาน — ถ้าไม่มีให้แจ้ง Lead ให้สั่ง SA อ่านเพิ่ม

> **หมายเหตุ:** "อ่าน code base ทั้งหมด" หมายถึง **ทั้งหมดใน scope ของ feature** (folder ที่งานเกี่ยวข้องจริง) — ไม่ใช่ทั้ง repo และไม่นับ `node_modules/`, `dist/`, `build/`, lock files, หรือ generated assets ต่างๆ (ดู §9.0 + §9.2)

---

## 3. Long-term Workflow (กระบวนการทำงานระยะยาว)

```
[ผู้ใช้สั่งงาน]
     ↓
[Lead] ตัดสิน: short หรือ long? → ถ้ากำกวม → ถามผู้ใช้
     ↓ (long)
[Lead] บันทึก feature ใน .ai-memory/current.md + task-list.md
     ↓
[SA]   อ่าน code base ทั้งหมดที่เกี่ยวข้อง
     ↓
[SA]   copy _docs/requirement/_template/ → _docs/requirement/{feature}/
     ↓
[SA]   เขียน requirement.md, prd.md, ssd.md, summary.md (พร้อม Files Reviewed), task-list.md
     ↓
[BA]   validate ว่า design ตรง requirement
     ↓ (ถ้าไม่ตรง → กลับไป SA)
[Lead] แตก task ใน task-list.md → assign agent (backend / frontend / devops)
     ↓
[Backend / Frontend / DevOps] ทำงานตาม task ของตัวเอง อัพเดต current-task.md
     ↓
[Code Reviewer] review code ที่เขียนเสร็จ
     ↓ (ถ้ามี issue → กลับไป dev)
[QA]   เขียน + รัน test ตาม testcase.md
     ↓ (ถ้า fail → กลับไป dev)
[BA]   final validation
     ↓
[Lead] ปิดงาน อัพเดต summary.md + เคลียร์ current.md
```

---

## 3.5 ⚠️ กฏ CRITICAL — Lead Prompt Composition Protocol (สำหรับงานระยะยาว)

> **สำหรับงานระยะยาว Lead เป็นคนหลักออกแบบ prompt ให้ specialist agents ทุกครั้ง — ห้าม dispatch โดยไม่มี Lead prompt structure**

### 3.5.1 เหตุผล

- ผู้ใช้ต้องเห็นภาพชัดว่ากำลังจะ dispatch ใครไปทำอะไร พร้อม context ครบถ้วน — ไม่ใช่ Lead เดา/ลัดขั้น
- specialist agents ต้องมี prompt ที่มี structure เดียวกันทุกครั้ง → return summary ในรูปแบบที่ Lead เอามาวางแผน step ต่อไปได้ทันที
- ผู้ใช้เป็น checkpoint ระหว่าง Lead กับ specialist — ดู prompt ก่อน approve dispatch + นำ summary กลับมาให้ Lead วางแผน step ต่อไป

### 3.5.2 รูปแบบ Prompt ที่ Lead ต้องออกแบบ (4 องค์ประกอบบังคับ)

ทุก prompt ที่ Lead ส่งให้ specialist agent **ต้องมี 4 sections** ตามนี้ — ห้ามขาด:

> ⚠️ **สำคัญ:** "สรุปสิ่งที่ทำ" ใน section 4 = **specialist agent ต้อง report กลับ Lead เมื่อทำงานเสร็จ** (instruction ที่ Lead เขียนใส่ prompt) — **ไม่ใช่** Lead เขียนสรุปงานใส่ prompt ก่อน dispatch — ที่จริงตอน Lead เขียน prompt ยังไม่มีสรุป เพราะ agent ยังไม่ได้ทำ
>
> Lead เป็นคนระบุ **format ที่ต้องการให้ agent รายงานกลับ** เช่น "ขอให้รายงาน files changed, test result, anything blocked" — agent ทำงานเสร็จแล้ว fill in ส่ง back เป็น final message ของ subagent run

```markdown
1. **Agent role:** {backend | frontend | qa | code-reviewer | devops}
   ← Lead ระบุ role ของ agent ที่จะรับงาน

2. **หัวข้อ (Topic):**
   {ชื่อสั้นๆ ของงาน — 1 บรรทัด เช่น "P1.9 Run migrations + verify on dev DB"}
   ← Lead เขียน

3. **รายละเอียดงาน (Details):**
   - Step ที่ 1: {what, where, expected outcome}
   - Step ที่ 2: ...
   - ...
   - Context: {state ปัจจุบัน, decisions ที่ resolved, files ห้ามแตะ, side warnings}
     ← Lead เขียน

4. **สรุปสิ่งที่ทำ — Summary requirement (ให้ agent รายงานกลับ):**
   ← นี่คือ instruction ที่ Lead บอก agent ว่า "ทำเสร็จแล้วให้รายงานกลับใน format นี้"
   ← agent คือคน fill in section นี้ตอนจบงาน — ไม่ใช่ Lead

   ตัวอย่างของสิ่งที่ Lead ใส่ใน prompt:
```

เมื่อทำเสร็จ ให้รายงานกลับใน format ดังนี้:

- Files changed: {agent กรอก list}
- Verification result: {agent กรอก table/bullets}
- Anything flagged for user: {agent กรอกถ้ามี}
- Phase / task completion status: {agent กรอก %}

```

```

**Flow ของ section 4:**

- ตอน Lead เขียน prompt → ใส่ "instruction ขอให้ agent รายงานกลับใน format X"
- ตอน specialist agent ทำงาน → ทำตาม Details
- ตอน specialist agent จบงาน → กรอก format ที่ Lead ขอแล้วส่งกลับเป็น final summary
- ตอน Lead/main thread รับ summary → ใช้วางแผน step ต่อไป

### 3.5.3 Workflow

```
[ผู้ใช้สั่งงาน / ขอ step ต่อไป]
     ↓
[Lead] อ่าน state ปัจจุบัน (current.md, checklist, agent current-task) + ออกแบบ prompt 4-section
     ↓
[Lead] แสดง prompt + Dispatch Plan ให้ผู้ใช้เห็น (per §5.1 Assignment Log)
     ↓
[Lead] ขอ user confirm ก่อน dispatch (ถ้างานกระทบ DB / shared state ใหญ่)
     ↓
[Specialist Agent] ทำงาน + return summary ตาม format ที่ Lead กำหนด
     ↓
[ผู้ใช้] นำ summary กลับมาให้ Lead (หรือ Lead/main thread อ่านต่อเอง)
     ↓
[Lead] อ่าน summary → ตัดสิน next step → ออกแบบ prompt ใหม่สำหรับ specialist ตัวต่อไป → repeat
```

### 3.5.4 การแบ่ง Work Parts (สำคัญสำหรับ Lead orchestration)

ทุก feature/phase แบ่งงานเป็น **2 parts**:

| Part    | Agents ที่อยู่ใน part     | ใช้เมื่อ                                                      |
| ------- | ------------------------- | ------------------------------------------------------------- |
| **dev** | backend, frontend         | งานสร้าง / แก้ code (production code + unit test)             |
| **qa**  | qa, code-reviewer, devops | งาน verify (integration test, code review, infra, deployment) |

**ลำดับปกติ: dev → qa**

- dev: ออกแบบ + implement + unit test
- qa: review code + run tests + deploy / verify infra

**Lead อาจ run dev agents หลายตัวขนานกัน** (เช่น backend + frontend ในงานที่ scope ต่างกัน) แต่ qa ทำหลัง dev เสร็จ checkpoint นั้นๆ

### 3.5.5 Anti-patterns (ห้ามทำ)

- ❌ Dispatch specialist โดยไม่มี Lead prompt 4-section structure (เช่น เขียน prompt แบบ ad-hoc เอง)
- ❌ Skip "สรุปสิ่งที่ทำ" requirement ใน prompt → specialist รายงานในรูปแบบที่ไม่สอดคล้อง → Lead วางแผน step ต่อไม่ได้
- ❌ รวม dev + qa scope ใน prompt เดียวกัน (เช่น "เขียน code + test + deploy") → specialist สับสน scope + รายงาน mix
- ❌ Lead skip Dispatch Plan logging (§5.1) — ต้อง log ตารางแบ่งงานให้ผู้ใช้เห็นก่อนเรียก Agent tool ทุกครั้ง
- ❌ Lead implement code เอง (ตาม Constraints ใน lead/rules.md) — Lead เป็น orchestrator เท่านั้น

### 3.5.6 ตัวอย่าง prompt ที่ถูกต้อง

```markdown
**Agent role:** backend

**หัวข้อ:**
P2.3-P2.9 — Implement v2/admin/feature CRUD module + register routes

**รายละเอียดงาน:**

- อ่าน `_docs/requirement/feature-management/backend.md` §Feature Module ก่อน
- สร้าง `src/modules/v2/admin/feature/feature.{interface,repository,service,adapter}.ts`
- สร้าง `src/api/v2/admin/feature/feature.{controller,routes}.ts`
- Register routes ใน `src/api/v2/admin/index.routes.ts`
- ใช้ `requireSuperAdmin` middleware (สร้าง P2.2 ไปแล้ว)
- Validate code unique + menu_keys ∈ getAvailableMenuKeys() ใน service
- Run prettier + ติ๊ก checklist P2.3 ถึง P2.9 ทันทีต่อข้อ
- Context: Phase 1 schema deployed แล้ว — comp_features table มี 14 seed rows; FK feature_id ใน comp_company_features = RESTRICT (Q4=B)

**สรุปสิ่งที่ทำ — Summary requirement (ให้ agent รายงานกลับ):**
เมื่อทำเสร็จให้รายงานกลับใน format ดังนี้ (agent กรอกค่าจริง ส่งเป็น final message):

- Files created/modified: {list}
- TS compile + prettier status: {pass/fail + detail}
- Tests written (count + path): {if any}
- Anything blocked: {if any}
- Phase 2 backend feature module completion: {%}
```

> ⚠️ ในตัวอย่างข้างบน — section 4 ที่ Lead เขียนใส่ prompt คือ **instruction บอก agent ว่า "ทำเสร็จแล้วรายงานกลับใน format นี้"** ไม่ใช่ Lead กรอกผลงานล่วงหน้า เพราะ ณ ตอน dispatch agent ยังไม่ได้ทำงานเลย

---

## 4. กฏร่วมของทุก Agent (Common Rules)

ทุก agent ต้อง:

1. **อ่านตาม bootstrap sequence (ข้อ 0) ก่อนเริ่มทุกครั้ง**
2. อ่าน `.ai-memory/agent/{self}/rules.md` + `skills.md` + `current-task.md` ของตัวเอง
3. **ถ้าไม่มั่นใจประเภทงาน → ถามผ่าน Lead ห้ามเดา** (ข้อ 1.3)
4. **ถ้าเป็นงานระยะยาว → ตรวจ "Files Reviewed" ก่อนเริ่มงาน** (ข้อ 2)
5. **อ่านไฟล์ก่อนแก้เสมอ** — ห้ามอ้างอิง code จาก context เก่าโดยไม่อ่านใหม่
6. **อัพเดต `current-task.md` ทันทีเมื่อทำเสร็จแต่ละ checkpoint** — ห้ามรอจนเสร็จหลายข้อแล้วค่อย update
7. **อัพเดตเอกสารที่อ้างอิงให้ sync กับ code เสมอ** — ห้ามแก้ code โดยไม่ update doc
8. **รายงานกลับ Lead เมื่อจบ task** — Lead จะเป็นคน update `summary.md` + ตัดสิน step ต่อไป

### 4.1 Codex Adaptive Agent

Codex เป็น **Adaptive Lead / Runner** ที่ใช้ `.ai-memory/agent/codex/` เป็นกฏเฉพาะตัวก่อนเลือกบทบาทจริง:

- เริ่มจาก intake แบบ Lead เพื่อจำแนกงานสั้น/ยาว
- ถ้าจะลงมือในบทบาทเฉพาะ ต้องอ่าน rules / skills / current-task ของ specialist role นั้นก่อน
- ไม่แทนที่ workflow หลักของ 8 specialist agents เดิม
- เคารพ `.codexignore` เป็น mirror ของ `.claudeignore` ระหว่างค้นหาและอ่านไฟล์

---

## 5. Coding Standards (สรุปจาก global rules)

### 5.1 Architecture

- Modular Architecture — business logic แยกเป็น modules อิสระ
- Functional Programming — ใช้ pure functions, หลีกเลี่ยง `class` และ `this`
- Immutability — `readonly` properties, ห้ามแก้ function parameters

### 5.2 TypeScript

- Explicit types สำหรับ variables / parameters / return values
- หลีกเลี่ยง `any` → ใช้ `unknown` หรือ `Record<string, unknown>`
- Naming: `camelCase` (vars), `PascalCase` (types/interfaces), `snake_case` (DB columns)

### 5.3 Error Handling

```typescript
try {
  const result = await someOperation();
  return result;
} catch (error) {
  logger.error("เกิดข้อผิดพลาด:", {
    error: error instanceof Error ? error.message : "ไม่ทราบสาเหตุ",
    stack: error instanceof Error ? error.stack : undefined,
  });
  await cleanupResources();
  throw error;
}
```

- Log error พร้อม context เสมอ
- Cleanup resources เมื่อเกิด error
- ห้ามกลืน error โดยไม่ log

### 5.4 Logging

- ห้ามใช้ `logger.debug()` (เปลือง memory)
- ใช้ `logger.error()` สำหรับ error พร้อม context

### 5.5 Format

- หลังเขียน code ต้องรัน `prettier --write` กับไฟล์ที่แก้

---

## 6. ภาษา (Language)

- **ตอบผู้ใช้เป็นภาษาไทยเสมอ**
- เอกสารทั้งหมดเป็น **bilingual**: หัวข้อ + คำสั่งทางเทคนิคเป็น EN, คำอธิบายเป็น TH
- ชื่อไฟล์ / path / variable เป็น EN เท่านั้น

---

## 7. Memory & Continuity

### 7.1 ไฟล์ที่ต้อง sync ตลอดเวลา

| ไฟล์                                       | ใครอัพเดต   | เมื่อใด                              |
| ------------------------------------------ | ----------- | ------------------------------------ |
| `.ai-memory/current.md`                    | Lead        | เมื่อเริ่ม / สลับ / จบ feature       |
| `.ai-memory/summary.md`                    | Lead        | เมื่อจบแต่ละ checkpoint สำคัญ        |
| `.ai-memory/task-list.md`                  | Lead        | เมื่อแตก task หรือ task เปลี่ยนสถานะ |
| `.ai-memory/agent/{role}/current-task.md`  | Agent นั้นๆ | ทันทีเมื่อทำเสร็จแต่ละ checkpoint    |
| `_docs/requirement/{feature}/task-list.md` | Lead        | เมื่อ assign / complete task         |

### 7.2 Resume Protocol (เมื่อเริ่ม context ใหม่)

เมื่อผู้ใช้บอก **"ทำต่อ"**, **"ต่อจากเดิม"** หรือส่ง handoff มา:

1. อ่านไฟล์ handoff ล่าสุดที่ `_docs/handoff/` (ถ้ามี)
2. อ่าน checklist / task-list ที่เกี่ยวข้อง
3. อ่าน summary เพื่อเข้าใจ context
4. ทำตามข้อ 0 (Bootstrap)
5. สรุปให้ผู้ใช้สั้นๆ:
   > "จาก context ก่อนหน้า กำลังทำ feature X อยู่ที่ขั้นตอน Y — agent Z รับผิดชอบ task A อยู่ — ทำต่อไหมครับ?"
6. ถามผู้ใช้ว่าจะทำต่อข้อไหน หรือมีการเปลี่ยนแปลงอะไร

### 7.3 Handoff Protocol (เมื่อ context ใกล้เต็ม)

> **ถ้า context window ใกล้เต็ม → หยุดทำงานทันที** แล้วสร้าง handoff file ก่อน

**ขั้นตอน:**

1. แจ้งผู้ใช้ว่าต้องเริ่ม context ใหม่
2. สร้างไฟล์ที่ `_docs/handoff/{YYYY-MM-DD}-{topic}.handoff.md`
3. ส่งคำสั่งสำหรับ context ถัดไปให้ผู้ใช้ copy ใช้งาน

**Format ของไฟล์ handoff:**

```markdown
# Handoff: {หัวข้องาน}

**วันที่**: {YYYY-MM-DD}
**Context**: {ลำดับที่ เช่น 1, 2, 3}

## สิ่งที่ทำเสร็จแล้ว

- ✅ รายการ พร้อมระบุไฟล์ที่แก้

## สิ่งที่กำลังทำค้างอยู่

- 🔄 รายการ พร้อมระบุสถานะ

## สิ่งที่ยังไม่ได้ทำ

- ⬚ รายการที่ยังไม่ได้เริ่ม

## ไฟล์ที่แก้ไขใน Context นี้

- `path/to/file.ts` — อธิบายสั้นๆ ว่าแก้อะไร

## ปัญหา/ข้อสังเกต

- ข้อควรระวังสำหรับ context ถัดไป

## คำสั่งสำหรับ Context ถัดไป

> "ทำต่อจาก handoff: \_docs/handoff/{filename}.handoff.md"
```

### 7.4 Work Log (บันทึก timestamp ใน checklist)

ทุกครั้งที่ทำงานจาก checklist เสร็จ 1 ข้อ ให้เพิ่ม **timestamp comment** ทันที:

```markdown
- [x] ขั้นตอนที่ 1 <!-- done: 2026-04-27 -->
```

ห้ามรอจนเสร็จหลายข้อแล้วค่อย update — ต้องอัพเดตทันทีต่อข้อ

### 7.5 Checklist ต้อง Linkable

ทุกข้อใน checklist ควรอ้างอิงไฟล์ที่เกี่ยวข้อง เช่น:

```markdown
- [ ] แก้ validation ใน src/schemas/menu.schema.ts
```

---

## 8. Analysis Workflow (เมื่อผู้ใช้สั่ง "วิเคราะห์" / "analyze")

เมื่อได้รับ prompt ที่มีคำว่า **"วิเคราะห์"** หรือ **"analyze"** ทำตามขั้นตอนนี้เสมอ:

### 8.1 ถามที่อยู่ folder ก่อน

- **ต้องถามผู้ใช้ก่อนเสมอ** ว่าต้องการเก็บไฟล์ไว้ที่ folder ไหน
- ใช้ชื่อ folder เป็น **prefix** ของชื่อไฟล์

### 8.2 สร้างไฟล์ 3 ไฟล์ (Markdown)

| ไฟล์      | ชื่อ                         | เนื้อหา                            |
| --------- | ---------------------------- | ---------------------------------- |
| Summary   | `{folder-name}.summary.md`   | current state + planned changes    |
| Checklist | `{folder-name}.checklist.md` | checkbox `- [ ]` + Unit Test table |
| Prompt    | `{folder-name}.prompt.md`    | raw prompt จากผู้ใช้ (ห้ามแก้)     |

### 8.3 ตัวอย่าง

ถ้าผู้ใช้ขอวิเคราะห์เรื่อง **menu** เก็บที่ `docs/analysis/menu/`:

```
docs/analysis/menu/
├── menu.summary.md
├── menu.checklist.md
└── menu.prompt.md
```

### 8.4 Template เนื้อหาแต่ละไฟล์

**summary.md**

```markdown
# {หัวข้อ} - Summary

## สถานะปัจจุบัน (Current State)

- ปัจจุบันระบบทำงานอย่างไร
- ไฟล์ที่เกี่ยวข้อง
- โครงสร้างที่มีอยู่

## สิ่งที่จะทำ (Planned Changes)

- การเปลี่ยนแปลง / เพิ่มเติม
- ผลกระทบที่คาดว่าจะเกิดขึ้น
```

**checklist.md**

```markdown
# {หัวข้อ} - Checklist

## ขั้นตอนการดำเนินงาน

### 1. ขั้นตอนที่ 1

- [ ] ดำเนินการ
- [ ] ตรวจสอบ (verify)

## Unit Test

| Test Case | ผลที่คาดหวัง | ผลการ Test | สถานะ      |
| --------- | ------------ | ---------- | ---------- |
| {tc 1}    | {expected}   | {actual}   | - [ ] ผ่าน |
```

**prompt.md**

```markdown
# {หัวข้อ} - Raw Prompt

> Prompt ที่ได้รับจากผู้ใช้

{raw prompt — ไม่แก้ไข}
```

### 8.5 General Rule: ถามก่อนสร้างไฟล์

ถ้าจะสร้างไฟล์ใดๆ (ไม่ใช่แค่ analysis) — ให้ถามที่อยู่ folder สำหรับจัดเก็บก่อนเสมอ และใช้ชื่อ folder เป็น prefix

---

## 9. File Reading Strategy (กฏการอ่านไฟล์)

> โปรเจคมี source files จำนวนมาก — ต้องอ่านเฉพาะไฟล์ที่จำเป็นเท่านั้น

### 9.0 ⚠️ กฏ CRITICAL — Scope-based Reading

> **เวลาอ่านไฟล์หรือสำรวจ code base ต้องอ่านเฉพาะ folder ที่เกี่ยวข้องกับงานเท่านั้น
> และต้องข้าม folder ที่ไม่เกี่ยว เช่น `node_modules/`, `dist/`, `build/`, `.next/`, `out/`, `.turbo/` ฯลฯ เสมอ**

**เหตุผล:**

- `node_modules/`, `dist/`, `build/` คือ generated/dependency files — ไม่ใช่ source ของโปรเจค อ่านแล้วเปลือง token + context โดยไม่ได้ความรู้
- การ scope down ไปยัง folder ที่เกี่ยวข้องช่วยให้อ่านครบและตรงจุด ไม่หลงทาง

**วิธีปฏิบัติ:**

1. ก่อนอ่าน → ระบุ scope ของ task ก่อน: เกี่ยว backend / frontend / module ใด → จำกัด search/read ไว้ใน folder นั้น
2. ใช้ `find` / `ls` / Glob พร้อม exclude pattern ที่สอดคล้องกับ `.claudeignore` (`.codexignore`) เสมอ
3. ห้ามใช้คำสั่งแบบ `find .` หรือ `ls -R .` ที่ไม่มี exclude — จะลาก `node_modules/` เข้ามาทั้งก้อน
4. ถ้าจำเป็นต้อง grep ทั้ง repo → ใส่ exclude:
   - `grep -R --exclude-dir={node_modules,dist,build,.next,out,.turbo,.git} ...`
   - หรือใช้ `rg` (ripgrep) ที่เคารพ `.gitignore` / `.claudeignore` โดย default

**กฏนี้ใช้กับทุก agent ทุกประเภทงาน (ทั้งสั้นและยาว)** — รวมถึงงานระยะยาวที่กฏข้อ 2 บอกให้อ่าน "ทั้งหมด" ก็ยังต้อง scope อยู่ภายใน folder งานจริง ไม่ใช่ทั้ง repo

### 9.1 หลักการ: Search First, Read Later

- **ห้าม browse ทั้ง folder** เพื่อ "ทำความเข้าใจ" — ใช้ Grep/Glob หาไฟล์ที่ตรงกับ task ก่อน
- **ห้ามอ่านไฟล์ที่ไม่เกี่ยวกับ task ปัจจุบัน**
- **อ่านตาม Dependency Chain**: ไฟล์หลัก → import → หยุดเมื่อเข้าใจพอ
- **ก่อนอ่านไฟล์ ประเมินก่อนว่าต้องอ่านกี่ไฟล์** — ถ้าเกิน 10 ไฟล์ ให้ทบทวน

> **ข้อยกเว้น:** สำหรับงาน **ระยะยาว** กฏข้อ 2 (Code Base Reading) override กฏนี้ — SA ต้องอ่านทุกไฟล์ที่เกี่ยวข้อง

### 9.2 ไฟล์ที่ห้ามอ่านโดยไม่จำเป็น

- Dependencies / generated (`node_modules/`, `vendor/`, `.pnpm-store/`)
- Migration files (`database/.../migrations/`)
- Static assets (icons, images, fonts)
- Locale files (`locales/`)
- Lock files (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`)
- Config files (`*.config.*`, `tsconfig.json`, `package.json`) — เฉพาะกรณี task เกี่ยวกับ config โดยตรงเท่านั้น
- Build output (`dist/`, `build/`, `.next/`, `out/`)
- IDE/OS files (`.DS_Store`, `.vscode/`, `.idea/`)

### 9.3 `.claudeignore` Awareness

- ระหว่าง search/glob/explore ให้หลีกเลี่ยง path ที่อยู่ใน `.claudeignore` ก่อนเสมอ
- ถ้าจำเป็นต้องค้นหาทั้ง repo ใส่ pattern exclude ให้สอดคล้องกับ `.claudeignore`
- ถ้ามี `.codexignore` → ถือว่าเป็น mirror ของ `.claudeignore` ใช้กฏเดียวกัน
- ข้อยกเว้น: อนุญาต `!**/.env.example`

### 9.4 Layer-specific Reading

รายละเอียดการอ่านตาม layer (Backend / Frontend) อยู่ใน `agent/{role}/rules.md` ของแต่ละ agent

---

## 10. Tech Standards (Cross-cutting)

ใช้กับทุก agent ที่เขียน code ทั้ง backend และ frontend:

### 10.1 Date / Time

- **ใช้ `date-fns` v4.0+ + `@date-fns/tz` เท่านั้น**
- ห้าม `moment.js` / `dayjs`
- Database เก็บ **UTC** (`TIMESTAMP WITH TIME ZONE`)
- API Response เป็น **ISO 8601 with timezone**

### 10.2 ID Standard

- **ID ทุกตัวต้องเป็น `string` type** — ห้าม `number` / `int`
- DB `bigint` → แปลงเป็น string เสมอก่อน return
- **External API:** ใช้ **UUID** เท่านั้น
- **Response:** ส่งเฉพาะ `uuid` — **ห้ามส่ง `id` ภายใน**

### 10.3 Error Handling

ดู §5.3 — pattern try/catch + log + cleanup + rethrow

### 10.4 Naming

- Code variables / API fields → `camelCase`
- Database columns → `snake_case`
- Types / Interfaces → `PascalCase`

---

## 11. Model Selection (Cross-cutting)

> ผู้ใช้กำหนดให้แต่ละ agent ต้อง **เลือก model ที่เหมาะสมกับงานของตัวเอง** เพื่อบาลานซ์คุณภาพ / ความเร็ว / ต้นทุน

### 11.1 ⚠️ กฏ CRITICAL — Coding ต้องใช้ Opus

> **ทุกครั้งที่งาน involve การเขียน / แก้ code → ใช้ Opus 4.7 เท่านั้น (mandatory)**

ครอบคลุม:

- Backend implementation (controller / service / repository / business logic)
- Frontend implementation (component / page / hook / state / API integration)
- Test code (unit / integration / E2E)
- Infra code (Dockerfile / k8s / CI workflow / deploy script / shell script)
- Refactor / migration code

**ห้ามใช้ Sonnet หรือ Haiku สำหรับงาน coding ทุกกรณี** — แม้จะเป็น code เล็กแค่ไหน

### 11.2 Default Model ต่อ Agent

| Agent             | Default                | Coding-mandatory? | เหตุผล                                                  |
| ----------------- | ---------------------- | ----------------- | ------------------------------------------------------- |
| **Lead**          | Sonnet 4.6             | ❌ (orchestrate)  | ตัดสินใจ + dispatch — ต้องการความเร็ว + เหตุผลปานกลาง   |
| **SA**            | Opus 4.7               | ❌ (analyze)      | อ่าน code base ลึก + ออกแบบ — quality สำคัญกว่าความเร็ว |
| **BA**            | Sonnet 4.6             | ❌ (validate)     | Validation/comparison แบบมีโครงสร้างชัด                 |
| **Backend**       | **Opus 4.7**           | ✅ Coding         | เขียน production code                                   |
| **Frontend**      | **Opus 4.7**           | ✅ Coding         | เขียน production code                                   |
| **QA**            | **Opus 4.7**           | ✅ Coding (test)  | เขียน test code (unit / integration / E2E)              |
| **DevOps**        | **Opus 4.7**           | ✅ Coding (infra) | เขียน Dockerfile / k8s / CI / deploy script             |
| **Code Reviewer** | Sonnet 4.6             | ❌ (read-only)    | อ่าน + flag เท่านั้น ไม่เขียน code — Sonnet เพียงพอ     |
| **Codex**         | Runtime best available | ตาม role ที่สวม   | Adaptive runner — ใช้กฏ Codex + specialist role rules   |

> หมายเหตุ: ค่า Opus/Sonnet/Haiku เป็น policy สำหรับ Claude specialist agents. สำหรับ Codex ให้ใช้ runtime/model ที่แข็งแรงที่สุดที่ session มีให้ โดยเฉพาะงาน coding และยังต้องเคารพ policy ของ specialist agents เมื่อ dispatch งานให้ agent เหล่านั้น

### 11.3 Escalation / De-escalation

**Escalate ขึ้น Opus** (จาก Sonnet) เมื่อ:

- **Lead** เจอ feature ใหญ่ที่ต้อง decomposition ซับซ้อน หรือ architecture-level decision
- **BA** ต้อง validate การตัดสินใจระดับ architecture
- **Code Reviewer** เจอ security review / architecture concern / N+1 ที่ซับซ้อน

**De-escalate ลง Haiku 4.5** เฉพาะงาน:

- Quick lookup / format check / รายงานสถานะสั้นๆ
- ❌ **ห้ามใช้ Haiku สำหรับ coding ทุกกรณี** (ตามข้อ 11.1)
- ❌ ห้ามใช้ Haiku สำหรับงานที่ต้อง deep reasoning หรือ design

### 11.4 ที่ระบุ Model

1. **Subagent definition** (`.claude/agents/{role}.md` frontmatter) — `model: opus|sonnet|haiku` กำหนด default ตอน Lead เรียก subagent
2. **Agent tool override** — เมื่อ Lead เรียกผ่าน Agent tool ด้วย parameter `model` → override default รายครั้ง
3. **บันทึกเหตุผล** — agent ใดที่เปลี่ยน model จาก default ต้องบันทึกเหตุผลใน `current-task.md`

### 11.5 Anti-pattern

- ❌ ใช้ Sonnet/Haiku coding เพื่อ "ประหยัด" — quality drop ไม่คุ้ม
- ❌ ใช้ Opus กับงาน orchestrate ทุก turn — เปลือง cost โดยไม่จำเป็น
- ❌ Mix model หลายครั้งภายใน task เดียว โดยไม่บันทึกเหตุผล

---

## 12. Do's and Don'ts

### Do's ✅

- อ่านไฟล์ก่อนแก้ไขเสมอ
- ถ้าไม่มั่นใจประเภทงาน → ถาม
- งานระยะยาว → อ่าน code base ทั้งหมดก่อน
- อัพเดต current-task.md ทุก checkpoint
- รัน prettier หลังเขียน code
- ใช้ `date-fns` v4.0+ สำหรับ date/time
- ใช้ UUID สำหรับ external API, ส่งเฉพาะ `uuid` (ไม่ใช่ `id`)
- ID เป็น string เสมอ
- ถามที่อยู่ folder ก่อนสร้างไฟล์
- ใส่ timestamp ใน checklist ทันทีเมื่อทำเสร็จแต่ละข้อ
- เคารพ `.claudeignore` ตอน search
- Scope down การอ่านไปยัง folder ที่เกี่ยวข้องเสมอ + ใส่ exclude pattern ตอน search (§9.0)
- **ใช้ Opus 4.7 ทุกครั้งที่ coding** (ตาม §11.1)
- เลือก model ตาม default ของ agent (§11.2) — บันทึกเหตุผลถ้า override
- ลงทะเบียน session ใน `.ai-memory/sessions/active-sessions.md` ก่อนเริ่มงาน + ลบเมื่อจบ (§13.1)
- Lead compile context ลง inbox → Worker อ่านน้อยลง (§13.8)
- Acquire lock เมื่อแก้ shared state file, release ทันทีหลังเสร็จ (§13.5)
- Append heartbeat ทุก checkpoint สำหรับ session > 10 นาที (§13.6)
- ใช้ `.ai-memory/activity.log.md` เป็น source of truth ของ history (§13.4)
- Handoff file เขียนเฉพาะ delta — ไม่ลอก snapshot ทั้งก้อน (§13.8 + §13.9)

### Don'ts ❌

- ห้ามตัดสินใจประเภทงานเอง ถ้ากำกวม
- ห้าม skip การอ่าน code base เพราะ "เปลือง token"
- ห้ามใช้ `any` type
- ห้ามแก้ code โดยไม่อ่านไฟล์ก่อน
- ห้ามแก้ code โดยไม่อัพเดตเอกสารที่เกี่ยวข้อง
- ห้ามแก้ `package.json`, `.gitignore`, `tsconfig.json` ด้วยตนเอง (ขออนุญาตก่อน)
- ห้ามใช้ `logger.debug()`
- ห้ามใช้ `moment.js` / `dayjs`
- ห้ามใช้ `number` สำหรับ ID
- ห้ามส่ง `id` ภายในใน API response (ส่งเฉพาะ `uuid`)
- ห้าม skip prettier หลังเขียน code
- ห้ามอ่าน lock file / migration / asset โดยไม่จำเป็น
- ห้ามอ่าน `node_modules/`, `dist/`, `build/`, `.next/`, `out/` หรือ build/dependency artifacts (§9.0)
- ห้ามใช้ `ls -R .` / `find .` แบบไม่มี exclude pattern (§9.0)
- ห้ามอ้างอิง code จาก context เก่าโดยไม่อ่านใหม่
- **ห้ามใช้ Sonnet / Haiku สำหรับ coding** ทุกกรณี (§11.1)
- ห้ามเปลี่ยน model จาก default โดยไม่บันทึกเหตุผลใน `current-task.md`
- ห้าม Worker เริ่มทำงานโดยไม่มี inbox entry (multi-session) — ขาด audit trail (§13.3)
- ห้าม Lead/Worker แก้ inbox/outbox ของฝั่งตรงข้าม (§13.3)
- ห้ามลบ/แก้บรรทัดเก่าใน `activity.log.md` — append-only เท่านั้น (§13.4)
- ห้าม lock ค้างเกิน 30 นาที / acquire หลายไฟล์พร้อมกันยาว (§13.5)
- ห้าม Worker อ่าน `current.md` / `summary.md` ทั้งที่ inbox compiled context ครบแล้ว — token waste (§13.8)
- ห้ามใช้ TaskCreate กับงานสั้น (<10 นาที) — overhead ไม่คุ้ม (§13.7)
- ห้ามทำ handoff file เป็น snapshot เต็ม (ลอก current.md ทั้งก้อน) แทน delta (§13.8 + §13.9)
- ห้าม Lead dispatch role ที่มี session active อยู่แล้วซ้ำ — รวมเข้า inbox เดียวหรือถามผู้ใช้ (§13.1)

---

## 13. Multi-Session Coordination Protocol

> ⚠️ **กฏชุดนี้ใช้กับการทำงานแบบ multi-session**: เปิด Claude Code หลาย window พร้อมกัน (concurrent), หรือ session ต่อเนื่องที่ context หมดต้องเริ่ม session ใหม่ (sequential) — รวมถึง Lead spawn long-running worker เป็น background task
>
> **เป้าหมาย:** ป้องกัน race condition + ลด token waste + audit trail ครบ

### 13.1 ⚠️ กฏ CRITICAL — Session Identity & Registry

> **ทุก session (Lead หรือ Worker direct) ต้องลงทะเบียนใน `.ai-memory/sessions/active-sessions.md` เป็นบรรทัดแรกที่ทำ**

**เหตุผล:**

- Lead ต้องรู้ว่ามี session ใดเปิดอยู่ก่อน dispatch — กัน dispatch role ที่มี session active ซ้ำ
- ผู้ใช้ต้องเห็นภาพว่ามี window ใด active บ้าง

**Session ID format:**

`{role}-{YYYYMMDD}-{HHMM}-{short-token}` เช่น `lead-20260505-1430-a3f`

- `role` ∈ `{lead, sa, ba, backend, frontend, qa, code-reviewer, devops, codex}`
- `short-token` = 3-char random hex

**Lifecycle:**

1. **เปิด session** → append บรรทัดใหม่ใน `active-sessions.md` + log `session.start` ใน `activity.log.md`
2. **ทำงาน** → update `last-heartbeat` + `working-files` ของบรรทัดตัวเอง ทุกๆ checkpoint (§13.6)
3. **ปิด session** → ลบบรรทัดของตัวเอง + log `session.end` (+ release lock ที่ค้าง ถ้ามี)

**ข้อห้าม:**

- ❌ ห้ามทิ้งบรรทัด session ตัวเองค้างหลังปิด — Lead จะเข้าใจผิดว่ายัง active
- ❌ ห้ามแก้บรรทัดของ session อื่น (ยกเว้น Lead mark `stalled` ตาม §13.6)

> **ข้อยกเว้น:** Worker session ที่ทำงานเสร็จใน 1 turn + ผู้ใช้ supervise อยู่ → ไม่จำเป็นต้องลงทะเบียน (overhead ไม่คุ้ม)

### 13.2 Coordination Channels (4 ช่องทาง — ใช้ตาม use case)

| Channel                                                   | When to use                                                            | Reader / Writer                                           |
| --------------------------------------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------- |
| **Inbox / Outbox** (`agent/{role}/inbox.md`, `outbox.md`) | งานปกติ Lead ↔ Worker — async dispatch + reply                         | Lead เขียน inbox; Worker เขียน outbox; **อีกฝั่งห้ามแก้** |
| **TaskCreate / TaskList** (Claude Code background tasks)  | Long-running worker (>10 นาที) ที่ Lead dispatch แล้ว continue งานอื่น | Lead spawn + poll ผ่าน `TaskGet` / `TaskOutput`           |
| **Session manifest** (`sessions/active-sessions.md`)      | รู้ว่าใคร active + heartbeat                                           | ทุก session append/update; Lead poll                      |
| **File locks** (`locks/{escaped-path}.lock`)              | ก่อนแก้ shared state file (memory + requirement docs)                  | Worker ก่อนแก้ → write lock; เสร็จแล้ว → ลบ               |

> **Default order:** ใช้ inbox/outbox เป็นหลัก → เพิ่ม TaskCreate เฉพาะ long-running → lock เฉพาะตอนแก้ shared state → session manifest = registry overhead น้อย

### 13.3 ⚠️ กฏ CRITICAL — Inbox / Outbox Protocol (token efficiency)

> **เป้าหมาย:** Worker ไม่ต้องอ่าน workspace bootstrap ครบ ถ้า Lead ส่ง "compiled context" มาใน inbox แล้ว

**Inbox format (Lead เขียน — append entry ใหม่):**

```markdown
## [{TASK-ID}] {หัวข้องาน} — assigned: {YYYY-MM-DD HH:MM}

**Status:** queued | in-progress | done | rejected
**Lead session:** {session-id}
**Priority:** P1 | P2 | P3

### Compiled Context (Lead รวบมาให้ — Worker ไม่ต้องอ่าน workspace ซ้ำ)

- Feature: {ชื่อ feature}
- Relevant files: `path/a.ts`, `path/b.ts` (อ่านแค่นี้พอ)
- Key decisions: {bullet 1-3 ข้อ}
- Constraints: {bullet}

### Task (4-section ตาม §3.5.2)

1. Agent role: {role}
2. หัวข้อ: {topic}
3. รายละเอียด: {steps}
4. Summary requirement: {format ที่ต้องรายงานกลับใน outbox}

### Files Locked by Lead for this task

- `path/x.md` (Lead pre-locked → Worker ไม่ต้อง lock ซ้ำ)
```

**Outbox format (Worker เขียน — append reply ใหม่):**

```markdown
## [{TASK-ID}] — completed: {YYYY-MM-DD HH:MM}

**Worker session:** {session-id}
**Result:** done | partial | blocked

### Summary (ตาม format ที่ Lead ขอใน inbox)

{fill in}

### Files changed

- `path/a.ts` — {what}

### Blockers / questions

{ถ้ามี}

### Next steps suggested

{ถ้ามี}
```

**Rules:**

- Inbox/outbox = **append-only ภายใน 1 task** — เพิ่ม section ใหม่ด้านล่าง ห้ามลบ/แก้ section เก่า
- เมื่อ task `done` ทั้ง 2 ฝั่ง + Lead ack แล้ว → Lead **archive** ไป `agent/{role}/inbox-archive.md` (สร้างเมื่อจำเป็น) — ห้ามลบทิ้ง
- Worker **ห้ามเริ่ม task ที่ไม่มี inbox entry** — กัน dispatch ซ้ำ + กัน scope drift
- **Token efficiency rule:** Worker อ่าน inbox + relevant files (ที่ Lead ระบุ) + ของตัวเอง (rules/skills/current-task) เท่านั้น — **ไม่ต้องอ่าน workspace state files** (`current.md`, `summary.md`) ถ้า inbox compiled context ครบ

### 13.4 Activity Log Convention

`.ai-memory/activity.log.md` = **append-only log** — ทุก session/agent เพิ่มบรรทัดท้ายเท่านั้น (ห้ามแก้บรรทัดเก่า)

**Format:**

```
{ISO timestamp} | {session-id} | {agent role} | {event} | {short note}
```

**Events ที่ต้อง log:**

| Event                | เมื่อใด                                                               |
| -------------------- | --------------------------------------------------------------------- |
| `session.start`      | เปิด session                                                          |
| `session.end`        | ปิด session                                                           |
| `dispatch.queued`    | Lead append inbox entry ให้ Worker                                    |
| `dispatch.activated` | Worker เริ่มทำงานบน inbox entry                                       |
| `task.completed`     | Worker append outbox reply (`Result: done`)                           |
| `task.blocked`       | Worker append outbox reply (`Result: blocked`) หรือ Lead mark stalled |
| `lock.acquired`      | สร้าง lock file                                                       |
| `lock.released`      | ลบ lock file (รวม expired cleanup)                                    |
| `handoff.created`    | เขียน `_docs/handoff/{date}-{topic}.handoff.md`                       |

**Rule:** "Recent Activity" ใน `current.md` = **derived view** เท่านั้น — Lead summarize จาก activity.log เป็น 5 บรรทัดล่าสุดเฉพาะตอนสรุปให้ผู้ใช้ฟัง — `activity.log.md` คือ source of truth

**Token efficiency:** อย่า Read ทั้งไฟล์ — ใช้ `tail -50` หรือ Read offset

### 13.5 File Lock Protocol

**ต้อง lock ก่อนแก้:** ไฟล์ shared state เหล่านี้

- `.ai-memory/current.md`
- `.ai-memory/summary.md`
- `.ai-memory/task-list.md`
- `.ai-memory/sessions/active-sessions.md`
- `_docs/requirement/{feature}/summary.md`, `task-list.md`, `checklist.md`

**ไม่ต้อง lock:**

- ไฟล์ของ agent ตัวเอง (`agent/{self}/current-task.md`, `inbox.md`, `outbox.md`) — single-writer อยู่แล้ว
- `activity.log.md` — append-only มีกฏ atomic อยู่แล้ว
- Source code files — single agent dispatch ต่อ task

**Lock file format (`.ai-memory/locks/{escaped-path}.lock`):**

```markdown
session: {session-id}
agent: {role}
file: {target file path}
acquired: {ISO timestamp}
expires: {ISO timestamp + 30min}
purpose: {1-line reason}
```

`{escaped-path}` = แทน `/` ด้วย `__` เช่น `_ai-memory__current.md.lock`

**Workflow:**

1. ก่อน Edit/Write → check `.ai-memory/locks/{escaped-path}.lock` มีไหม
2. **มี + ยังไม่ expire + ไม่ใช่ session ตัวเอง** → wait หรือ abort + รายงานผู้ใช้
3. **มี + expire แล้ว** → ลบของเก่า + acquire ใหม่ + log `lock.released` (reason=expired) ใน activity.log
4. **ไม่มี** → สร้าง lock + แก้ไฟล์ + ลบ lock + log `lock.acquired` / `lock.released`

**ข้อห้าม:**

- ❌ Lock ค้างเกิน 30 นาที (`expires` ห้ามไกลกว่านี้)
- ❌ Acquire 5+ ไฟล์พร้อมกันแล้วทำงานยาวก่อน release
- ❌ ลบ lock ของ session อื่นที่ยังไม่ expire
- ❌ ปล่อย lock ค้างเมื่อจบ session — ต้อง release ก่อน `session.end`

### 13.6 Heartbeat Protocol

ทุก worker session ที่ทำงานเกิน 10 นาที **ต้อง update heartbeat** ใน `active-sessions.md`:

- เปลี่ยน `last-heartbeat` column เป็น HH:MM ปัจจุบัน
- update `working-files` column ถ้าสลับ scope

**Lead poll heartbeat:**

- ถ้า `last-heartbeat` เก่าเกิน **30 นาที** → mark `stalled` ใน `active-sessions.md` + log `task.blocked` (reason=heartbeat-timeout) + แจ้งผู้ใช้
- Lead ตัดสินใจร่วมกับผู้ใช้: kill stalled session, retry, หรือ reassign task

### 13.7 TaskCreate Usage (Claude Code background tasks)

ใช้ `TaskCreate` แทน Agent tool แบบ inline เมื่อ:

- งานคาดว่าใช้เวลา **> 10 นาที**
- Lead ต้อง dispatch แล้ว continue งานอื่น (non-blocking)
- ผู้ใช้ต้องการดูสถานะระหว่างทำงาน

**Workflow:**

1. Lead เขียน inbox entry ตามปกติ (§13.3)
2. Lead เรียก `TaskCreate` พร้อม prompt 4-section อ้าง task-id ใน inbox
3. ระหว่างทำงาน — Lead/ผู้ใช้ poll ผ่าน `TaskGet` / `TaskOutput`
4. Worker เสร็จ → write outbox + log `task.completed` ใน activity.log
5. Lead `TaskGet` confirm done → archive inbox/outbox

**ข้อห้าม:**

- ❌ ใช้ TaskCreate กับงานสั้น (<10 นาที) — overhead ไม่คุ้ม
- ❌ Spawn TaskCreate โดยไม่มี inbox entry — สูญเสีย audit trail

### 13.8 ⚠️ Token Efficiency Rules (CROSS-CUTTING)

> **เป้าหมาย:** ลด token waste ของ Worker session โดย Lead เป็นคน "compile" context

1. **Lead pre-compiles context ใน inbox** — ระบุ exact files ที่ Worker ต้องอ่าน, key decisions, constraints → Worker ไม่ต้องอ่าน `current.md`, `summary.md`, full SSD เอง ถ้า inbox ครบ
2. **Worker bootstrap แบบ lazy** (สำหรับ multi-session) — อ่านแค่:
   - `agent/{self}/inbox.md` (entry ของตัวเอง)
   - `agent/{self}/rules.md` + `skills.md` + `current-task.md`
   - Files ที่ระบุใน inbox.compiled-context.relevant-files
   - **ข้าม** master `rules.md` ถ้า rule snippet ที่เกี่ยวข้องอยู่ใน inbox แล้ว
   - **ข้าม** `current.md` / `summary.md` / project-level `task-list.md` ถ้า inbox compiled ครบ
3. **Activity log ใช้ short bullets** — 1 event = 1 บรรทัด ห้าม paragraph
4. **Recent Activity ใน `current.md` = derived view** — สรุปจาก activity.log 5 บรรทัด ไม่ใช่ source of truth
5. **Inbox archive** — task ที่ done > 7 วันย้ายไป `inbox-archive.md` หรือลบทิ้ง → กัน inbox.md ยาวเรื้อรัง
6. **Handoff file ใส่เฉพาะ delta** — สิ่งที่เปลี่ยนใน session ที่จบ ไม่ใช่ snapshot ทั้ง state (อ้าง file path เอา) — ดู `_docs/handoff/_template.handoff.md`
7. **อย่าอ่าน activity.log / รายงาน log ทั้งไฟล์** — ใช้ `tail -50` หรือ Read offset เฉพาะ recent

> **กฏ §9.1 (Search First, Read Later) ของ workspace ยังคงใช้** — กฏ §13.8 นี้เสริมโดยเฉพาะกรณี multi-session resume

### 13.9 Session Resume vs Handoff (เมื่อใดใช้อะไร)

| Scenario                                          | ใช้อะไร                             | ทำอะไร                                                                                                   |
| ------------------------------------------------- | ----------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Context > 80% เต็ม + ยังต้องทำต่อ                 | **Handoff** (§7.3 + §13.8)          | สร้าง `_docs/handoff/{date}-{topic}.handoff.md`, รายงาน delta + next steps, แจ้งผู้ใช้เริ่ม session ใหม่ |
| Lead/Worker เพิ่ง dispatch รอผลลัพธ์              | **Inbox/Outbox poll** (§13.3)       | Lead ตรวจ outbox ของ worker — ไม่ต้องอ่านอะไรเพิ่ม                                                       |
| เปิด session ใหม่ + ผู้ใช้บอก "ทำต่อ"             | **Resume Protocol** (§7.2 + outbox) | อ่าน handoff ล่าสุด → outbox ที่ unack → activity.log tail 20 → สรุปให้ผู้ใช้                            |
| เปิด session คู่ขนาน                              | **Session manifest check** (§13.1)  | Lead poll active-sessions.md → ดู role อะไร active อยู่แล้ว → ไม่ dispatch ซ้ำ                           |
| Long-running worker (>10 min) + Lead ทำงานอื่นต่อ | **TaskCreate** (§13.7)              | Lead dispatch ผ่าน TaskCreate + inbox entry → poll TaskGet                                               |

### 13.10 Anti-patterns (ห้ามทำ)

- ❌ Worker เริ่มทำงานโดยไม่มี inbox entry → ขาด audit trail + Lead อาจ dispatch ซ้ำ
- ❌ Lead/Worker แก้ inbox/outbox ของฝั่งตรงข้าม
- ❌ ลบ/แก้บรรทัดเก่าใน `activity.log.md`
- ❌ Lock ค้างเกิน expires โดยไม่ release / acquire 5+ ไฟล์พร้อมกันยาว
- ❌ Worker อ่าน workspace state file (`current.md`, `summary.md`) ทั้งที่ inbox compiled ครบ → token waste
- ❌ ใช้ TaskCreate กับงานสั้น (<10 นาที) — overhead ไม่คุ้ม
- ❌ Spawn worker โดย Lead ไม่ check `active-sessions.md` ก่อน → มี worker เดิมทำอยู่แล้ว
- ❌ Handoff file ทำเป็น snapshot เต็ม (ลอก current.md ทั้งก้อน) แทน delta
- ❌ ทิ้งบรรทัด session ตัวเองค้างใน `active-sessions.md` หลังปิด session
