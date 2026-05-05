---
agent: backend
updated: 2026-05-04
---

# Backend Agent — Current Task

> งานที่ Backend กำลังถืออยู่ — Lead เป็นคน assign

---

## Active Task

Implement Phase 1 — Schema Foundation for feature-management

**Task ID:** FEAT-101 ถึง FEAT-117 (P1.1–P1.18)
**Feature:** feature-management
**Description:** สร้าง 8 migrations + 7 Objection.js models ตาม `migration.md` + `ssd.md §3`
**Assigned by:** Lead (main thread)
**Started:** 2026-05-04
**Doc reference:** `document/requirement/feature-management/{migration.md,ssd.md,checklist.md,backend.md}`

**Acceptance criteria:**

- [x] 8 migration files created (M1–M8)
- [x] 7 Objection.js models created
- [x] All models define relationMappings + Type interfaces
- [x] Q1=C (Cartesian seed in M3) enforced
- [x] Q4=B (FK RESTRICT for feature_id in M6) enforced
- [x] UUIDs in M2/M4 seed preserve `packageMasterData` / `addonsMasterData` exactly
- [x] All files prettier-formatted
- [x] TypeScript compile clean (no errors in new files)
- [ ] User to run `npm run db:migrate:latest` on dev (P1.9 — outside agent scope)

## Pre-implementation Checklist

- [x] อ่าน rules.md / agent/backend/rules.md / skills.md / current-task.md <!-- done: 2026-05-04 -->
- [x] อ่าน migration.md (source of truth สำหรับ migration code) <!-- done: 2026-05-04 -->
- [x] อ่าน ssd.md §3 (schema definition + Q1-Q5 decisions) <!-- done: 2026-05-04 -->
- [x] อ่าน backend.md (overall structure) <!-- done: 2026-05-04 -->
- [x] อ่าน checklist.md (P1.1–P1.18) <!-- done: 2026-05-04 -->
- [x] ตรวจ Files Reviewed ใน summary.md ครอบคลุมไฟล์ที่จะแก้ (✅ pass) <!-- done: 2026-05-04 -->
- [x] อ่าน reference patterns: tableTemplate, alterTableOnUpdate, packageMasterData/addonsMasterData ตัวจริง <!-- done: 2026-05-04 -->
- [x] อ่าน existing models pattern (compAnnualHolidays, compCompanies, aprvFlowsFormMapping) <!-- done: 2026-05-04 -->

## Implementation Sub-checklist

- [x] M1 — `20260504-1000-create-comp_features.ts` (+ GIN index + 14 seed features)
- [x] M2 — `20260504-1001-create-comp_packages.ts` (+ GIN index + 7 seed packages, UUID preserved)
- [x] M3 — `20260504-1002-create-comp_package_features.ts` (+ Cartesian seed Q1=C)
- [x] M4 — `20260504-1003-create-comp_addons.ts` (+ GIN index + 9 seed addons, HappyBeacon quantifiable)
- [x] M5 — `20260504-1004-create-comp_addon_features.ts` (link table)
- [x] M6 — `20260504-1005-create-comp_company_features.ts` (Q4=B: feature_id RESTRICT)
- [x] M7 — `20260504-1006-create-comp_company_addons.ts` (+ migrate from comp_companies.add_ons)
- [x] M8 — `20260504-1007-alter-comp_companies-package-id-fk.ts` (HIGH RISK — backfill + FK swap)
- [x] Models 1-7 with full relationMappings + JSON schemas + Type interfaces

## Files Created

