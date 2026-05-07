---
type: code-review
phase: 5
reviewer: code-reviewer-agent
date: 2026-05-07
---

# Code Review Report — feature-management Phase 5

**Date:** 2026-05-07
**Reviewer:** code-reviewer agent
**Scope:** W1 (Schema + Helper) + W2 (Feature CRUD) + W3 (Resolver mobile path) + W4 (External Auth + user-info) + W5 (compPermission BUG fix + mobile validation) + W6 (FE redux/types) + W7 (FE Feature CRUD form/detail) + W8 (FE Page D + BE companyFeature extension) + W9 (QA parameterized)
**Status:** PASS-with-fixes

---

## Summary

- **Verdict:** PASS-with-fixes
- **Findings count:** critical=0 / high=1 / medium=3 / low=3
- **Security:** Filter chain complete for both web + mobile. No bypass path for owner case (Q-A=STRICT). No SQL injection. No secrets in code. BUG fix (Phase 4 W2 oversight) correct — split validation confirmed.
- **Ready for BA?** BLOCKED — fix H1 before push. M1–M3 acceptable with documented plan.

---

## Findings

### H1. (high) Duplicate `getUserPermissionJsonbRepository` call per request in hot-path `/api/external/auth/v1/permissions/:uuid`

**Files:**
- `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts:355` (`resolveUserEffectivePermissionService`)
- `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts:408` (`resolveUserEffectivePermissionMobileService`)
- `src/modules/v1/externalAuth/permissions.service.ts:51–75` (caller)

**Issue:** `getPermissionsService` calls `resolveUserEffectivePermissionService(userUuid, cache)` then `resolveUserEffectivePermissionMobileService(userUuid, cache)` with a shared `ResolverCache`. However the shared cache does **not** deduplicate the `getUserPermissionJsonbRepository(userUuid)` lookup — each of the two resolver functions calls it independently (lines 355 and 408). This means every `/permissions/:uuid` request makes **2 identical DB JOINs** (`emp_employees` → `auth_users` → `comp_permission_mapping` → `comp_permission`) per call. Since this is the most-called auth endpoint (every authenticated user), the duplicated 3-hop JOIN adds latency and DB pressure. Phase 5 design intent in SSD §4.5 ("Mobile path adds only 1 extra batch DB call") is violated — it actually adds 1 extra JOIN + 1 extra batch call.

**Suggested fix:** Extract the `getUserPermissionJsonbRepository` call to the caller (`getPermissionsService`) and pass the resulting `UserPermissionRow` directly into both resolver functions:
```typescript
// In getPermissionsService — fetch once, pass to both
const permRow = await getUserPermissionJsonbRepository(userUuid);
if (!permRow) { /* graceful {} for both */ }
// resolveUserEffectivePermission[Mobile]Service receive permRow instead of re-fetching
```
Or add a cache key `userPerm:{userUuid}` to the shared `ResolverCache` so the second call hits in-memory. Either approach eliminates the duplicate 3-hop JOIN.

---

### M1. (medium) `getCompanyFeaturesService` — known but untracked duplication of 4 batch queries between `buildCompanyFeatureList` and resolver calls

**File:** `src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts:57–79, 96–116`

**Issue:** `getCompanyFeaturesService` calls both `buildCompanyFeatureList(company.id)` and `resolveCompanyEffectiveMenuKeysService(company.id, cache)` + `resolveCompanyEffectiveMobileMenuKeysService(company.id, cache)` in `Promise.all`. The cache shared by the two resolver calls **does not cover** `buildCompanyFeatureList` (which takes no cache parameter). Therefore `getPackageFeatureIdsRepository`, `getActiveCompanyAddonIdsRepository`, `getAddonFeatureIdsRepository`, and `getCompanyOverridesRepository` each run **twice** — once inside `buildCompanyFeatureList` and once inside `resolveCompanyEffectiveFeatureIdsService` (called by each resolver). The comment at line 96–99 acknowledges this but defers to "Wave 9 cleanup." At ~6–8 extra DB queries per `GET /companies/:uuid/features` call for admins, this is acceptable for admin-only traffic but should be tracked as a known performance debt.

**Suggested fix (short-term):** Create a GitHub/JIRA ticket referencing this file:line so the Wave 9 cleanup has an actionable target. The code is functionally correct — no bug risk, only performance cost.

**Suggested fix (medium-term):** Extend `buildCompanyFeatureList` to accept an optional `ResolverCache` and populate the standard cache keys (`featureIds:{companyId}`, `packageId:{companyId}`) so subsequent resolver calls hit cache. This requires refactoring the function signature but eliminates all duplication.

