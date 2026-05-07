---
type: code-review
phase: 4
reviewer: code-reviewer-agent
date: 2026-05-06
---

# Code Review Report — feature-management Phase 4

**Date:** 2026-05-06
**Reviewer:** code-reviewer agent
**Scope:** W1 (permissions endpoint) + W2 (compPermission service filter) + W3.5 (4 audit fixes) + W4 (25 integration tests)
**Status:** PASS-with-fixes

---

## Summary

- **Verdict:** PASS-with-fixes
- **Findings count:** critical=0 / high=2 / medium=2 / low=3
- **Security:** Filter chain is complete. No bypass path found for owner case (Q-A=STRICT enforced). No injection risk. No secrets in code.
- **Ready for BA?** BLOCKED — fix H1 + H2 before push.

---

## Findings

### H1. (high) Double resolver invocation in `getCompPermissionByUuidController` — wasted DB round-trips per request

**Files:**
- `src/modules/v1/admin/compPermission/compPermission.service.ts:144-145` (Wave 2 service-level filter)
- `src/api/v1/admin/compPermission/compPermission.controller.ts:174-175` (W3.5 Fix 2 controller-level filter)

**Issue:** `getCompPermissionByUuidController` calls `getCompPermissionByUuidService`, which internally creates its own `ResolverCache` at line 144 and resolves `resolveCompanyEffectiveMenuKeysService(compPermission.companyId, cache)`. The controller then creates a **second, separate** `ResolverCache` at line 174 and calls `resolveCompanyEffectiveMenuKeysService(rest.companyId, cache)` again. Because the two caches are independent `new Map()` instances, the full resolver chain (DB lookup: package_id → package_features + addon_ids → addon_features + overrides → menu_keys) runs **twice per request** for the same `companyId`. Each resolver call is roughly 4–5 DB queries. On a heavy usage path (admin opens a role editor), this doubles the DB load.

**Root cause:** When W3.5 Fix 2 added the pre-merge baseline filter to the controller, it introduced a new resolver call without passing a shared cache down from the service, nor consuming the already-resolved `allowedKeys` returned by the service. The service's filter output is discarded by the `{ permission, permissionMobile, ...rest }` destructure, and the service does not expose its `allowedKeys`.

**Suggested fix:** Two options (either works):
1. Move the resolver call and both filter steps (service JSONB filter + controller baseline filter) entirely into the service. The service should return `{ ...record, permission, permissionMobile, allowedKeys }` and the controller uses `allowedKeys` it receives — zero extra DB calls.
2. Pass a `cache` parameter from controller into `getCompPermissionByUuidService` so both the service filter and the controller baseline filter share the same memoized result (add `cache?: ResolverCache` param; caller creates `new Map()` before service call and reuses it for baseline filter).

---

### H2. (high) `tests/unit/externalAuth/permissions.service.test.ts` — stale assertions will FAIL after Phase 4 W1

**File:** `tests/unit/externalAuth/permissions.service.test.ts:20,28`

**Issue (DEF-001):** Both passing test cases assert `expect(result).toEqual({ userUuid: '...', isAdmin: true/false })` — the old two-field shape. Phase 4 W1 added a mandatory third field `permission: Record<string, unknown>` to `PermissionsResponse`. The service now calls `resolveUserEffectivePermissionService`, which is **not mocked** in this test file. When the test suite runs, the mock chain for `checkIsAdminByEmployeeIdService` is set up but `resolveUserEffectivePermissionService` is not mocked at all — the real implementation will be called against unmocked repositories, likely throwing or producing an unexpected result. Even if it does not throw, `toEqual({ userUuid, isAdmin })` will fail because the real response now includes `permission`.

**Evidence:** The test mocks `@/modules/v1/externalAuth/externalAuth.repository` and `@/modules/v1/admin/compPermission/compPermission.service` only. It does not mock `@/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service` (added in W1).

