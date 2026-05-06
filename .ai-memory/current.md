# Current Focus — ปัจจุบันกำลังทำอะไรอยู่

> **อ่านไฟล์นี้เป็นอันดับแรกเมื่อเริ่ม session ใหม่** เพื่อให้รู้ว่าสถานะงานล่าสุดอยู่ตรงไหน
> **Lead agent อัพเดตไฟล์นี้** เมื่อเริ่ม / สลับ / จบ feature

---

## Active Feature

**Feature:** feature-management

**Type:** long-term

**Status:** Phase 2 **COMPLETE** — BA validation PASS-with-notes (0 critical, 5 non-critical doc-drift), pushed to `origin/ball/feature/feature-management` ทั้ง hw-be (`20eca15f`) + hw-sale-cms (`0734f7a`) (2026-05-06). Doc drift cleaned + effectiveMenus contract fix shipped. Pending Phase 3 dispatch.

**Started:** 2026-05-04

**Owner Agents:**

- Lead: Claude (main thread, session lead-20260505-1319-a1b)
- SA: Claude (done — analysis docs ready 2026-05-04)
- Developer: Claude (Opus 4.7 — full-stack agent, Phase 2 B1-B4 backend track + F1-F6 frontend track ทั้งหมด done) — *agent merged from backend + frontend on 2026-05-05*
- QA: _(pending — end of Phase 2 for tests P2.22-P2.25 + UAT F2.35)_
- Code Reviewer: _(pending — single review pass after all Phase 2 waves done)_

**Branch:** `ball/feature/feature-management` (both happywork-backend + happywork-sale-cms). Push per phase.

**Documents:**

- Requirement folder: `document/requirement/feature-management/`
- All decision docs ready: requirement.md, prd.md, ssd.md, summary.md, task-list.md, testcase.md, cr.md
- All engineering playbooks ready: backend.md, frontend.md, migration.md, checklist.md
- Open Questions §9: ทุกข้อ resolved 2026-05-04 (Q1=C, Q2=A, Q3=A, Q4=B, Q5=resolved by code review)

---

## Active Task (ที่กำลังทำจริงในขณะนี้)

**Phase 1 — Schema Foundation: COMPLETE 2026-05-04** — 8 migrations + 7 models deployed on dev DB (`hwv4_data4`); commit `2d3bc574` pushed to `origin/ball/feature/feature-management` 2026-05-05.

**Phase 2 — Master CRUD: IN PROGRESS (started 2026-05-05)**

User-confirmed sequencing (2026-05-05):
1. Developer ทำ backend + frontend tracks (in-session role switching default — agent merged 2026-05-05)
2. One wave at a time (Lead set scope ก่อน switch, poll สรุปหลังจบ)
3. Code review = single pass at end of Phase 2 (no per-wave review)
4. Tests deferred to QA at end of Phase 2 (Developer specialist does NOT write tests inline)
5. Branch = `ball/feature/feature-management`, push per phase

Wave breakdown:
- **B1** (Backend) → P2.1 helper `getAvailableMenuKeys()` + P2.2 `requireSuperAdmin` middleware
- **B2** (Backend) → P2.3-P2.9 Feature module (interface/repo/service/adapter/controller/routes/register)
- **B3** (Backend) → P2.10-P2.21 Package CRUD + Addon module
- **F1** (Frontend) → F2.1-F2.4 setup (rbac, paths, navigation, i18n)
- **F2** (Frontend) → F2.5-F2.8 reusable components
- **F3** (Frontend) → F2.9-F2.16 Redux slices (3 slices)
- **F4** (Frontend) → F2.17-F2.28 Page A Feature Management
- **F5** (Frontend) → F2.29-F2.31 Page B Package Management
- **F6** (Frontend) → F2.32-F2.34 Page C Addon Management
- **QA pass** → P2.22-P2.25 unit/integration tests + F2.35 manual UAT
- **Review pass** → Code Reviewer (FEAT-501 + FEAT-502)

**Wave 1-2 status:**
- ✅ B1 done — `getAvailableMenuKeys()` helper + `requireSuperAdmin` middleware
- ✅ F1 done — RBAC + paths + navigation + i18n setup
- ✅ B2 done — Feature module CRUD ครบชุด (interface/repo/service/adapter/controller/routes/register at `/api/v2/admin/features`)
- ✅ F2 done — 4 reusable components (multilingual-text-field, menu-keys-select, menu-tree-preview, feature-source-badge)
- ✅ Q6 resolved — Master Data Package Management ใช้ resource `master_packages`

