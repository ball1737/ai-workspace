---
type: ba-validation
phase: 3
validator: BA agent
date: 2026-05-06
---

# BA Validation Report — feature-management Phase 3

**Date:** 2026-05-06
**Verdict:** PASS-with-notes
**Scope:** Per-Company Toggle + Permission Resolver (Wave W1–W4 + CR Fix Wave)

---

## Summary

Phase 3 implementation covers all required functional requirements (FR-4, FR-5) and all checklist items P3.1–P3.12, F3.1–F3.10. The 75/75 test suite passes. All CR High/Medium fixes are verified in code. DEF-001 is confirmed false positive. Two deferred items (L1 logger.warn, L3 cosmetic) and one pending UAT (F3.11) are non-blocking for push.

**Findings:** 0 critical / 3 non-critical (notes)

---

## Section 1 — Requirement Coverage (FR-4, FR-5, Q3, Q4, Q9)

| Req | Acceptance Criteria | Status | Evidence |
|-----|---------------------|--------|---------|
| FR-4 (AC-1) | Features tab shows all features + source | PASS | `company-features-tab-view.tsx` loads via `getCompanyFeatures`; `computeCompanyFeatureItem` returns 5-source enum |
| FR-4 (AC-2) | Toggle → upsert override_status enabled/disabled + reason | PASS | `setCompanyFeatureOverrideService` → `upsertOverrideRepository` (ON CONFLICT UPSERT); reason field max 500 validated in Zod |
| FR-4 (AC-3) | Reset → delete row from comp_company_features | PASS | `removeCompanyFeatureOverrideService` → `removeOverrideRepository` (hard delete) |
| FR-4 (AC-4) | After toggle → state reloaded correctly | PASS | `getCompanyFeatureItemAfterMutation` refetch list after mutation; reducer `replaceRowByFeatureUuid` updates single row |
| FR-5 (AC-3) | Resolver logic: effective = (pkg ∪ addon ∪ override+) − override− | PASS | `resolveCompanyEffectiveFeatureIdsService` in `permissionResolver.service.ts:99–147`; union then delta in correct order |
| FR-5 (AC-5) | In-request memoization | PASS | `ResolverCache = Map<string, unknown>` explicit param pattern (Option C); 3 cache keys (featureIds/menuKeys/packageId) |
| Q3=A | addon expiry: `expires_at IS NULL OR expires_at > NOW()` | PASS | `permissionResolver.repository.ts:82–88`: `.whereNull('expires_at').orWhere('expires_at', '>', knex.fn.now())` |
| Q4=B | archived feature: resolver filter `feature.status_type='active'` at DB JOIN level | PASS | `getCompanyOverridesRepository`, `getAddonFeatureIdsRepository`, `getPackageFeatureIdsRepository` all apply `status_type=active` filter |
| Q9=A | Route pattern: `/api/v2/sale-dashboard/companies/:companyUuid/features` | PASS | `companyFeature.routes.ts:18` `basePath = '/api/v2/sale-dashboard/companies'`; mounted at `sale-dashboard.routes.ts:288` under `/companies` |
| FR-6 (AC-1) | requireSuperAdmin on all 3 routes | PASS | `companyFeature.routes.ts:28,43,60` — all 3 routes (GET/PUT/DELETE) have `requireSuperAdmin` |
| BR-7 | Effective = `(pkg ∪ addon ∪ override+) − override−` | PASS | `permissionResolver.service.ts` step (4)–(5) builds baseSet then applies override delta |
| FR-5 resolver signatures (backend.md §2.5) | `resolveCompanyEffectiveFeatureIdsService(companyId, cache?)` | PASS | Signature matches; `resolveCompanyEffectiveMenuKeysService` and `filterPermissionByMenuKeysService` also match spec |

---

## Section 2 — Checklist Completeness

### Backend P3.1–P3.9