**Suggested fix:** Add `jest.mock('@/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service', ...)` to mock `resolveUserEffectivePermissionService` returning a test fixture, and update both assertions to include `permission: expect.any(Object)` (or a specific fixture value). The 404 test case is unaffected (it throws before reaching the resolver).

---

### M1. (medium) P4.4 `assertPermissionKeysWithinCompanyMenusService` — unnecessary resolver call for `{}` JSONB

**File:** `src/modules/v1/admin/compPermission/compPermission.service.ts:48-58`

**Issue:** The shortcut at line 48 only returns early when `!permissionJsonb && !permissionMobileJsonb`. An empty `{}` object is **truthy** in JavaScript, so `!{}` = `false`. When both `permission` and `permissionMobile` are `{}` (a common "no overrides" payload for a new role), the code proceeds to call `resolveCompanyEffectiveMenuKeysService` and fetches the full resolver chain from DB, only to exit at line 58 (`desiredKeys.length === 0`) after `collect({})` returns `[]`. This is an unnecessary 4–5 DB query round-trip on every save of an empty-permission role.

**Related test quality note:** The test `"IT-L: empty {} JSONB → no validation called"` (line 549) has a misleading title. The resolver **is** called; the test only passes by accident because `mockResolveMenuKeys` returns `undefined` (cleared mock) and `allowedKeys.has()` is never reached (early-return before it). The comment `"Resolver may or may not be called"` reflects the ambiguity.

**Suggested fix:** Change the shortcut to also cover empty objects:
```typescript
const isEmpty = (v: object | null | undefined): boolean =>
  !v || (typeof v === 'object' && !Array.isArray(v) && Object.keys(v).length === 0);
if (isEmpty(permissionJsonb) && isEmpty(permissionMobileJsonb)) return;
```
Update the test title and add `expect(mockResolveMenuKeys).not.toHaveBeenCalled()` to lock the behavior.

---

### M2. (medium) `updateCompPermissionByUuidController` — double DB read for the same record

**File:** `src/api/v1/admin/compPermission/compPermission.controller.ts:210` + `src/modules/v1/admin/compPermission/compPermission.service.ts:181`

**Issue:** `updateCompPermissionByUuidController` calls `getCompPermissionByUuidService(uuid)` (line 210) to validate existence and get `companyId`. This service now resolves the full resolver chain for the JSONB filter. Then `updateCompPermissionByUuidService` internally calls `getCompPermissionByUuid(uuid, ['id', 'companyId'])` again (line 181) to retrieve `companyId` for P4.4 validation. This is a second DB read of the same record. The service-level re-read was designed to avoid recursive self-call (per the comment), but the controller already has the `companyId` from the first call.

**Note:** This pattern predates Phase 4 partially (controller already called `getCompPermissionByUuidService` for existence check), but Phase 4's P4.4 introduced the second service-level lookup making it 2 DB reads instead of 1.

**Suggested fix:** Pass `compPermission.companyId` from the controller into `updateCompPermissionByUuidService` as an optional parameter (e.g., `existingCompanyId?: string`). When provided, skip the internal `getCompPermissionByUuid` lookup. This is a safe additive change with no interface breakage.

---

### L1. (low) `permissions.service.ts:45` — `logger.error` used with `level: 'info'` context for non-error path

**File:** `src/modules/v1/externalAuth/permissions.service.ts:45`

**Issue:** `logger.error('User has no permission mapping...', { level: 'info', userUuid })` is called in a non-error branch (user simply has no mapping yet — expected state for new users). Using `logger.error` for an informational event pollutes error dashboards/alerting with noise. The comment acknowledges the diagnostic logger lacks a `warn`/`info` level, which is the underlying limitation.

**Suggested fix:** Suppress this log entirely (the downstream `permission: {}` response is the correct silent behavior per the comment), or if observability is needed, open a separate task to add an `info` level to the diagnostic logger. This is a pre-existing limitation flagged at resolver level (Phase 3 L1); accept as-is or fix the logger.

---