**Wave 3 status:**
- ✅ B3 done — Package CRUD (P2.10-P2.15) mounted `/api/v2/admin/packages` + Addon module (P2.16-P2.21) mounted `/api/v2/admin/addons`
- ✅ F3 done — 3 Redux slices registered in rootReducer + rootSaga
- ✅ Q7 resolved (Decision B rename-only): legacy slice → `sale-dashboard-package-config-feature`; F3 Master Data slices ใช้ชื่อ `sale-dashboard-{feature,package,addon}-management` ตาม spec; pure rename, TS pass, 4 production views updated

**Wave B4 + B4-FE status (added 2026-05-05 — Stripe sync scope addition):**
- ✅ B4 done — `PATCH /api/v2/admin/packages/:packageUuid/identifier` (P2.15a-e); trx updates both `comp_packages.uuid` + `comp_companies.selected_package_uuid` (denormalized cache canonical)
- ✅ B4-FE done — `sale-dashboard-package-management` slice extended (action/reducer/saga/service/hook method) + reusable `package-uuid-edit-dialog.tsx` component + 12 i18n keys (en+th parity); UUID v4 strict regex matches backend; F2.36-F2.40 ticked
- Architectural finding: `comp_companies.selected_package_uuid` is canonical source-of-truth for runtime flows; SQL trigger auto-audits via `comp_subscription_historys`
- F5 (Page B Package Management) จะ mount `package-uuid-edit-dialog` ใน detail/edit view ภายหลัง

**Wave 4 status (2026-05-05):**
- ✅ F4 done — Page A Feature Management (11 section files + 4 pages + 53 i18n keys parity)
- ✅ F5 done — Page B Package Management + `master_packages` added to rbac.ts (Q6) + `PackageUuidEditDialog` mounted in detail view + URL navigation on success (B4-FE finding)
- ✅ F6 done — Page C Addon Management (8 section files + 4 pages + 89 i18n keys parity)
- ✅ Q8 resolved — addon billing_interval renamed monthly→month/yearly→year/one_time→null (frontend aligned with backend B3 Zod)

**Phase 2 Implementation = 100% complete** — Backend (B1+B2+B3+B4) + Frontend (F1+F2+F3+B4-FE+F4+F5+F6) + Q6+Q7+Q8 + Bug fix + URL fix + **Move-to-SaleDashboard refactor** (3 modules ย้ายจาก `v2/admin/` → `v2/sale-dashboard/` + middleware `validateSaleDashboardAccessToken` + URL `/api/v2/sale-dashboard/{features,packages,addons}`).

**Verified:**
- `validateSaleDashboardAccessToken` populates `req.user.role` — `requireSuperAdmin` chain works
- Frontend axios wrappers (`sd_method_*`) attach sale dashboard Bearer token from sessionStorage automatically
- Operational note: super_admin gating now via `sale_dashboard_users.role` (separate pool from `accounts` table)

**Status filter label fix (2026-05-05):**
- TS enum identifier `ARCHIVED` → `INACTIVE` ใน 3 enums (`FeatureStatus`, `PackageStatusTypeEnum`, `AddonStatus`)
- i18n key `packageManagement.status.archived` → `inactive` ("Inactive" / "ไม่ใช้งาน"); Feature + Addon already use shared `common.inactive` key
- 14 files modified, TS pass, en+th parity confirmed

**Status value migration (2026-05-05):**
- **Frontend:** 3 enums wire literal `"archived"` → `"inactive"` + 3 redux type unions + 3 service JSDoc
- **Backend:** Zod schemas already use `'inactive'` (drift discovered — Phase 2 was inconsistent: filter accepts `inactive`, soft-delete writes `archived`). Fixed 3 soft-delete repos: `StatusTypeEnumCode.ARCHIVED` → `INACTIVE` (shared enum constant `INACTIVE = 'inactive'` already exists at `general.ts:13`)
- **DB:** `status_type` = `VARCHAR(25)` no constraint; 0 archived rows in current data → no migration created
- TS pass both sides, en+th parity unchanged