| Item | Ticked | Code Verified |
|------|--------|---------------|
| P3.1 `companyFeature.interface.ts` | [x] | Zod schemas (get/set/remove) + types `CompanyFeatureItem`, `OverrideRowDb` alias to `CompanyOverrideRow` (H1 unification) |
| P3.2 `companyFeature.repository.ts` | [x] | 5 functions: `getCompanyByUuidRepository`, `listActiveFeaturesRepository`, `getFeatureByUuidForOverrideRepository`, `upsertOverrideRepository`, `removeOverrideRepository`; override list read removed (H1 — uses resolver repo) |
| P3.3 `companyFeature.service.ts` | [x] | 3 public services + `buildCompanyFeatureList`; imports `resolveEffectivePackageIdService` (M3), uses `getCompanyOverridesRepository` from resolver (H1) |
| P3.4 `companyFeature.adapter.ts` | [x] | `computeCompanyFeatureItem` + `toCompanyFeatureItems`; source priority: override > package > addon > default-disabled |
| P3.5 `companyFeature.controller.ts` | [x] | 3 controllers; `resolveActorUuid` throws 401 if missing; uses `res.success` / `handleError` pattern |
| P3.6 routes + mount | [x] | `companyFeature.routes.ts`; middleware chain `validateSaleDashboardAccessToken` → `requireSuperAdmin` → `validateRequest`; specific paths registered before generic; mounted at `/companies` in `sale-dashboard.routes.ts:288` |
| P3.7 `permissionResolver.interface.ts` | [x] | `EffectiveFeatureIds`, `EffectiveMenuKeys`, `CompanyOverrideRow`, `UserPermissionRow`, `ResolverCache` |
| P3.8 `permissionResolver.repository.ts` | [x] | 7 functions: `getCompanyPackageIdRepository` (L4 fixed — select only `packageId`), `getSeedPackageIdRepository`, `getPackageFeatureIdsRepository`, `getActiveCompanyAddonIdsRepository`, `getAddonFeatureIdsRepository`, `getCompanyOverridesRepository`, `getFeatureMenuKeysRepository` (M2 jsonb_typeof guard added), `getUserPermissionJsonbRepository` |
| P3.9 `permissionResolver.service.ts` | [x] | 4 public services; `resolveEffectivePackageIdService` extracted (M3); in-request cache via ResolverCache; SSD §7 error codes: `COMPANY_NOT_FOUND`, `SEED_PACKAGE_NOT_FOUND`, `PERMISSION_DATA_NOT_FOUND`, `PERMISSION_DATA_CORRUPTED` |

### Backend Tests P3.10–P3.12

| Item | Ticked | Verified |
|------|--------|---------|
| P3.10 `companyFeature.service.test.ts` | [x] | 26 cases (TC-1..TC-23b); covers 5-source attribution, Seed fallback, Q4=B, Q3=A, 409 deadlock/serialization, idempotent remove, whitespace-reason normalization |
| P3.11 `permissionResolver.service.test.ts` | [x] | 32 cases; covers `resolveEffectivePackageId` (5), effective feature ids (10), menu keys (5), filter (8), user permission (4) |
| P3.12 integration test | [x] | 17 cases (GET 4, PUT 8, DELETE 3, round-trip 1, sanity 1); URL path `/api/v2/sale-dashboard/companies/...` |

### Frontend Slice F3.1–F3.4

| Item | Ticked | Verified |
|------|--------|---------|
| F3.1 types (`sale-dashboard-company-features.ts`) | [x] | `CompanyFeatureSource` 5-variant; `CompanyFeatureItem`; request/response types; `Root` state shape |
| F3.2 service (`company-features-request.ts`) | [x] | 3 HTTP wrappers; URL builders `saleDashboardCompanyFeatures` + `saleDashboardCompanyFeatureDetail` in `sale-dashboard.ts:137–143` |
| F3.3 actions/reducer/saga + register | [x] | `takeLatest` for GET, `takeEvery` for set/remove (no `console.*`); reducer `replaceRowByFeatureUuid` immutable; registered in 4 index files |
| F3.4 hook `useSaleDashboardCompanyFeatures` | [x] | 5 dispatchers wrapped `useCallback([dispatch])`; returns state + dispatchers |

### Frontend Page D F3.5–F3.10