### L2. (low) `compPermission.ts` Zod schema — `z.record(z.string(), z.any())` violates no-`any` rule

**File:** `src/schemas/compPermission.ts:14,15,37,38,137,138`

**Issue:** Six schema fields (`permission` and `permissionMobile`) are typed as `z.record(z.string(), z.any())`. The project rules (CLAUDE.md §10 + backend CLAUDE.md FORBIDDEN) prohibit `any`. These lines predate Phase 4 partially but were touched in W3.5 Fix 3.

**Suggested fix:** Use `z.record(z.string(), z.unknown())` as a direct substitute. The JSONB shape is validated semantically by the P4.4 service layer — the Zod schema needs only to accept any valid JSON object, which `z.unknown()` achieves without the `any` escape.

---

### L3. (low) Test `IT-J` (line 632) — logic inline in test rather than testing actual controller code

**File:** `tests/integration/sale-dashboard/permission-resolver.integration.spec.ts:632-652`

**Issue:** The IT-J test for backward compatibility (no `companyUuid` → full baseline) inlines the controller's branch logic directly in the test body (manually setting `permission = PermissionDefault` in an `if/else`). It does not actually call `getCompPermissionDefaultListController` or compose the controller's code path; it just re-implements the conditional. This means if the controller's shortcut condition changes (e.g., from `length > 0` to `!== undefined`), the test would still pass falsely.

**Suggested fix:** Extract the test to call the controller function through a mock Express `req/res`, or at minimum call `filterPermissionByMenuKeysService` + `resolveCompanyEffectiveMenuKeysService` through the same function that the controller calls (mirroring what IT-I does for the `companyUuid` branch). This is a low-priority change given 25/25 pass but should be addressed before Phase 5.

---

## Section: Filter Completeness

All five read paths are correctly filtered:

| Path | Filter applied | Verdict |
|------|---------------|---------|
| `/api/external/auth/permissions/:uuid` (W1) | `resolveUserEffectivePermissionService` wraps filter | OK |
| `/auth/user-info` (`getEmployeePermissionService`) | baseline pre-filter + post-deepMerge second-pass filter | OK |
| `getCompPermissionByUuidService` (admin role-edit) | service-level filter of saved JSONB | OK |
| `getCompPermissionByUuidController` (admin deepMerge) | baseline pre-filter before deepMerge (Fix 2) | OK — but double resolver cost (H1) |
| `/comp-permission/default?companyUuid=` (Fix 3) | filter applied when `companyUuid` present | OK |

`getCompPermissionByIdService` (line 241) does **not** select `permission`/`permissionMobile` fields (only `id, uuid, name, description, isOwner`) — no filter needed.

Hidden caller check: `grep deepMerge.*PermissionDefault` found no unguarded caller outside the reviewed files. The `auth.service.ts` call at line 1132 routes through `getEmployeePermissionService` which is fully filtered.

---

## Section: Owner Case (Q-A=STRICT)

Fix 1 in `getEmployeePermissionService` filters `PermissionDefault` **before** `deepMerge`. Since `deepMerge(filteredBaseline, ownerJSONB, isOwner=true)` operates on a filtered baseline, the owner's `setAllPermissionsTrue` behavior can only reach keys already present in the filtered tree. The post-deepMerge second-pass filter (allowedKeys) further eliminates any leftover key. Both layers enforce STRICT. IT-G test confirms owner parity. **Verified OK.**

Fix 2 in `getCompPermissionByUuidController` applies `filterPermissionByMenuKeysService(PermissionDefault, allowedKeys)` before `deepMerge(filteredDefault, permission, isOwner)`. Same logic — owner cannot surface a menu absent from `allowedKeys`. **Verified OK.**

---

## Section: Save Validation (P4.4)

`assertPermissionKeysWithinCompanyMenusService` validates both `permission` (web) and `permissionMobile` correctly. Empty `{}` does not trigger an error (desiredKeys = [] → early return). `null`/`undefined` both short-circuit via the `collect` helper. The conditional guard `if (data.permission !== undefined || data.permissionMobile !== undefined)` in `updateCompPermissionByUuidService` correctly skips validation when payload has neither field.