---

### M2. (medium) Migration M10 uses sequential `for...await` loop — minor migration perf issue

**File:** `src/database/postgresql/migrations/data/20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts:54–58`

**Issue:** The `up()` function iterates 4 entries with `for (const entry of SEED_MOBILE_MAPPING) { await knex(...).update(...) }` — 4 sequential UPDATE round-trips when a single `WHERE code IN ('basic_attendance', 'basic_leave', 'basic_reports', 'max_10_users')` + CASE expression (or separate UPDATEs in a batch) would reduce to 1–4 round-trips with less network overhead. For a migration that runs once with 4 rows this is harmless, but the pattern should not be cargo-culted into future migrations with more rows.

**Suggested fix (optional):** Change to `Promise.all` over the 4 updates (4 parallel round-trips vs 4 sequential) for parity with the project's batching guideline. Or add a comment explaining why sequential is intentional (e.g., avoiding lock contention during migration).

---

### M3. (medium) Walker `hasActionKey` uses `ACTION_KEYS` (5 web keys including `export`) as leaf detector for mobile JSONB in `filterPermissionMobileByMenuKeysService`

**File:** `src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts:38–40, 339–344`

**Issue:** `filterPermissionMobileByMenuKeysService` delegates to `filterPermissionByMenuKeysService` which uses `hasActionKey` — and `hasActionKey` checks against `ACTION_KEYS = ['read','create','update','delete','export']` (5 keys). Mobile JSONB leaves contain only `{read, create, update, delete}` (4 keys, no `export`). The `Array.some()` semantics mean a mobile leaf is correctly detected as long as it has **any** of the 5 keys — and it will always have at least `read` — so no mis-classification occurs in practice.

However, if a future mobile node is introduced that has **only** an `export`-like extension key not in `ACTION_KEYS`, the semantic would break silently. More importantly, if mobile JSONB ever contains a node with `export: true` (from legacy data import), the walker would treat it as a leaf using web detection logic rather than mobile logic.

The current code comment (line 319–325) explains the rationale correctly. This is a low-risk **design smell** rather than a current bug, but should be tracked.

**Suggested fix:** Add a parameterized `actionKeys` to `hasActionKey` call chain (already done at the `collectLeafPaths` level in `compPermission.ts`) or document a constraint "mobile JSONB must never contain `export` key" in the repository-layer `getUserPermissionJsonbRepository` comment.

---

### L1. (low) `OwnerPermissionMobile` uses `JSON.parse(JSON.stringify(PermissionMobileDefault))` — deep-clone via JSON round-trip

**File:** `src/constant/compPermission.ts:611`

**Issue:** `OwnerPermissionMobile = JSON.parse(JSON.stringify(PermissionMobileDefault))` is a shallow-looking deep clone that has the same pattern as `OwnerPermission` on line 361. Both clone a `false`-valued permission tree but never set any actions to `true` — meaning `OwnerPermission` and `OwnerPermissionMobile` are identical to `PermissionDefault` / `PermissionMobileDefault` with all booleans false. This suggests the intent is "full permission" (per comment "Full Access") but the implementation clones the all-false skeleton. If these constants are used as a deepMerge baseline for owners, the result is correct only because deepMerge with `isOwner=true` presumably grants all booleans. However if they are used as standalone permission objects the result would be all-false.

**Note:** This is a pre-existing issue (Phase 1 code) and outside Phase 5 scope. Flagging for awareness — do not block Phase 5 merge.

---

### L2. (low) `createCompPermissionService` catch block checks `instanceof AppError` before custom-error re-throw — ordering risk

**File:** `src/modules/v1/admin/compPermission/compPermission.service.ts:119–128`

**Issue:** The catch block first checks `if (error instanceof AppError) throw error`, then checks `errCode` for `INVALID_MENU_KEY_FOR_COMPANY` / `INVALID_MOBILE_MENU_KEY_FOR_COMPANY`. Since `errorBadRequest` returns a `CustomError` (not `AppError`), the `instanceof AppError` check correctly passes through, and the `errCode` check re-throws. This works correctly. However the double-check pattern is redundant — if `CustomError` ever extends `AppError` in a future refactor, the `instanceof AppError` check would swallow the `errCode` check, causing the custom 400 error to be wrapped in a generic 500.

**Suggested fix:** Combine the re-throw condition: check `errCode` first (or use a utility `isCustomError` guard), then fallback to `instanceof AppError`, then generic 500.