| Item | Ticked | Verified |
|------|--------|---------|
| F3.5 `company-features-tab-view.tsx` | [x] | Load on mount, cleanup on unmount; sourceFilter + dialog state; counts + filteredList via useMemo; toast for errors; L2 fix: `wasSubmittingRef` + useEffect for dialog auto-close |
| F3.6 `company-feature-list.tsx` | [x] | File exists in `/feature-tab/components/` |
| F3.7 `company-feature-row.tsx` | [x] | File exists; per-row `disabled = togglingFeatureUuid === item.featureUuid` |
| F3.8 `company-feature-toggle-confirm-dialog.tsx` | [x] | `isOverrideSource` check → 3-button (Cancel + Reset + Set) vs 2-button; reason field max 500; buttons disabled on `isSubmitting` |
| F3.9 `company-feature-source-filter.tsx` | [x] | File exists; 6 options (all + 5 sources) |
| F3.10 Tab "Features" in `list-view.tsx` | [x] | `list-view.tsx:357,365–367`: `<Tab value="features">` + conditional `<CompanyFeaturesTabView companyUuid={...}/>` |
| F3.11 Manual UAT | [ ] | Pending — deferred post-push (non-blocking) |

### CR Fix Wave

| Fix | Addressed | Evidence |
|-----|-----------|---------|
| H1: Remove `getOverridesByCompanyRepository`, use resolver's `getCompanyOverridesRepository` | DONE | `companyFeature.repository.ts:77–78` (comment: "H1 unification: override list read query removed"); `companyFeature.service.ts:10,63` imports/uses `getCompanyOverridesRepository` from resolver |
| H2: Remove `console.log`/`console.error` from saga | DONE | `sale-dashboard-company-features.ts` saga has no `console.*`; startup log removed |
| M1: Remove `'23505'` from error catch | DONE | `companyFeature.service.ts:136–143` catches only `'40001'` and `'40P01'`; comment explains why 23505 cannot occur |
| M2: `jsonb_typeof(f.menu_keys) = 'array'` guard | DONE | `permissionResolver.repository.ts:164`: `.whereRaw("jsonb_typeof(f.menu_keys) = 'array'")` |
| M3: Replace private `resolveEffectivePackageId` with `resolveEffectivePackageIdService` | DONE | `companyFeature.service.ts:55` calls `resolveEffectivePackageIdService(companyId)` from resolver; private helper removed |
| L2: Dialog stays open during in-flight submit | DONE | `company-features-tab-view.tsx:61–140`: `wasSubmittingRef` + `useEffect` watching `togglingFeatureUuid`; `handleConfirmSet`/`handleConfirmReset` do NOT call `handleCloseDialog` immediately |
| L4: Drop `id` from `getCompanyPackageIdRepository` select | DONE | `permissionResolver.repository.ts:31`: `.select('packageId')` only (comment: "L4: select เฉพาะ packageId") |
| L1: logger.warn defer | Deferred to Phase 4 (documented in code comment at `permissionResolver.service.ts:85`) | Non-blocking |
| L3: EMPTY_COUNTS mutation cosmetic | Deferred | Non-blocking (L3 was categorized cosmetic/low) |

---

## Section 3 — SSD vs Code Consistency

| Spec Point | SSD/Requirement | Code | Status |
|-----------|-----------------|------|--------|
| API URL pattern (Q9=A) | `/api/v2/sale-dashboard/companies/:companyUuid/features` | `basePath = '/api/v2/sale-dashboard/companies'`; routes: `/:companyUuid/features` + `/:companyUuid/features/:featureUuid` | PASS |
| Middleware chain | `validateSaleDashboardAccessToken` → `requireSuperAdmin` → `validateRequest` | Routes file: all 3 routes apply middleware in this order | PASS |
| SSD §4.4 stale URL (`/api/v2/companies/...`) | Known stale — acknowledged in checklist P3.1 note | Code uses correct Q9=A path | NOTE (doc lag, not code bug) |
| CompanyFeatureItem shape (SSD §3.3) | `featureUuid, featureCode, name{th,en}, enabled, source (5-enum), overrideReason?` | `companyFeature.interface.ts:56–63` exactly matches | PASS |
| Response shape GET | `{list: CompanyFeatureItem[]}` | `controller.ts:37`: `res.success({ list }, ...)` | PASS |
| Response shape PUT/DELETE | single `CompanyFeatureItem` | `controller.ts:59,80`: `res.success(item, ...)` | PASS |
| Resolver cache param | `cache?: ResolverCache` explicit argument | All 4 public services in `permissionResolver.service.ts` accept optional `cache?` | PASS |
| SSD §7 error codes | `COMPANY_NOT_FOUND`, `FEATURE_NOT_FOUND`, `FEATURE_NOT_ACTIVE`, `CONCURRENT_OVERRIDE`, `SEED_PACKAGE_NOT_FOUND`, `PERMISSION_DATA_NOT_FOUND`, `PERMISSION_DATA_CORRUPTED` | All present in services (verified via grep and code read) | PASS |

