---
type: code-review
phase: 3
reviewer: code-reviewer-agent
date: 2026-05-06
---

# Code Review Report — feature-management Phase 3

**Date:** 2026-05-06
**Reviewer:** code-reviewer agent
**Scope:** 4 waves (W1 permissionResolver + W2 companyFeature + W3 redux + W4 page D)
**Status:** PASS-with-fixes

---

## Summary

- **Verdict:** PASS-with-fixes
- **Findings count:** critical=0 / high=2 / medium=3 / low=4
- **Security:** Authorization chain complete (all 3 routes: GET/PUT/DELETE have `requireSuperAdmin`). No IDOR, no injection risk.
- **Ready for QA?** BLOCKED — fix H1 + H2 before push.

---

## Findings

### H1. (high) Duplicate override-query logic — DRY violation with divergence risk

**Files:**
- `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.repository.ts:83`
- `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts:121`

**Issue:** `getOverridesByCompanyRepository` (companyFeature repo) and `getCompanyOverridesRepository` (resolver repo) are functionally identical — both do the same `INNER JOIN comp_features` + double `status_type='active'` filter + same column aliases. The `companyFeature.service` correctly avoids duplicating resolver logic for package/addon fetches (imports from permissionResolver.repository directly), but the override query is duplicated. If one copy adds a new filter (e.g., `archived_at IS NULL` in a future migration) the other will silently diverge, producing inconsistent results between the GET list view and the effective permission calculation.

**Suggestion:** Remove `getOverridesByCompanyRepository` from `companyFeature.repository.ts`. Have `buildCompanyFeatureList` import and call `getCompanyOverridesRepository` from `permissionResolver.repository` instead. The return type `OverrideRowDb` and `CompanyOverrideRow` are structurally identical — unify into one shared type in `permissionResolver.interface.ts`, or re-export from there.

---

### H2. (high) `console.log` / `console.error` in saga production code

**File:** `happywork-sale-cms/src/store/sagas/sale-dashboard-company-features.ts:37,60,81,95`

**Issue:** All three worker sagas use `console.error(...)` for error logging, and `watchSaleDashboardCompanyFeatures` uses `console.log("[sagas] watchSaleDashboardCompanyFeatures: ✅")`. The global project rules (CLAUDE.md §6) explicitly forbid `console.*` output; errors must go through the structured logger. In a Next.js client bundle, `console.error` is visible in browser DevTools and leaks internal error stack traces to end users. The startup `console.log` also persists in every production page load.

**Suggestion:**
- Replace `console.error(...)` with the project logger (`logger.error(...)`) where available for client-side, or omit if no client logger is configured — at minimum remove the raw `console.error` and dispatch a failure action (which is already done — the log is redundant).
- Remove `console.log("[sagas] watchSaleDashboardCompanyFeatures: ✅")` — startup confirmation via console is a debugging artifact.

---

### M1. (medium) 23505 (unique_violation) catch is dead code for UPSERT path

**File:** `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts:154`

**Issue:** `upsertOverrideRepository` uses `ON CONFLICT (company_id, feature_id) DO UPDATE`, meaning PostgreSQL will never throw `23505` for this specific conflict — the UPSERT resolves it atomically. Catching `23505` in the caller is dead code and misleads future maintainers into thinking a unique-violation escape is possible here. The genuine race conditions are `40P01` (deadlock) and `40001` (serialization failure, e.g. if `REPEATABLE READ` is used upstream).

**Suggestion:** Remove `'23505'` from the error code check. Keep `40001` and `40P01`. Add a comment explaining why `23505` cannot occur with this UPSERT pattern.

---

### M2. (medium) `getFeatureMenuKeysRepository` — LATERAL JOIN fails silently on malformed `menu_keys` JSONB

**File:** `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts:153`