One concern: the resolver IS called unnecessarily for `{}` inputs (M1 above).

Concurrent race (company toggles feature off while save is in-flight): the P4.4 check validates at save-time against the current effective set. If the toggle races ahead, the next save attempt will catch it. If the save wins the race, the saved JSONB may contain a now-disabled key, but the read paths filter it on every response. This is an acceptable eventual-consistency behavior given no transaction spanning two tables is required.

---

## Section: In-request Cache

- `getPermissionsService`: `new Map()` per call, passed to `resolveUserEffectivePermissionService`. GC'd after response. No global state.
- `getEmployeePermissionService`: `new Map()` per call, passed to `resolveCompanyEffectiveMenuKeysService`. Multiple `p.companyId` values in a multi-permission employee are all the same company → single resolver hit. If a user somehow spans companies (pathological case), each company key would be cached separately within the same map.
- `getCompPermissionByUuidController`: two separate `new Map()` instances (H1) — performance issue but no correctness issue.
- No global Map or module-level cache found. Cache pollution risk: none.

---

## Section: Backward Compatibility

`getCompPermissionDefaultListSchema` at `compPermission.ts:31` uses `z.string().trim().uuid('...').optional()` — correctly optional. Old clients omitting `companyUuid` fall through to the `if (typeof companyUuid === 'string' && companyUuid.length > 0)` guard which returns the full `PermissionDefault`. **Backward compat: OK.**

---

## Section: Test Quality

- 25/25 integration tests pass.
- Mock isolation: `beforeEach(() => jest.clearAllMocks())` correctly resets between tests.
- Fixture realism: test uses real `PermissionDefault` (imported from constant) and real `filterPermissionByMenuKeysService` — high confidence the filter shape is correct.
- `resolveCompanyEffectiveMenuKeysService` is mocked and the test asserts it was called with the correct `companyId` and `expect.any(Map)`.
- Gap: IT-J test does not exercise actual controller code (L3).
- Gap: empty `{}` test has misleading title and ambiguous assertion (M1).
- DEF-001: stale unit test will fail in CI (H2).

---

## Section: DEF-001 Triage

**Decision: BLOCKER.** The stale unit test at `tests/unit/externalAuth/permissions.service.test.ts` will fail when CI runs the full test suite because `resolveUserEffectivePermissionService` is now called in the real `getPermissionsService` but is not mocked in this test file. The response shape assertion `toEqual({ userUuid, isAdmin })` will also fail because the actual response now has a third field `permission`. This must be fixed before push (H2 above).

---

## Section: Code Style

- TypeScript: no `any` usage in service/controller TypeScript files. `z.any()` in Zod schemas (L2 — pre-existing, minor).
- Functional style: no `class`/`this` usage. Pure functions (`filterPermissionByMenuKeysService`, `extractMenuKeysFromPermissionJsonbService`).
- Logger pattern: mostly consistent. One misuse of `logger.error` for info-level event (L1).
- Prettier: reported as passing by developer agent — not re-run here, trusted per policy.
- N+1: no loop-per-row DB calls. All batch queries use `WHERE IN` pattern through existing resolver.

---

## Praise

- The two-pass filter in `getEmployeePermissionService` (baseline filter + post-deepMerge re-filter) is a thoughtful defense against Q-C=no-migration legacy data. The second pass catches leftover keys from old JSONB rows that deepMerge would otherwise inject back in.
- `extractMenuKeysFromPermissionJsonbService` is correctly co-located in the resolver service (not compPermission service) and is a clean pure function.
- Cache design (Option C: explicit `ResolverCache` parameter) is testable and avoids global state — good pattern.
- P4.4 validates both web and mobile JSONB in a single helper with deduplication (`new Set(desiredKeys.filter(...))`), avoiding partial validation drift.