---

## Section 4 — CR Fixes Verification

| CR Finding | Severity | Fix Status |
|-----------|---------|-----------|
| H1 Duplicate override query | High | FIXED — `getOverridesByCompanyRepository` removed; `getCompanyOverridesRepository` from resolver used in service |
| H2 console.* in saga | High | FIXED — no `console.*` in saga file; startup log removed |
| M1 Dead `23505` catch | Medium | FIXED — only `40001`/`40P01` caught; comment explains |
| M2 JSONB malformed crash risk | Medium | FIXED — `jsonb_typeof(f.menu_keys) = 'array'` guard in `getFeatureMenuKeysRepository` |
| M3 Duplicate Seed fallback | Medium | FIXED — `resolveEffectivePackageIdService` extracted and shared; `companyFeature.service` calls resolver helper |
| L1 logger.warn pattern | Low | DEFERRED to Phase 4 (documented; non-blocking for Phase 3 push) |
| L2 Dialog close before dispatch | Low | FIXED — `wasSubmittingRef` + useEffect auto-close pattern; dialog stays open during in-flight submit |
| L3 EMPTY_COUNTS cosmetic | Low | DEFERRED (cosmetic; no semantic issue) |
| L4 Over-fetch `id` in repo | Low | FIXED — `.select('packageId')` only |

All blocking fixes (H1, H2) confirmed addressed. Non-blocking deferred items documented.

---

## Section 5 — Test Coverage Assessment

| Test Area | Count | Coverage |
|-----------|-------|----------|
| `companyFeature.service.test.ts` | 26/26 PASS | 5-source attribution, Seed fallback, Q3=A (addon expiry at repo level), Q4=B (archived block), 409 concurrent, idempotent remove, reason normalization |
| `permissionResolver.service.test.ts` | 32/32 PASS | `resolveEffectivePackageId` (5), effective feature IDs (10 incl. union/delta/cache), menu keys (5), filter (8 incl. deep nesting, immutability), user permission (4) |
| Integration `companyFeature.integration.spec.ts` | 17/17 PASS | HTTP 200/400/401/403/404/409 scenarios; GET/PUT/DELETE; round-trip toggle; error pass-through |
| **Total** | **75/75 PASS** | |

Notable coverage: cache memoization (TC-9, TC-13 in resolver test), concurrent 409 (TC-17/18), JSONB malformed guard (TC-13 resolver), addon expiry semantics (TC-4 companyFeature).

Not covered by automated tests (by design for Phase 3):
- F3.11 Manual UAT (deferred post-push)
- Phase 4 integration with `/api/external/auth/v1/permissions` (out of Phase 3 scope)

---

## Section 6 — Frontend Behavioral Spec (frontend.md §1.4)