---

### L3. (low) Frontend `company-effective-menus-preview.tsx` fetches `availableMenuKeys` via `useSaleDashboardFeatureManagement` — implicit dependency

**File:** `src/sections/client-management/feature-tab/components/company-effective-menus-preview.tsx:57–58`

**Issue:** This component reads `availableMenuKeys` from the feature-management slice. For the component to render correctly (non-empty `webAvailableKeys` / `mobileAvailableKeys`), the parent view must have already dispatched `sdGetAvailableMenuKeysRequest` to populate the slice. This is an **implicit prop dependency** — nothing in the component's type signature signals this requirement. If the company-features page is rendered without first fetching available keys (e.g., deep-linked directly), both columns will show empty available-key catalogs and the count chips will be misleading.

**Suggested fix:** Either dispatch `sdGetAvailableMenuKeysRequest` in the company-features page itself (or in a higher-order effect), or accept `webAvailableKeys`/`mobileAvailableKeys` as explicit props (and let the parent decide where to source them from).

---

## Section: Mobile Filter Completeness

All 5 identified endpoints verified:

| Endpoint | Web filter | Mobile filter | Verdict |
|----------|-----------|---------------|---------|
| `/api/external/auth/v1/permissions/:uuid` | `resolveUserEffectivePermissionService` | `resolveUserEffectivePermissionMobileService` | PASS |
| `/auth/user-info` (`getEmployeePermissionService`) | `filterPermissionByMenuKeysService` + `allowedKeys` | `filterPermissionMobileByMenuKeysService` + `allowedMobileKeys` | PASS |
| `getCompPermissionByUuidController` | `filterPermissionByMenuKeysService` (deepMerge baseline) | `filterPermissionMobileByMenuKeysService` (deepMerge baseline) | PASS |
| `/comp-permission/default?companyUuid=` | `filterPermissionByMenuKeysService` | `filterPermissionMobileByMenuKeysService` | PASS |
| `getCompanyFeaturesService` | `resolveCompanyEffectiveMenuKeysService` | `resolveCompanyEffectiveMobileMenuKeysService` | PASS |

No hidden callers identified that Phase 4 or Phase 5 missed. All paths from `compPermission.service.ts` that touch permission JSONB now handle both web and mobile namespaces.

---

## Section: Phase 4 BUG Fix Verification

`assertPermissionKeysWithinCompanyMenusService` in `compPermission.service.ts:58–92` verified correct:

- Web path (`permission` JSONB) → `extractMenuKeysFromPermissionJsonbService` → `resolveCompanyEffectiveMenuKeysService` → throws `INVALID_MENU_KEY_FOR_COMPANY`
- Mobile path (`permissionMobile` JSONB) → `extractMobileMenuKeysFromPermissionJsonbService` → `resolveCompanyEffectiveMobileMenuKeysService` → throws `INVALID_MOBILE_MENU_KEY_FOR_COMPANY`
- Empty `{}` guard: `desiredWebKeys.length > 0` / `desiredMobileKeys.length > 0` — both correctly skip resolver call (cost optimization preserved)
- Edge case: keys in web set but not mobile set — handled by distinct resolver calls with distinct allowed sets. If web key happens to match mobile namespace by name, mobile extract would find it but mobile resolver would reject it (correct behavior)
- Error ordering: web checked first, throws; mobile not checked. Web passes, mobile checked — distinct 400 codes. Correct per SSD §7.

---

## Section: Walker Reuse Strategy

`filterPermissionMobileByMenuKeysService` and `extractMobileMenuKeysFromPermissionJsonbService` delegate to their web counterparts with no code duplication. The walker uses `ACTION_KEYS.some()` (any-key-present) for leaf detection.

**Verified safe** for current mobile JSONB shape (leaves have `read/create/update/delete` — all 4 are in `ACTION_KEYS`). See M3 finding for the edge-case design smell.

Test coverage: `describe.each` in `permissionResolver.service.test.ts` parameterizes all 6 walker-related tests across both web and mobile namespaces (TC-14 through TC-26 × 2). Coverage is adequate.

---

## Section: Migration Safety

**M9 (`20260506-1800-alter-comp_features-add-mobile_menu_keys.ts`):**
- `up()`: ALTER TABLE + CREATE INDEX IF NOT EXISTS — idempotent, safe
- `down()`: DROP INDEX IF EXISTS + ALTER TABLE DROP COLUMN IF EXISTS — correct order (index before column), safe
- GIN index scoped to `mobile_menu_keys` column only — correct