**Issue:** The LATERAL `jsonb_array_elements_text(f.menu_keys)` call will throw a PostgreSQL runtime error (`ERROR: cannot call jsonb_array_elements_text on a non-array`) if any `comp_features.menu_keys` row contains a JSONB object or scalar instead of an array. Although write-path validates `menu_keys` as a string array (Zod schema), there is no defense at the read path. This would surface as an unhandled 500 in the resolver, cascading to all users of that company.

**Suggestion:** Add a DB-level guard: `WHERE jsonb_typeof(f.menu_keys) = 'array'` in the query. This skips malformed rows rather than crashing the resolver. Combine with an alerting/monitoring check for rows where `jsonb_typeof(menu_keys) != 'array'`.

---

### M3. (medium) `resolveEffectivePackageId` in `companyFeature.service` duplicates fallback logic from resolver service

**File:** `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts:43-59`

**Issue:** `resolveEffectivePackageId` in `companyFeature.service` re-implements the exact Seed-fallback logic (null-check → getSeed → errorBadRequest) that `resolveCompanyEffectiveFeatureIdsService` in `permissionResolver.service` also implements (lines 65-80). If the fallback behavior changes (e.g., error code changes from `SEED_PACKAGE_NOT_FOUND` to a different code, or the logger call changes), two places must be updated. The companyFeature service already imports several resolver repo functions — it could instead call the resolver service directly and pass a local `ResolverCache`.

**Suggestion:** In `buildCompanyFeatureList`, replace the direct `getCompanyPackageIdRepository` + `getSeedPackageIdRepository` calls with a call to `resolveCompanyEffectiveFeatureIdsService(companyId, cache)` to get the feature ID set directly. The adapter `toCompanyFeatureItems` only needs the sets — it doesn't need the intermediate packageId. This eliminates both `resolveEffectivePackageId` and the duplicated fallback, and reuses the cache if one is passed.

---

### L1. (low) `logger.error` with `level: 'warn'` pattern — misleading log severity

**Files:**
- `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts:79`
- `happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts:58`

**Issue:** Both files call `logger.error('...', { level: 'warn', ... })` — a workaround because the diagnostic logger doesn't expose a `warn` method. The log level emitted is `error` regardless of the `level` field in the context object, which means "company has NULL package_id (normal operational state)" will appear as an ERROR in monitoring dashboards, potentially triggering false alerts.

**Suggestion:** Add a `logger.warn()` method to the diagnostic logger, or map to an existing lower-severity log method. The comment acknowledges this is a known limitation — it should be resolved before Phase 4 where this path will be hit on every permission request for companies on the Seed package.

---

### L2. (low) `handleConfirmSet` / `handleConfirmReset` in `CompanyFeaturesTabView` close dialog before dispatch completes

**File:** `happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx:120-146`

**Issue:** Both handlers call `handleCloseDialog()` immediately after dispatching the saga action (before the saga completes). The `isSubmitting` prop in the dialog is computed as `togglingFeatureUuid !== null`, which is set when the REQUEST action fires. However, because `handleCloseDialog` is called in the same synchronous tick as the dispatch, the dialog is already closed before `togglingFeatureUuid` is set in Redux state. This means the `isSubmitting=true` state is never visible to the user — the dialog closes, then the row shows a loading indicator.

This is **functionally correct** (no data lost, no crash) but the UX is inconsistent with the spec which implies the dialog's buttons should be disabled while submitting. The spec calls for the "Confirm" button to show a loading/disabled state.

**Suggestion:** Either (a) close the dialog on the SUCCESS action (in the saga, emit a close-dialog action after success), or (b) keep dialog open while `togglingFeatureUuid === targetItem?.featureUuid` and close it when it clears to `null`. Option (b) is simpler with the current architecture.

---

### L3. (low) `EMPTY_COUNTS` object is mutated inside `useMemo` via spread + property assignment

**File:** `happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx:91-99`

