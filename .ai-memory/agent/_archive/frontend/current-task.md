---
agent: frontend
updated: 2026-05-05
---

# Frontend Agent — Current Task

> งานที่ Frontend กำลังถืออยู่ — Lead เป็นคน assign

---

## Active Task

**Task ID:** Wire Value Migration — `"archived"` → `"inactive"` (3 enums + 3 Redux types + 3 JSDoc comments)
**Feature:** feature-management
**Description:** Pure wire-value flip ใน Phase 2 — match backend Zod schema migration ที่ทำขนานกัน (backend agent เป็นคนแก้). หลัง wave SFL ก่อนหน้า TS identifier เป็น `INACTIVE` แล้ว แต่ wire literal ยังเป็น `"archived"` — wave นี้ flip wire literal ของ 3 enums (FeatureStatus / PackageStatusTypeEnum / AddonStatus) ให้เป็น `"inactive"` พร้อม sync 3 Redux type aliases (`{Feature,Package,Addon}StatusType = "active" | "inactive"`) + JSDoc soft-delete comment ใน 3 service request files (documentation sync). Display label "Inactive" / "ไม่ใช้งาน" คงเดิม (i18n keys ไม่ต้องแก้ — `common.inactive` + `packageManagement.status.inactive` มีอยู่แล้วจาก SFL wave). แก้รวม 9 ไฟล์.
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/checklist.md` "Wire Value Migration" section (added 2026-05-05)

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 <!-- done: 2026-05-05 -->
- Prettier: 9 files all conformant (no reformat needed) <!-- done: 2026-05-05 -->
- Grep: `'archived'|"archived"` across Phase 2 scope (sections + components + store/{actions,reducers,sagas,services,types}) → 0 hits <!-- done: 2026-05-05 -->
- i18n parity: featureManagement (53/53), packageManagement (103/103), addonManagement (89/89) — en + th equal; no orphan `archived` keys; no new keys needed <!-- done: 2026-05-05 -->
- Dev server (port 4001) HTTP 200 on `/dashboard/feature-management`. Full Network-tab smoke (`statusType=inactive`) deferred to user — depends on backend Zod migration parallel completion <!-- done: 2026-05-05 -->

---

## Earlier Task

**Task ID:** Status Filter Label Fix — "Archived" → "Inactive" / "ไม่ใช้งาน" (3 pages: Feature/Package/Addon)
**Feature:** feature-management
**Description:** Display-only label change — filter dropdown + status chip ของ 3 หน้า Master CRUD ต้องแสดง "Active" / "Inactive" (en) และ "ใช้งาน" / "ไม่ใช้งาน" (th) แทน "Active" / "Archived". Wire value `archived` คงเดิมทั้งหมด (backend `statusType` enum). Approach: Option B — rename TS enum identifier `ARCHIVED` → `INACTIVE` (string literal value `"archived"` คงเดิม) + rename i18n key `packageManagement.status.archived` → `inactive`. Feature/Addon enums map status display ผ่าน `common.active` / `common.inactive` อยู่แล้ว — แค่ rename TS identifier เพื่อ semantic consistency. แก้ 14 ไฟล์: 3 enums + 3 list-views + 2 forms + 2 form utils + 1 status-option + 1 detail-view + 1 table + 2 i18n.
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/checklist.md` "Status Filter Label Fix" section (added 2026-05-05)

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 <!-- done: 2026-05-05 -->
- Prettier: 14 files all conformant (no reformat needed) <!-- done: 2026-05-05 -->
- Grep: `FeatureStatus.ARCHIVED|PackageStatusTypeEnum.ARCHIVED|AddonStatus.ARCHIVED|packageManagement.status.archived` → 0 hits <!-- done: 2026-05-05 -->
- Wire value verified: TS enum string-literal value still `"archived"` ในทุก 3 enums — payload `statusType=archived` ไม่เปลี่ยน <!-- done: 2026-05-05 -->
- i18n parity: 3 management namespaces (`featureManagement.*`, `packageManagement.*`, `addonManagement.*`) — en + th มี keys ครบเท่ากัน <!-- done: 2026-05-05 -->
- Browser smoke deferred to user — HMR จะ pick up; ตรวจ Network tab `statusType=archived` ใน list filter query string

---

## Earlier Task