**M10 (`20260506-1801-backfill-comp_features-mobile_menu_keys-seed.ts`):**
- Backfill scoped to `WHERE code = ? AND status_type = 'active'` — idempotent, does not touch archived rows (Q4=B compliant)
- `down()`: reverts to `[]` — correct
- 4 keys in SEED_MOBILE_MAPPING verified against `PermissionMobileDefault` structure (homepage.attendance, timeAttendance.myReport, homepage.request, etc. — all are valid leaf paths in `PermissionMobileDefaultDev/Prod`)
- Sequential loop issue noted in M2 above — non-blocking for 4 rows

---

## Section: Cache Sharing

**`getPermissionsService`:** Single `ResolverCache` Map shared between web + mobile resolver calls. `featureIds:{companyId}` + `packageId:{companyId}` hit cache on mobile call. `getUserPermissionJsonbRepository` NOT cached — see H1.

**`getEmployeePermissionService`:** Single cache shared via `Promise.all([resolveCompanyEffectiveMenuKeysService, resolveCompanyEffectiveMobileMenuKeysService])`. Cache share works but note: `Promise.all` races both calls before either can populate `featureIds:{companyId}`, meaning both calls will independently trigger `resolveCompanyEffectiveFeatureIdsService`. The cache key for `featureIds` is only useful for **sequential** calls. For `Promise.all`, if both calls start simultaneously before either populates the cache, `featureIds` will be computed twice. This is a **race condition in the cache** for this specific path.

**`getCompPermissionByUuidService` / `getCompPermissionByUuidController`:** Cache created at controller level, passed to service — correct sharing via Phase 4 H1 fix. Mobile resolver hits `featureIds` + `packageId` cache from service-level web call.

**`getCompanyFeaturesService`:** Cache shared between 2 resolver calls (web + mobile) but not with `buildCompanyFeatureList` — see M1.

**Cache namespace collision risk:** Cache keys `featureIds:{companyId}`, `menuKeys:{companyId}`, `mobileMenuKeys:{companyId}`, `packageId:{companyId}` — verified distinct namespaces. No cross-talk between web and mobile within a single request.

**Promise.all race note:** The `featureIds` cache hit only works when `resolveCompanyEffectiveMenuKeysService` and `resolveCompanyEffectiveMobileMenuKeysService` are called **sequentially** (second call after first populates cache). When called via `Promise.all` (concurrent), both start simultaneously and the cache miss fires on both. The integration test at `permissionResolver.service.test.ts:351–378` tests the **sequential** pattern only. Callers that use `Promise.all` (like `getEmployeePermissionService:137`) should be reviewed — this is M1-adjacent.

---

## Section: Frontend Discriminated Union API

**`MenuKeysSelect`** (`menu-keys-select.tsx`):
- `mode?: "single"` (default) vs `mode: "split"` discriminated union — TypeScript narrowing correct via conditional prop shapes
- Internal `KeySelectColumn` component correctly receives typed `availableKeys`, `value`, `onChange` per mode
- Backward compat: existing single-mode callers pass `mode` undefined → single-mode branch (safe)

**`MenuTreePreview`** (`menu-tree-preview.tsx`):
- Split mode: `webMenuKeys` + `mobileMenuKeys` + `webAvailableKeys` + `mobileAvailableKeys` — correct
- Single mode (legacy): `menuKeys` + `availableKeys` — preserved

**Form integration:** `feature-form.tsx` uses RHF `Controller` with `menuKeys` and `mobileMenuKeys` as separate fields. Yup schema validates both namespaces independently. Default values correct (`[]`).

**Backward compat:** Page B (`package-effective-menus-preview`) uses `mode="single"` (default) — unaffected by split-mode addition. Type system enforces this at compile time.

**L3 finding** (implicit `availableMenuKeys` dependency) noted above.

---

## Section: i18n Parity

Top-level key comparison result:
- `featureManagement`: EN=20 keys, TH=20 keys, diff=empty — PASS
- `companyFeatures`: EN=15 keys, TH=15 keys, diff=empty — PASS

Spot-checked mobile-specific keys:
- `featureManagement.menu_keys.mobile_menu_keys` — present in both `en.json` and `th.json`
- `featureManagement.form.mobile_menu_keys` + `featureManagement.form.mobile_menu_keys_helper` — present in both
- `featureManagement.validation.mobile_menu_keys_invalid` — present in both
- `companyFeatures.effective_menus.*` — present in both