**Issue:** The `counts` useMemo spreads `EMPTY_COUNTS` into `next` and then assigns `next[row.source]`. The spread creates a shallow copy so `EMPTY_COUNTS` itself is not mutated — this is correct. However, the type of `next` is `CompanyFeatureSourceCounts` which is a `Record<...>` — not `readonly`. The mutation of `next` is safe here but the pattern is slightly at odds with the immutability principle in the codebase rules.

**Suggestion:** Minor: convert to a `reduce` pattern or use `Object.fromEntries` to build the counts immutably. Not a blocker.

---

### L4. (low) `getCompanyPackageIdRepository` selects `id` unnecessarily

**File:** `happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts:30`

**Issue:** The query `.select('id', 'packageId')` fetches the `id` column but the function only uses `row.packageId`. The `id` is never returned or used by any caller.

**Suggestion:** Change to `.select('packageId')` to avoid over-fetching. Minor.

---

## Section: Resolver Correctness

| Check | Result |
|-------|--------|
| **Q3=A (addon expiry):** `expires_at IS NULL OR expires_at > NOW()` | PASS — `permissionResolver.repository.ts:82-84` correctly uses `whereNull('expires_at').orWhere('expires_at', '>', knex.fn.now())` |
| **Q4=B (archived feature filter):** resolver JOINs `comp_features` and filters `f.status_type = 'active'` at DB level | PASS — both `getCompanyOverridesRepository` (line 126-127) and `getAddonFeatureIdsRepository` (line 104) apply this filter |
| **Effective set logic:** `(packageFeatures ∪ addonFeatures) − overrides[disabled] + overrides[enabled]` | PASS — `permissionResolver.service.ts:85-108` performs union first, then override delta in correct order |
| **Seed fallback:** company.package_id NULL → use Seed → log warn → throw if Seed not found | PASS — both resolver service (line 71-80) and companyFeature service (line 50-59) implement this; see M3 re: duplication |

---

## Section: Performance / N+1

| Query | Assessment |
|-------|-----------|
| `getPackageFeatureIdsRepository` | PASS — single query, 1:few JOIN |
| `getActiveCompanyAddonIdsRepository` | PASS — single query with runtime expiry filter |
| `getAddonFeatureIdsRepository` | PASS — batch WHERE IN + DISTINCT |
| `getCompanyOverridesRepository` | PASS — single query JOIN |
| `getFeatureMenuKeysRepository` | PASS — batch WHERE IN + LATERAL; see M2 re: malformed JSONB risk |
| `buildCompanyFeatureList` Promise.all | PASS — 4 independent queries parallelized correctly; addon→feature query waits for addon IDs (correct sequential dependency) |
| `getCompanyFeatureItemAfterMutation` | ACCEPTABLE — full list refetch (~14 rows, 4 batch queries) after mutation. Small dataset. No N+1. |
| `resolveUserEffectivePermissionService` (Phase 4) | PASS — sequential lookup chain is correct (user→employee→mapping→permission are dependent); `Promise.all` used for independent package+addon batch |

---

## Section: Concurrent Handling

- UPSERT with `ON CONFLICT` is atomic at DB level — concurrent toggles on the same `(company_id, feature_id)` pair serialize correctly.
- Error code catch: `40P01` (deadlock) and `40001` (serialization) → 409 `CONCURRENT_OVERRIDE`. **See M1** — `23505` catch is dead code for UPSERT.
- `removeOverrideRepository` uses a simple DELETE — no concurrent conflict handling needed (DELETE is idempotent).
- `takeEvery` in saga is correct for toggle actions — allows concurrent toggles on different rows.

---

## Section: Frontend