**Edit back navigation (2026-05-05):**
- 3 list views (`feature/package/addon-list-view.tsx`): `handleEdit` append `?from=list`
- 3 edit views (`feature/package/addon-edit-view.tsx`): `useSearchParams()` branch — `from=list` → root; else → detail (safe default)
- Detail views ไม่แก้ — ไม่ส่ง `?from` → branch ทำงานถูกต้อง (back → detail)
- EBN.1-EBN.8 ticked ใน checklist.md; TS pass + prettier conformant

**Currently:** Phase 2 closeout complete (2026-05-06):
- ✅ BA validation done — Verdict **PASS-with-notes** (0 critical / 5 non-critical doc-drift); report at `document/requirement/feature-management/ba-validation-phase2.md`
- ✅ Phase 2 commits already on origin verified (hw-be `20eca15f` + hw-sale-cms `f61b70d`); 0/0 ahead/behind
- ✅ Doc drift fixes — checklist.md P2.1+P2.2 ticked; P2.3-P2.21 paths synced `v2/admin` → `v2/sale-dashboard`; JSDoc in 3 service files synced
- ✅ Sale-cms contract fix discovered + pushed — 2 commits `b21eaaf` (effectiveMenus `menus` → `menuKeys` to match backend `Promise<readonly string[]>`) + `0734f7a` (JSDoc URL drift) → `origin/ball/feature/feature-management`
- 🟡 Outstanding (non-blocker, defer to Phase 3 prep):
  - P2.25 integration test — written 19 TCs, OOM blocked local, ต้องรันบน CI (`--max-old-space-size=4096`)
  - F2.35 manual UAT — pending; รัน post-deploy
  - UT-6 / UT-7 / UT-8 standalone spec files — covered indirectly, defer Phase 3

---

## Paused / Waiting

| Feature                  | Status | Reason                                                                |
| ------------------------ | ------ | --------------------------------------------------------------------- |
| type-safety-any-cleanup  | review | งาน implementation เสร็จแล้ว 2026-04-29 รอ final review/closeout      |

---

## Recent Activity (ย้อนหลัง 5 รายการ)

_(จะถูกอัพเดตอัตโนมัติโดย Lead เมื่อมี activity)_