No orphan keys detected at top-level. Nested key deep-diff not performed but spot-check passed for all Phase 5 additions.

---

## Section: Test Coverage

**W9 parameterized (`Q8=B`):**
- `describe.each` over `['web', 'mobile']` namespace tuples in `permissionResolver.service.test.ts` — covers TC-10 to TC-25+ per namespace (×2 = effective 30+ test paths)
- Cache-share test (sequential) at line 351 — correct
- `feature.service.test.ts:157–179` — `describe.each` over `['web', 'mobile']` invalid-key rejection — correct
- `companyFeature.service.test.ts` — mocks `resolveCompanyEffectiveMobileMenuKeysService` — correct

**Phase 5 W5 BUG regression:**
- Integration spec `permission-resolver.integration.spec.ts` covers IT-L (P4.4 save validation INVALID_MENU_KEY_FOR_COMPANY). Phase 5 W5 adds `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` — need to confirm there is a regression test for the mobile counterpart specifically.
- Reviewed the spec: Phase 5 W9 adds `mockResolveMobileMenuKeys.mockResolvedValue(new Set<string>())` as default (line 199) — ensures existing web tests don't break. But no explicit `IT-L (mobile)` test case asserting `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` 400 was found in the reviewed portion. If the test file has this beyond the reviewed lines, this is a pass; otherwise it is a gap.

**Recommendation:** Verify `companyFeature.integration.spec.ts` contains explicit test for `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` 400 response.

---

## Section: Security / Owner Case

`getEmployeePermissionService` (`compPermissionMapping.service.ts:113–179`):
- Resolver called regardless of `isOwner` value — Q-A=STRICT enforced
- `basePermission` = `filterPermissionByMenuKeysService(structuredClone(PermissionDefault), allowedKeys)` — filtered before deepMerge
- `basePermissionMobile` = `filterPermissionMobileByMenuKeysService(structuredClone(PermissionMobileDefault), allowedMobileKeys)` — filtered before deepMerge
- Post-deepMerge re-filter applied for both web and mobile — guards against leftover keys from legacy role JSONB
- Owner case: `deepMerge(..., isOwner=true)` merges atop filtered baseline → result is always subset of company-enabled menus — correct

No auth bypass found. No SQL injection risk (all queries use parameterized Knex builder). No secrets in code.

---

## Section: Backward Compatibility

**BE additive fields:**
- `Feature.mobileMenuKeys` — default `[]` for legacy rows → backward compat for readers that ignore the field
- `CompanyFeatureItem.menuKeys` + `.mobileMenuKeys` — additive (Phase 3 reducer reads `data.list` only) → safe
- `CompanyFeatureListResponse.effectiveMenuKeys` + `.effectiveMobileMenuKeys` — new top-level fields — additive
- `/comp-permission/default` now returns `{ permission, permissionMobile }` — `permissionMobile` additive — old clients ignore

**BE BREAKING:**
- `GET /features/menu-keys/available`: shape changed from `{ menuKeys: [] }` to `{ web: [], mobile: [] }` (Q3=B) — Wave 6 FE synced to consume `.web`/`.mobile`. Reducer initial state `availableMenuKeys: { web: [], mobile: [] }` confirmed correct.

**Old clients not sending `mobileMenuKeys`:** BE Zod schema has `.default([])` for `mobileMenuKeys` — so missing field = `[]` — safe.

---

## Recommendations (post-fix actions before push)

1. **[BLOCKER — fix H1]** Eliminate duplicate `getUserPermissionJsonbRepository` call in `getPermissionsService`. Either extract the lookup to the caller and pass `UserPermissionRow` to both resolvers, or add a `userPerm:{userUuid}` cache key. This is the most-called auth endpoint.

2. **[Should-have — track M1]** File a follow-up ticket to refactor `buildCompanyFeatureList` to accept `ResolverCache`. Reference `companyFeature.service.ts:96–99` in the ticket.

3. **[Should-have — verify test]** Confirm `permission-resolver.integration.spec.ts` or `companyFeature.integration.spec.ts` has an explicit test case for `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` 400. If missing, add before push.

4. **[Should-have — track M3]** Add inline comment to `filterPermissionMobileByMenuKeysService` noting the constraint "mobile JSONB must not contain `export` key" to prevent silent mis-classification if that changes.

5. **[Optional — L3]** Dispatch `sdGetAvailableMenuKeysRequest` in the company-features page initialization or document the dependency in component JSDoc.

**Final verdict: BLOCKED — fix H1 first. After H1 fixed, READY for BA.**