| Path                                                                                                                | Purpose                                                                |
| ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1000-create-comp_features.ts`                   | M1: master features table + 14 seed features                           |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1001-create-comp_packages.ts`                   | M2: master packages table + 7 seed packages (UUIDs preserved)          |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1002-create-comp_package_features.ts`           | M3: link table + Cartesian seed (Q1=C)                                 |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1003-create-comp_addons.ts`                     | M4: master addons table + 9 seed addons (HappyBeacon quantifiable)     |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1004-create-comp_addon_features.ts`             | M5: link table                                                         |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1005-create-comp_company_features.ts`           | M6: per-company override (Q4=B: feature_id FK RESTRICT)                |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1006-create-comp_company_addons.ts`             | M7: per-company addons + migrate from `comp_companies.add_ons` JSONB   |
| `happywork-backend/src/database/postgresql/migrations/data/20260504-1007-alter-comp_companies-package-id-fk.ts`     | M8: backfill + FK swap (HIGH RISK)                                     |
| `happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts`                                       | Model + relationMappings (packages, addons, companyOverrides)          |
| `happywork-backend/src/database/postgresql/models/data/compPackages.model.ts`                                       | Model + relationMappings (features, packageFeatures)                   |
| `happywork-backend/src/database/postgresql/models/data/compPackageFeatures.model.ts`                                | Link model + relationMappings (package, feature)                       |
| `happywork-backend/src/database/postgresql/models/data/compAddons.model.ts`                                         | Model + relationMappings (features, addonFeatures, companyAddons)      |
| `happywork-backend/src/database/postgresql/models/data/compAddonFeatures.model.ts`                                  | Link model + relationMappings (addon, feature)                         |
| `happywork-backend/src/database/postgresql/models/data/compCompanyFeatures.model.ts`                                | Override model + relationMappings (company, feature)                   |
| `happywork-backend/src/database/postgresql/models/data/compCompanyAddons.model.ts`                                  | Per-company addon model + relationMappings (company, addon)            |

## Deviations from SSD

| Description                                                                                                                                              | Reason                                                                                                                                          | Reported to Lead?            |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `comp_packages` seed = **7 records** (not 8 as stated in SSD §3.2 / checklist P1.2)                                                                       | `packageMasterData` ใน `package.interface.ts` มีจริง 7 records: Seed + LITE×2 + CORE×2 + PRO×2 = 7. SSD/checklist เขียน 8 น่าจะ count ผิด        | Yes — flagged in this report |
| `comp_addons` seed = **9 records** (HappyBeacon + E-Slip×2 + Evaluation×2 + HappyPM×2 + E-Signature×2)                                                    | จำนวน addons ที่อยู่ใน `addonsMasterData` จริง                                                                                                  | Yes — informational          |
| `tableTemplate` adds `company_id BIGINT` to all master tables (comp_features, comp_packages, comp_addons)                                                | ตามคำแนะนำของ Lead — keep tableTemplate consistent + column nullable; conform to existing pattern (`const_packages` ก็มี structure คล้ายกัน) | No deviation — by design     |

## Blockers

ไม่มี blocker ตอนนี้ — Phase 1 ทำเสร็จครบ 18/18 (เว้น P1.9 ที่ user ต้อง execute เอง)

## Pre-execution Notes for User (Before Running P1.9)

ก่อนรัน `npm run db:migrate:latest` บน dev:

1. **Pre-validation query** (ตามใน `migration.md` Pre-Migration Checklist) — ตรวจว่า `comp_companies.selected_package_uuid` ทุกค่า match กับ 7 UUID ใน packageMasterData. ถ้ามี orphan UUID ต้องแก้ก่อน mig M8 จะ rollback
2. **Backup DB** ก่อน apply M8 (HIGH RISK)
3. หลังรัน latest:
   - `SELECT COUNT(*) FROM comp_features;` ต้อง = 14
   - `SELECT COUNT(*) FROM comp_packages;` ต้อง = 7
   - `SELECT COUNT(*) FROM comp_addons;` ต้อง = 9
   - `SELECT COUNT(*) FROM comp_package_features;` ต้อง = 14×7 = 98 (Cartesian)
   - `SELECT COUNT(*) FROM comp_companies cc LEFT JOIN comp_packages cp ON cc.package_id=cp.id WHERE cp.id IS NULL;` ต้อง = 0

## Recent updates (latest 5)

| Date       | Update                                                                                       |
| ---------- | -------------------------------------------------------------------------------------------- |
| 2026-05-04 | **HOTFIX** — M6 + M7 duplicate `company_id` (PG 42701) แก้แล้ว: ใช้ `table.foreign()` แทน redeclare + raw `ALTER COLUMN ... SET NOT NULL` (Option A). M8 review แล้วไม่มี bug. Prettier + tsc clean. รอ DevOps resume `npm run db:migrate:latest` |
| 2026-05-04 | Phase 1 complete — 8 migrations + 7 models created, formatted, type-checked clean             |
| 2026-05-04 | Models include relationMappings + Type interfaces + jsonAttributes for JSONB fields           |
| 2026-05-04 | Q1=C Cartesian seed in M3, Q4=B FK RESTRICT in M6, M8 backfill via selected_package_uuid       |
| 2026-04-29 | Previous task: type-safety-any-cleanup — closed                                              |