| Date       | Agent     | Action                                                                                                |
| ---------- | --------- | ----------------------------------------------------------------------------------------------------- |
| 2026-05-06 | Lead      | Phase 2 closeout — sale-cms contract fix (effectiveMenus menus→menuKeys) + JSDoc drift; 2 commits pushed `b21eaaf`+`0734f7a`; checklist + memory updated |
| 2026-05-06 | Lead      | Doc drift fix — P2.1+P2.2 ticked, P2.3-P2.21 paths synced v2/admin→v2/sale-dashboard, JSDoc 3 service files synced |
| 2026-05-06 | BA        | BA validation Phase 2 — Verdict PASS-with-notes (0 critical, 5 non-critical); FR/middleware/URL/enum/Q6-Q8/CR all verified; report file written |
| 2026-05-05 | QA        | P2.22-P2.24 unit tests written + PASS (56 tests); P2.25 integration test written (OOM blocker local); CR findings reported |
| 2026-05-05 | Frontend  | Edit back nav — 3 list-views append ?from=list; 3 edit-views branch on useSearchParams(); EBN.1-EBN.8 done |
| 2026-05-05 | Lead      | Spec docs sync — ssd.md + backend.md replaced `/api/v2/admin/` → `/api/v2/` (whole-file replace_all)     |
| 2026-05-05 | Lead      | Spec docs sync — ssd.md + backend.md → /api/v2/sale-dashboard/{features,packages,addons}             |
| 2026-05-05 | Backend+Frontend | Move-to-SaleDashboard — 6 folders moved + middleware swap + 11 URL builders + /sale-dashboard/ |
| 2026-05-05 | Backend+Frontend | URL fix wave — `/admin` segment removed from OpenAPI basePath (3 routes) + URL builders (11) |
| 2026-05-05 | Frontend  | Bug fix done — RSC error (3 feature pages) + max-depth loop (3 hooks useCallback); HTTP 200 verified    |
| 2026-05-05 | Lead      | Q8 spec docs updated (frontend.md §3, ssd.md §10 Q8, requirement.md §9 Q8 entry)                        |
| 2026-05-05 | Frontend  | Q8 fix done — AddonBillingInterval rename; addon CRUD aligned with backend; TS pass + parity            |
| 2026-05-05 | Frontend  | Wave F6 done — Page C Addon Management; flagged Q8 billing_interval value mismatch                      |
| 2026-05-05 | Frontend  | Wave F5 done — Page B Package + master_packages rbac + PackageUuidEditDialog mounted with URL nav      |
| 2026-05-05 | Frontend  | Wave F4 done — Page A Feature Management (11 section files + 4 pages + 53 i18n keys)                    |
| 2026-05-05 | Frontend  | Wave B4-FE done — package-management slice extended + reusable uuid-edit dialog (F5 will mount)         |
| 2026-05-05 | Backend   | Wave B4 done — PATCH /packages/:uuid/identifier (Stripe sync); flagged selectedPackageUuid canonicality |
| 2026-05-05 | Lead      | Plan updated for Stripe-sync scope (requirement.md FR-2, ssd.md §4.2, backend.md, checklist P2.15a-e)   |
| 2026-05-05 | Lead      | Q7 spec docs updated (frontend.md §3, ssd.md §10, requirement.md §9 — Q7 entry added)                  |
| 2026-05-05 | Frontend  | Q7 rename done — legacy slice + F3 slices renamed; pure rename verified; TS pass                        |
| 2026-05-05 | Frontend  | Wave F3 done — 3 redux slices `-master`; Q7 (legacy slice collision) escalated                          |
| 2026-05-05 | Backend   | Wave B3 done — packageCrud + addon mounted at /api/v2/admin/packages|/addons                            |
| 2026-05-05 | Lead      | Wave 3 dispatch (B3 Package+Addon + F3 Redux slices) — sub-agent inline parallel                       |
| 2026-05-05 | Frontend  | Wave F2 done — 4 reusable components (5-source enum, 2-level menu grouping, RHF Controller pattern)    |
| 2026-05-05 | Backend   | Wave B2 done — Feature module CRUD ครบ (mounted at /api/v2/admin/features)                              |
| 2026-05-05 | Lead      | Q6 resolved (Decision A: master_packages); spec docs updated                                            |
| 2026-05-05 | Frontend  | Wave F1 done — rbac/paths/nav/i18n setup; flagged `packages` resource collision for Lead              |
| 2026-05-05 | Backend   | Wave B1 done — `getAvailableMenuKeys()` + `requireSuperAdmin` middleware                              |
| 2026-05-05 | Lead      | Phase 1 commit `2d3bc574` pushed to `origin/ball/feature/feature-management`; Wave B1+F1 dispatched   |
| 2026-05-04 | DevOps | Phase 1 P1.9 complete: M1-M8 applied on dev (batches 31-33), all 6 verification queries passed       |
| 2026-05-04 | Backend | Hotfix M6/M7 duplicate company_id via Option A (table.foreign + raw ALTER NOT NULL); M8 verified safe |
| 2026-05-04 | DevOps | Migrate retry cycle: pre-marked 2 drift items, applied 30 unrelated migrations (batch 32), failed M6 |
| 2026-05-04 | Claude | Switched active feature to feature-management; Phase 1 dispatch incoming                              |
| 2026-05-04 | Claude | Resolved all 5 Open Questions in requirement.md §9 (Q1=C, Q2=A, Q3=A, Q4=B, Q5=code-resolved)         |
| 2026-05-04 | Claude | Updated migration.md M3+M6 + ssd.md §3.1.6+§6.2+§10 to reflect Q1-Q4 decisions                        |
| 2026-05-04 | Claude | Added rules.md §1.4 — Question Batching Protocol                                                      |
| 2026-04-29 | Codex  | Scoped implementation back to model type exports only; no-explicit-any lint set to warning            |

---

## Note for AI agents reading this

หาก section "Active Feature" ว่าง → workspace ยังไม่มีงาน active กำลังทำอยู่

- ถ้าผู้ใช้สั่งงานใหม่ → Lead ตัดสินประเภทงานก่อน (ดู `.ai-memory/rules.md` ข้อ 1)
- ถ้าผู้ใช้ถาม "ทำอะไรอยู่" → ตอบว่ายังไม่มีงาน active และพร้อมรับงานใหม่