**Task ID:** Move-to-SaleDashboard URL update — Add `/sale-dashboard/` segment to Phase 2 Master CRUD URL builders
**Feature:** feature-management
**Description:** Pure URL string update in `src/store/services/sale-dashboard.ts` lines 100-128 (11 builders). User instructed backend to move 3 Phase 2 modules (`feature` / `packageCrud` / `addon`) from `v2/admin/...` → `v2/sale-dashboard/...` + switch middleware to `validateSaleDashboardAccessToken`. New backend mount = `saleDashboardV2Router` at `/api/v2/sale-dashboard`. Re-added the `/sale-dashboard/` segment to all 11 builders (3 feature + 5 package + 3 addon) → `/api/v2/sale-dashboard/{features|packages|addons}/...`. Builder symbols unchanged (`adminFeatures`, `adminPackageIdentifier`, etc.) so 3 request files + downstream consumers continue type-checking. Section comments updated to reference `saleDashboardV2Router` mount + sale dashboard token. Pre-existing subscription URLs at lines 72-76 (`/api/v2/admin/subscription/*`) intentionally untouched — out of scope.
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/checklist.md` Move-to-SaleDashboard URL update section (added 2026-05-05)

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 <!-- done: 2026-05-05 -->
- Prettier: `npx prettier --write src/store/services/sale-dashboard.ts` already conformant <!-- done: 2026-05-05 -->
- Grep: `api/v2/features|api/v2/packages|api/v2/addons` across `src/` returns zero hits (consumers route through `saleDashboardService().adminFeatures` etc.) <!-- done: 2026-05-05 -->
- Token handling verified: `sd_method_*` in `src/store/services/sale-dashboard-request.ts` reads `saleDashboardAccessToken` and attaches `authorization: Bearer ...` — exactly what `validateSaleDashboardAccessToken` middleware expects. No auth client change needed. <!-- done: 2026-05-05 -->
- Browser smoke deferred to user (HMR will pick up once backend agent finishes parallel move; Network tab should show `/api/v2/sale-dashboard/features` etc.). <!-- done: 2026-05-05 -->

---

## Earlier Task

**Task ID:** URL Fix — Drop `/admin` segment from Phase 2 Master CRUD URL builders
**Feature:** feature-management
**Description:** Pure URL string update in `src/store/services/sale-dashboard.ts` lines 100-125 (11 builders). Backend `v2AdminRouter` mounts at `/api/v2` (verified by Lead), not `/api/v2/admin`. Renamed paths only (`/api/v2/admin/{features|packages|addons}/...` → `/api/v2/{features|packages|addons}/...`). Builder symbol names unchanged (`adminFeatures`, `adminPackageIdentifier`, etc.) so all 3 request files (feature/package/addon) and downstream consumers continue type-checking. Pre-existing subscription URLs at lines 72-76 (`/api/v2/admin/subscription/*`) intentionally untouched — out of Phase 2 scope. Grep confirmed no inline hardcoded URLs under sections/components — only JSDoc comments mention legacy `/api/v2/admin/...` paths (informational, not fetched).
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/checklist.md` URL Fix section (added 2026-05-05)

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 <!-- done: 2026-05-05 -->
- Prettier: `npx prettier --write src/store/services/sale-dashboard.ts` already conformant <!-- done: 2026-05-05 -->
- Dev server (port 4001): HTTP 200 on `/dashboard/feature-management`. Full Phase 2 network round-trip smoke test deferred to user — backend OpenAPI basePath fix is parallel scope. <!-- done: 2026-05-05 -->

---

## Earlier Task

**Task ID:** Q8 Fix — Addon `billingInterval` value rename (frontend ↔ backend B3 alignment)
**Feature:** feature-management
**Description:** Pure value rename: `AddonBillingInterval` in `src/types/store/sale-dashboard-addon-management.ts` from `'monthly' | 'yearly' | 'one_time'` → `'month' | 'year'` (matches backend B3 `z.enum(['month','year']).nullable()`). Drop `'one_time'` literal — `null` now carries that semantic (mirrors Package pattern fixed in F5). Updated 8 files: type, enum, form util (Yup schema + mappers + payload builders), form Select, table + detail-view display, en/th i18n (`addonManagement.billing.monthly→month`, `yearly→year`, kept `one_time` as label key for null). Form Select shows 3 options (One-time / Monthly / Yearly) where One-time → form value `""` → wire `null`.
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/checklist.md` Q8 Fix section + `happywork-backend/src/modules/v2/admin/addon/addon.interface.ts` lines 34/81/113

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 <!-- done: 2026-05-05 -->
- Prettier: `npx prettier --write` ran on 8 files (all already conformant) <!-- done: 2026-05-05 -->
- i18n parity: 89 `addonManagement.*` keys in en + th — no drift <!-- done: 2026-05-05 -->
- Tests: deferred per task instruction

---

## Earlier Task

**Task ID:** F2.32 – F2.34 (Wave F6 — Page C: Addon Management — section + views + App Router pages)
**Feature:** feature-management
**Description:** Built full Addon Management UI under `src/sections/addon-management/` (enum, Yup form util, table, shared form, delete dialog, conditional max-quantity sub-form, features section + edit dialog) plus list/create/detail/edit views and App Router pages (4) with `<Can resource="addons">` guards. Wired to `useSaleDashboardAddonManagement` (Wave F3 + Q7) and `useSaleDashboardFeatureManagement` for the multi-select features dialog. Extended `addonManagement` i18n namespace (en + th, 89 keys parity). Defense-in-depth Yup conditional `is_quantifiable=true → max_quantity ≥ 1`. Type-only extension on `Addon`/`CreateAddonRequest`/`UpdateAddonRequest` to add `priceAmount` / `currency` / `companyCount` (Q7 slice stopped at `billingInterval`).
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05
**Doc reference:** `document/requirement/feature-management/frontend.md` §1.3 + checklist F2.32-F2.34 + ssd.md §4.3

**Verification:**

- TypeScript: `npx tsc --noEmit` exits 0 (clean) <!-- done: 2026-05-05 -->
- Prettier: `npx prettier --write` applied to 19 files (12 reformatted, 7 already conformant) <!-- done: 2026-05-05 -->
- i18n parity: 89 `addonManagement.*` keys in en + th — no drift <!-- done: 2026-05-05 -->
- Tests: deferred per task instruction

---

## Earlier Task (Wave F5)

**Task ID:** F2.29 – F2.31 (Wave F5 — Page B: Package Management) + F5.0 pre-step (rbac `master_packages`) + B4-FE mount
**Feature:** feature-management
**Description:** Implement full Page B: Package Management UI — section files (enum, form schema), components (table, form, delete dialog, features section, features edit dialog, effective menus preview), views (list/create/detail/edit), pages (App Router list/create/[id]/[id]/edit). Mount `<PackageUuidEditDialog>` in detail view (B4-FE finding) with URL navigation on success. Pre-step: add `master_packages` resource to `rbac.ts` (Q6) — separate from legacy `packages`.
**Assigned by:** Lead
**Started:** 2026-05-05
**Completed:** 2026-05-05

Previous: F2.36 – F2.40 (Wave B4-FE — Package UUID Update / Stripe sync — counterpart of Backend Wave B4)

**Doc reference:** `document/requirement/feature-management/frontend.md` §1.2 + §4 (Q6) + `ssd.md` §4.2 + `checklist.md` Frontend Page B section

**Acceptance criteria:**

- [x] `master_packages` added to `Resource` union + super_admin permissions in `rbac.ts` (legacy `packages` untouched) <!-- done: 2026-05-05 -->
- [x] `package.enum.ts` with billing/status/currency options <!-- done: 2026-05-05 -->
- [x] `utils/package-form.ts` with Yup schema + form↔backend mappers (price ≥ 0, code regex, max ≥ min) <!-- done: 2026-05-05 -->
- [x] `package-table.tsx` with code/name/billing/price/userLimit/flags/status columns + sortable + RBAC-gated actions <!-- done: 2026-05-05 -->
- [x] `package-form.tsx` shared (create+edit) with MultilingualTextField + native MUI inputs <!-- done: 2026-05-05 -->
- [x] `package-delete-dialog.tsx` with inline warning + backend-rejection error display <!-- done: 2026-05-05 -->
- [x] `package-features-section.tsx` with linked feature list + Edit button <!-- done: 2026-05-05 -->
- [x] `package-features-edit-dialog.tsx` multi-select with search + dispatches `sdReplacePackageFeaturesRequest` <!-- done: 2026-05-05 -->
- [x] `package-effective-menus-preview.tsx` wraps `<MenuTreePreview>` + loads via `sdGetPackageEffectiveMenusRequest` <!-- done: 2026-05-05 -->
- [x] 4 views (list/create/detail/edit) — detail view mounts `<PackageUuidEditDialog>` and navigates URL on success <!-- done: 2026-05-05 -->
- [x] 4 App Router pages with `<Can resource="master_packages">` guard fallback <!-- done: 2026-05-05 -->
- [x] i18n: ~80 `packageManagement.*` keys in en + th (parity verified) + 2 `common.permission_denied_*` <!-- done: 2026-05-05 -->
- [x] `npx tsc --noEmit` passes (no new errors in package-management scope; addon-management errors are pre-existing/out of scope) <!-- done: 2026-05-05 -->
- [x] Prettier ran on all 20 modified files <!-- done: 2026-05-05 -->

## Pre-implementation Checklist

- [x] อ่าน current-task.md <!-- done: 2026-05-05 -->
- [x] อ่าน frontend.md §1.2 + §4 (Q6) + ssd.md §4.2 + checklist.md Page B section <!-- done: 2026-05-05 -->
- [x] อ่าน existing patterns (survey list/create/detail/edit views, package-uuid-edit-dialog, MultilingualTextField, MenuTreePreview, hook-form components, Can permission guard) <!-- done: 2026-05-05 -->
- [x] อ่าน rbac.ts ก่อนแก้ — confirmed legacy `packages` resource intact and used by `package-config-feature-view` <!-- done: 2026-05-05 -->
- [x] Backend API พร้อม — Wave B3 + B4 endpoints live <!-- done: 2026-05-05 -->

## Files Modified

| Path                                                                              | Change                                                                                                                              | Status |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------ |
| `src/utils/rbac.ts`                                                               | Added `master_packages` to `Resource` union + 4 PERMISSIONS rows                                                                    | done   |
| `src/types/store/sale-dashboard-package-management.ts`                            | Extended `Package` + request types with priceAmount, currency, userLimitMin/Max, isContactUs; aligned `billingInterval` to month/year | done   |
| `src/sections/package-management/package.enum.ts`                                 | New                                                                                                                                 | done   |
| `src/sections/package-management/utils/package-form.ts`                           | New: Yup schema + form↔backend mappers                                                                                              | done   |
| `src/sections/package-management/components/package-table.tsx`                    | New                                                                                                                                 | done   |
| `src/sections/package-management/components/package-form.tsx`                     | New                                                                                                                                 | done   |
| `src/sections/package-management/components/package-delete-dialog.tsx`            | New                                                                                                                                 | done   |
| `src/sections/package-management/components/package-features-section.tsx`         | New                                                                                                                                 | done   |
| `src/sections/package-management/components/package-features-edit-dialog.tsx`     | New                                                                                                                                 | done   |
| `src/sections/package-management/components/package-effective-menus-preview.tsx`  | New                                                                                                                                 | done   |
| `src/sections/package-management/package-list-view.tsx`                           | New                                                                                                                                 | done   |
| `src/sections/package-management/package-create-view.tsx`                         | New                                                                                                                                 | done   |
| `src/sections/package-management/package-detail-view.tsx`                         | New: mounts `<PackageUuidEditDialog>` (manage) + `router.replace` to new uuid on success                                            | done   |
| `src/sections/package-management/package-edit-view.tsx`                           | New                                                                                                                                 | done   |
| `src/app/dashboard/package-management/page.tsx`                                   | New: `<Can action="view">`                                                                                                          | done   |
| `src/app/dashboard/package-management/create/page.tsx`                            | New: `<Can action="create">`                                                                                                        | done   |
| `src/app/dashboard/package-management/[id]/page.tsx`                              | New: `<Can action="view">`                                                                                                          | done   |
| `src/app/dashboard/package-management/[id]/edit/page.tsx`                         | New: `<Can action="edit">`                                                                                                          | done   |
| `src/locales/langs/en.json`                                                       | Extended `packageManagement.*` (~80 keys) + `common.permission_denied_*`                                                            | done   |
| `src/locales/langs/th.json`                                                       | Mirror of en (parity verified)                                                                                                      | done   |
| `document/requirement/feature-management/checklist.md`                            | Ticked F2.29/F2.30/F2.31 + sub-items + Wave F5 Notes                                                                                | done   |

## Deviations from PRD/SSD

| Description                                                                                                                                       | Reason                                                                                                                                                                          | Reported?           |
| ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| Extended `Package` redux type with priceAmount/currency/userLimitMin-Max/isContactUs and corrected `billingInterval` to `'month'\|'year'`         | F3 types were incomplete vs backend B3 contract; alignment was needed for the form to type-check. Backend zod schema (backend.md §2.2) is the source of truth.                  | Yes — flagged below |
| Delete dialog does not pre-load company usage count                                                                                               | SSD `getPackageByUuid` response only carries `features`, not `companyCount`. The dialog shows an inline warning + surfaces backend rejection if companies reference the package. | Yes — flagged below |
| Page-level guards use `<Can fallback={...}>` instead of redirect                                                                                  | Existing app router pages do not perform redirects; using `<Can>` with fallback follows the pattern used by `permission-guard.tsx`.                                              | n/a                 |

## Blockers

_(none)_

## Recent updates (latest 5)

| Date       | Update                                                                                                                                                |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-05-05 | F2.31 4 App Router pages created with `<Can fallback>` gating. i18n parity verified via flatten script.                                                |
| 2026-05-05 | F2.30 4 views created. Detail view mounts `<PackageUuidEditDialog>` (gated `manage`) + `router.replace` to new uuid on success. Edit view disables code. |
| 2026-05-05 | F2.29 6 components created (table, shared form, delete dialog, features section, features edit dialog, effective menus preview).                       |
| 2026-05-05 | F5.0 pre-step: added `master_packages` resource to rbac.ts. Legacy `packages` row + `package-config-feature-view` consumer untouched.                  |
| 2026-05-05 | i18n: extended `packageManagement.*` ~30 → ~80 keys (en + th parity). Added `common.permission_denied_{title,message}`.                                |
