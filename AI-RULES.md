# AI Rules — Root Pointer

> **All AI agents MUST read [`.ai-memory/rules.md`](.ai-memory/rules.md) first before doing anything in this workspace.**

This file is a **bootstrap pointer**. The single source of truth for AI rules in this AI Space is:

→ **[.ai-memory/rules.md](.ai-memory/rules.md)** (master rules)

## Symlinks

The following files are symlinks to this file (verified via `ls -la`):

- `CLAUDE.md` → `AI-RULES.md`
- `AGENTS.md` → `AI-RULES.md`
- `GEMINI.md` → `AI-RULES.md`
- `CODEX.md` → `AI-RULES.md`

So no matter which AI tool reads which file name, all of them are funneled here, and from here to `.ai-memory/rules.md`.

## Bootstrap Sequence (ลำดับการอ่านเมื่อเริ่ม session ใหม่)

1. Read [`.ai-memory/rules.md`](.ai-memory/rules.md) — กฏหลักของ workspace
2. Read [`.ai-memory/current.md`](.ai-memory/current.md) — งานที่กำลังทำอยู่ปัจจุบัน
3. Read [`.ai-memory/summary.md`](.ai-memory/summary.md) — สรุปภาพรวม (ถ้ามี)
4. Read [`.ai-memory/task-list.md`](.ai-memory/task-list.md) — รายการ task ระดับโปรเจค
5. ถ้าจะทำงานในบทบาท agent ใดบทบาทหนึ่ง → อ่าน `.ai-memory/agent/{role}/rules.md` + `skills.md` + `current-task.md` ของ agent นั้น
6. ถ้าเป็น Codex → อ่าน `.ai-memory/agent/codex/rules.md` + `skills.md` + `current-task.md` แล้วค่อยเลือก specialist role ที่เหมาะสม

## Quick Reference

| Topic                     | Where                          |
| ------------------------- | ------------------------------ |
| Master rules              | `.ai-memory/rules.md`          |
| Current focus             | `.ai-memory/current.md`        |
| Project summary           | `.ai-memory/summary.md`        |
| Project-level task list   | `.ai-memory/task-list.md`      |
| Agent rules / skills      | `.ai-memory/agent/{role}/`     |
| Codex adaptive rules      | `.ai-memory/agent/codex/`      |
| Feature requirement docs  | `_docs/requirement/{feature}/` |
| Requirement template      | `_docs/requirement/_template/` |
| Claude Code subagent defs | `.claude/agents/{role}.md`     |