| Spec Point | Expected | Code | Status |
|-----------|---------|------|--------|
| Tab "Features" in client detail | Tab added to existing client detail | `list-view.tsx:357,365–367` | PASS |
| Load on mount / cleanup on unmount | `getCompanyFeatures(companyUuid)` + `clearList()` on unmount | `company-features-tab-view.tsx:64–72` | PASS |
| 5 components | tab-view + list + row + toggle-confirm-dialog + source-filter | All 5 files present in `/feature-tab/` | PASS |
| FeatureSourceBadge reuse | Phase 2 `<FeatureSourceBadge>` in row | Component referenced in row (checklist F3.7 confirms) | PASS |
| 6-option source filter | All + 5 sources | `company-feature-source-filter.tsx` (checklist F3.9 confirmed MUI Select 6 options) | PASS |
| Single-row update (no list refetch after toggle) | `replaceRowByFeatureUuid` in reducer | `reducers/sale-dashboard-company-features.ts` — reducer `replaceRowByFeatureUuid` immutable map | PASS |
| Dialog stays open during submit (L2 fix) | `isSubmitting=true` visible until saga completes | `company-features-tab-view.tsx:61–140` `wasSubmittingRef` + useEffect | PASS |
| i18n parity EN+TH | 37 `companyFeatures.*` keys + `client_management.features` | Both `en.json:1092` and `th.json:1091` contain `companyFeatures` block; `client_management.features` present in both at line ~440 | PASS |
| No `<Can>` wrapper on Features tab | Backend `requireSuperAdmin` enforces; frontend sale-dashboard context super_admin-only | `list-view.tsx:365–367` no `<Can>` wrapper | PASS (per spec) |

---

## Section 7 — DEF-001 Closure

**DEF-001 finding:** QA found that `errorMethods.errorXxx()` errors (CustomError with `.status` field) might not be recognized by `isHttpException` type guard → HTTP 500 instead of intended status code.

**BA Verification:**

Reading `src/utils/errorMethods.ts:6–10`:
```typescript
interface CustomError extends Error {
  status: number;    // <-- .status field present
  code?: string | number;
  data?: Record<string, unknown>;
}
```

Reading `src/middlewares/errorHandlers.middleware.ts:49–51`:
```typescript
const isHttpException = (error: unknown): error is HttpException => {
  return (error as HttpException).status !== undefined;  // checks .status field
};
```

The type guard checks `(error as HttpException).status !== undefined`. `CustomError` from `errorMethods.ts` has `.status: number` (always set, not undefined). Therefore `isHttpException(CustomError)` returns `true` and the handler at line 84–91 correctly uses `err.status` as the HTTP status code.

**Verdict: DEF-001 confirmed false positive.** `errorMethods.errorBadRequest(...)` (status=400), `errorNotFound(...)` (status=404), `errorConflict(...)` (status=409) all pass `isHttpException` and produce correct HTTP status codes.

Note: Integration tests throw `AppError` from mocked services — this was a test implementation choice, not a reflection of a real issue in production code. The actual services throw `CustomError` via `errorMethods` which works correctly.

---

## Section 8 — Outstanding / Blockers

### Blockers (must fix before push)
None.

### Non-critical Notes (post-push)

| # | Severity | Description | Suggested Action |
|---|---------|------------|-----------------|
| N1 | Low | L1 deferred: `logger.error(..., { level: 'warn', ... })` pattern in `permissionResolver.service.ts:85` and `companyFeature.service.ts:187` — emits ERROR level for a normal operational state (NULL package_id = Seed company) | Add `logger.warn()` to diagnostic logger before Phase 4 deploy (path will be hit on every permission request for Seed companies) |
| N2 | Low | L3 deferred: `EMPTY_COUNTS` mutation pattern in `company-features-tab-view.tsx:97–106` — shallow copy + property assignment on `next` | Convert to `reduce` pattern post-push (no functional impact) |
| N3 | Non-blocking | F3.11 Manual UAT pending | Complete after push; required before Phase 4 cutover |
| N4 | Doc | SSD §4.4 and backend.md §1.4 still list stale URL `/api/v2/companies/...` and `v2/admin/...` respectively | Lead to batch-update documentation in parallel with Phase 4 work |

---

## Section 9 — Recommendation (Push Readiness)

**Phase 3 is READY TO PUSH.**

All Phase 3 functional requirements (FR-4, FR-5) are fully implemented. All business rules (BR-7 delta logic, BR-11 super_admin access control) are enforced at correct layers. All CR high/medium fixes are verified in code. 75/75 tests pass. DEF-001 is false positive.

The two deferred items (L1 logger severity, L3 cosmetic) have no impact on correctness or security. F3.11 manual UAT is a post-push activity. SSD/backend.md doc stale references are known and tracked for Phase 4 batch update.

**Prerequisite before Phase 4 cutover:** Address N1 (logger.warn) and complete F3.11 UAT.