| Check | Result |
|-------|--------|
| **Toggle dialog logic:** `isOverrideSource` uses `source === "override-enabled" \|\| source === "override-disabled"` → 3-button mode | PASS — correct for all 5 source values |
| **nextEnabled:** `!item.enabled` | PASS — correct flip |
| **3-button mode behavior:** Reset-to-Default calls DELETE (removeOverride), Confirm calls PUT (setOverride) | PASS |
| **Reducer single-row update:** `replaceRowByFeatureUuid` using `list.map(...)` | PASS — immutable, returns unchanged list if UUID not found |
| **`useCallback` wrappers:** all 5 dispatchers wrapped | PASS |
| **togglingFeatureUuid clear on success/failure:** both reducer cases set `null` | PASS |
| **Source filter `useMemo` deps:** `[list]` only | PASS — `counts` depends only on `list` |
| **i18n parity:** 37 `companyFeatures.*` keys + `client_management.features` | PASS — 100% parity EN=TH |
| **Dialog close before submission completes** | See L2 — functionally correct, UX gap |

---

## Section: Security / Authorization

| Check | Result |
|-------|--------|
| GET `/:companyUuid/features` has `requireSuperAdmin` | PASS — line 58 in routes |
| PUT `/:companyUuid/features/:featureUuid` has `requireSuperAdmin` | PASS — line 25 in routes |
| DELETE `/:companyUuid/features/:featureUuid` has `requireSuperAdmin` | PASS — line 41 in routes |
| IDOR risk: `companyUuid` validated as UUID format (Zod) | PASS — Zod schema enforces `.uuid()` for both params |
| Actor UUID resolved from `req.user` (set by middleware) | PASS — `resolveActorUuid` throws 401 if missing |
| Frontend tab visible to non-super_admin | LOW RISK — sale dashboard is super_admin-only context per design; backend gating is sufficient |

---

## Section: Filter JSONB (`filterPermissionByMenuKeysService`)

| Check | Result |
|-------|--------|
| **Path building:** `prefix ? \`${prefix}.${key}\` : key` | PASS — correctly handles top-level (prefix='') and nested (prefix='payroll') |
| **Leaf detection:** `hasActionKey` checks for `read/create/update/delete/export` in node keys | PASS — matches PermissionDefault action key convention |
| **Malformed JSONB catch:** outer try-catch in `resolveUserEffectivePermissionService` wraps `filterPermissionByMenuKeysService` call → throws `PERMISSION_DATA_CORRUPTED` | PASS — SSD §7 row 7 satisfied |
| **Pure function:** no side effects, returns new object | PASS — no mutation of input |
| **Empty result:** returns `{}` (not null/undefined) | PASS — safe for caller |
| **metadata keys (name/sequence):** preserved only if node has allowed children | PASS — dropped with branch when `hasAllowedChild = false` |

---

## Recommendations (post-fix actions before push)

1. **Fix H1:** Consolidate duplicate override query — remove `getOverridesByCompanyRepository` from `companyFeature.repository.ts`, reuse `getCompanyOverridesRepository` from resolver repo. Unify `OverrideRowDb` / `CompanyOverrideRow` types.

2. **Fix H2:** Remove `console.log`/`console.error` from saga file. Remove startup log. For error cases, the `yield put(Actions.*Failure(...))` dispatch is sufficient — no additional logging needed in the saga worker.

3. **Fix M1:** Remove `'23505'` from DB error code catch in `setCompanyFeatureOverrideService`. Document why it cannot occur.

4. **Fix M2:** Add `WHERE jsonb_typeof(f.menu_keys) = 'array'` guard in `getFeatureMenuKeysRepository` to prevent runtime crash on malformed JSONB.

5. **Fix M3 (recommended):** Replace `resolveEffectivePackageId` private helper + direct repo calls in `buildCompanyFeatureList` with a call to `resolveCompanyEffectiveFeatureIdsService` from resolver service, passing a local cache. This eliminates duplicate Seed-fallback logic.

6. **Fix L1 (before Phase 4):** Add `logger.warn()` to diagnostic logger. The NULL-package-id path will be hit frequently in Phase 4.

7. **L2 (consider for UX):** Keep dialog open while toggle is in-flight. Close on SUCCESS/FAILURE action in saga or watch `togglingFeatureUuid` clearing.

---

**Final verdict: BLOCKED — fix H1 + H2 first, then READY for QA.**