---

## Recommendations (post-fix actions before push)

1. **Fix H2 (DEF-001) first** — add resolver mock + update assertions in `tests/unit/externalAuth/permissions.service.test.ts`.
2. **Fix H1** — eliminate double resolver call in `getCompPermissionByUuidController/Service`. Preferred: share cache by passing it from controller into service.
3. **Fix M1** — add `isEmpty` shortcut in `assertPermissionKeysWithinCompanyMenusService` for `{}` inputs and tighten the IT-L empty test assertion.
4. M2 (double DB read in update path) — can be deferred to Phase 5 cleanup if sprint timeline is tight.
5. L2 (`z.any()` → `z.unknown()`) — low risk, can batch with next Zod schema change.

**Final verdict after H1+H2 fixes: READY for BA.**

---

## Phase 5 W5 — Post-CR BUG FIX (2026-05-06)

### Bug summary

Phase 4 P4.4 helper `assertPermissionKeysWithinCompanyMenusService` รวม leaf keys ของทั้ง `permission` (web) + `permission_mobile` (mobile) เข้าด้วยกัน แล้ว validate ผ่าน effective **web** menu keys อันเดียว — wrong namespace สำหรับ mobile path. CR Phase 4 §Praise §200 เคย note ว่า "validates both web and mobile JSONB in a single helper with deduplication" → ตอนนั้นยังไม่มี mobile namespace แยก (Wave 3 ยังไม่ได้ implement) จึง pass review. หลังจาก Phase 5 Wave 3 + Wave 4 implement mobile resolver path → bug surface: mobile keys ไม่ตรงกับ web namespace → admin save mobile permission ถูก reject 400 ทุกครั้ง (หรือ pass เพราะบังเอิญ key ชื่อตรง — แต่ save ผิด context).

### Fix scope (Phase 5 Wave 5 — P5.5.1..P5.5.5)

| File | Fix |
|------|-----|
| `src/modules/v1/admin/compPermission/compPermission.service.ts` | Split helper เป็น 2 path independent: web ผ่าน `extractMenuKeys`+`resolveCompanyEffectiveMenuKeysService`, mobile ผ่าน `extractMobileMenuKeys`+`resolveCompanyEffectiveMobileMenuKeysService`. `getCompPermissionByUuidService` filter `permissionMobile` ผ่าน mobile filter+keys (เดิมใช้ web ผิด). New error code `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` (400) distinct จาก web. |
| `src/api/v1/admin/compPermission/compPermission.controller.ts` | `getCompPermissionByUuidController` filter `PermissionMobileDefault` baseline ผ่าน mobile filter+keys (เดิมใช้ web ทั้ง 2 baseline). `getCompPermissionDefaultListController` เปลี่ยนเดียวกัน. Cache shared (`Promise.all` parallel resolve) → mobile hit `featureIds`+`packageId` cache (saved 2 batch DB calls). |

### Backward compat impact

- Mobile validation = NEW path → old clients ที่ไม่ส่ง `permission_mobile` ไม่กระทบ
- Mobile validation strict — old saves ที่มี invalid mobile keys (ก่อน Wave 5) จะ reject 400 → intended (bug correction)
- Web error code unchanged — clients ที่จัดการ `INVALID_MENU_KEY_FOR_COMPANY` คงเดิม
- New error code `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` distinct → frontend Wave 6 จะ surface แยก toast

### Verification

- TS pass + prettier conformant
- Grep: `INVALID_MOBILE_MENU_KEY_FOR_COMPANY` defined + thrown ใน service.ts
- Grep: `extractMobileMenuKeysFromPermissionJsonbService` + `resolveCompanyEffectiveMobileMenuKeysService` + `filterPermissionMobileByMenuKeysService` import + use ใน service + controller
- Q-A=STRICT propagated to mobile path — `getCompPermissionByUuidController` baseline-filter-before-deepMerge pattern คงไว้ทั้ง web + mobile
