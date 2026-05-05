# AI Space — Multi-Agent Workspace

> ระบบจัดระเบียบการทำงานกับ AI agent หลายตัว แบ่งงานเป็น **ระยะสั้น** (ทำเลย) และ **ระยะยาว** (วิเคราะห์ → เอกสาร → แตก task → มอบหมาย)

---

## Concept

งานที่สั่งให้ AI ใน workspace นี้ จะถูกแบ่งเป็น 2 ประเภทเสมอ:

| ประเภท                    | ตัวอย่าง                                         | วิธีทำ                                                                                                                                    |
| ------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **ระยะสั้น (Short-term)** | bug fix, typo, improve เล็กๆ, refactor ส่วนเดียว | ทำเลย                                                                                                                                     |
| **ระยะยาว (Long-term)**   | feature ใหม่, module ใหม่, งานสเกลใหญ่           | วางแผน → SA วิเคราะห์ + อ่าน code base ทั้งหมด → สร้างเอกสาร → Lead แตก task → มอบหมาย agent → BA validate → QA test → Code Reviewer ตรวจ |

**ถ้า AI ไม่มั่นใจว่าเป็นประเภทไหน → ต้องถามผู้ใช้เสมอ ห้ามตัดสินใจเอง**

---

## Folder Structure

```
.
├── AI-RULES.md                    # bootstrap pointer → .ai-memory/rules.md
├── CLAUDE.md / AGENTS.md / GEMINI.md / CODEX.md  # symlinks → AI-RULES.md
├── .claudeignore / .codexignore   # ignore rules สำหรับ AI tools
├── README.md                      # this file
│
├── .ai-memory/                    # AI memory (กฏ + state)
│   ├── rules.md                   # master rules ของ workspace
│   ├── current.md                 # งานที่กำลังทำปัจจุบัน
│   ├── summary.md                 # สรุปภาพรวม
│   ├── task-list.md               # task ระดับโปรเจค
│   └── agent/                     # rules + skills + current-task ของแต่ละ agent
│       ├── lead/
│       ├── sa/
│       ├── ba/
│       ├── backend/
│       ├── frontend/
│       ├── qa/
│       ├── devops/
│       ├── code-reviewer/
│       └── codex/                 # adaptive Codex rules
│
├── _docs/                         # เอกสารโปรเจค (artifact ของงาน)
│   └── requirement/
│       ├── _template/             # template เปล่า (SA copy ไปใช้)
│       └── {feature}/             # เอกสาร feature (สร้างต่อ feature)
│
└── .claude/
    └── agents/                    # Claude Code subagent definitions
        ├── lead.md
        ├── sa.md
        ├── ba.md
        ├── backend.md
        ├── frontend.md
        ├── qa.md
        ├── devops.md
        └── code-reviewer.md
```

---

## Agents Overview

| Agent                     | บทบาท                                                                  | Output หลัก                                      |
| ------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------ |
| **Lead**                  | orchestrator ตัดสินประเภทงาน, แตก task, มอบหมาย, ติดตาม, อัพเดต memory | `current.md`, `task-list.md`, dispatch           |
| **SA** (System Analyst)   | อ่าน code base ทั้งหมด, วิเคราะห์, ออกแบบ, เขียน PRD/SSD               | `_docs/requirement/{feature}/*.md`               |
| **BA** (Business Analyst) | validate ว่า design + dev ตรง requirement                              | gap report                                       |
| **Backend**               | เขียน backend ตาม SSD                                                  | API, DB, business logic                          |
| **Frontend**              | เขียน frontend ตาม PRD                                                 | UI components, pages                             |
| **QA**                    | เขียน + รัน test (unit ก่อน, e2e ภายหลัง)                              | `testcase.md`, test results                      |
| **DevOps**                | ตรวจ infra/server-related code                                         | review report                                    |
| **Code Reviewer**         | ตรวจคุณภาพ code (bug risk, performance, security)                      | review report                                    |
| **Codex**                 | adaptive runner เริ่มแบบ Lead แล้วสวม specialist role ตามงาน           | implementation / review / plan ตาม role ที่เลือก |

ดูรายละเอียดของแต่ละ agent ที่ [`.ai-memory/agent/{role}/`](.ai-memory/agent/)

---

## Getting Started

### สำหรับ Human ผู้สั่งงาน

แค่สั่งงานปกติ — ระบบจะวิเคราะห์เองว่าเป็นงานสั้นหรือยาว ถ้ากำกวมจะถามกลับ

### สำหรับ AI Agent (เริ่ม session ใหม่)

อ่านตามลำดับ:

1. [`.ai-memory/rules.md`](.ai-memory/rules.md)
2. [`.ai-memory/current.md`](.ai-memory/current.md)
3. ถ้าจะทำงานในบทบาทใด → `.ai-memory/agent/{role}/{rules,skills,current-task}.md`
4. ถ้าเป็น Codex → อ่าน `.ai-memory/agent/codex/{rules,skills,current-task}.md` ก่อน แล้วค่อยอ่าน role เฉพาะที่เลือกสวม

---

## License & Notes

- เอกสารเป็น **bilingual** (TH + EN headings, TH explanations)
- AI ต้อง **ตอบเป็นภาษาไทยเสมอ**
- กฏ global ใน `~/.claude/CLAUDE.md` ยังใช้ได้ แต่ workspace rules ชนะเมื่อขัดกัน (เช่น "Search First, Read Later" ถูก override โดยกฏ "อ่าน code base ทั้งหมด" สำหรับงานระยะยาว)
