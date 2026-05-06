---
feature: feature-management
type: ba-validation
phase: 4
date: 2026-05-06
validator: ba-agent
verdict: PASS-with-notes
---

# BA Validation Report — feature-management Phase 4

**Date:** 2026-05-06
**Phase:** Phase 4 — Resolver Integration + Regression
**Verdict:** PASS-with-notes
**Findings:** 0 critical / 0 high (all H1+H2 from CR fixed) / 3 non-critical notes

---

## Summary

Phase 4 implements FR-5 (Permission Resolver Integration) ของ `requirement.md` ใน 4 hot paths บวก save validation. Code อ่านแล้วตรงกับ checklist ที่ tick ครบ. CR findings H1+H2+M1+M2+L1+L2 ถูก fix แล้วและยืนยันด้วย code จริง. 25/25 integration tests + 5/5 unit tests + 17/17 sibling regression ผ่าน. ไม่พบ critical gap.

มี 3 notes เชิง non-critical ที่ควรติดตาม post-push (ไม่ block push):
1. IT-J test ยัง inline controller logic (L3 deferred)
2. IT-L empty `{}` test title ยังไม่ได้ tighten assertion (CR M1 fix ทำ early-return ถูกต้องแล้ว แต่ test comment ยัง ambiguous)
3. L1 logger workaround ยังคงอยู่ (TODO documented สำหรับ Phase 5)

---

## Section 1 — Requirement Coverage (FR-5 + Q-A/B/C)

| Req | Description | Status | Evidence |
|-----|-------------|--------|----------|
| FR-5 | ทุก endpoint ที่ return comp_permission.permission ต้อง filter ผ่าน effective features | PASS | 4 hot paths filter แล้ว — ดู Section 2 |
| FR-5 AC-1 | `GET /api/external/auth/v1/permissions` → filtered JSONB | PASS | `permissions.service.ts:39` — `resolveUserEffectivePermissionService` wired |
| FR-5 AC-2 | Permission management CMS → filter เมนูตาม company | PASS | `getCompPermissionByUuidController` + baseline pre-filter (Fix 2) |
| FR-5 AC-3 | Save permission → validate ต้องไม่มี key นอก enabled menus | PASS | `assertPermissionKeysWithinCompanyMenusService` ใน create+update service |
| FR-5 AC-4 | Resolver logic: `effective = (pkg.features ∪ addon.features ∪ overrides[enabled]) − overrides[disabled]` | PASS | `resolveCompanyEffectiveFeatureIdsService` ใน `permissionResolver.service.ts:99` |
| FR-5 AC-5 | In-request memoization — ไม่ N+1 | PASS | `ResolverCache = new Map<string, unknown>()` per request, CR-H1 fix cache-sharing |
| Q-A = STRICT | Owner ของ company ที่ feature ปิด → ไม่เห็น menu | PASS | `getEmployeePermissionService:132` filter baseline ก่อน deepMerge; IT-G confirms |
| Q-B | `/comp-permission/default` รับ `companyUuid` optional | PASS | `getCompPermissionDefaultListSchema.query.companyUuid` optional UUID; controller lines 128–135 |
| Q-C | No data migration | PASS | per-read filter เท่านั้น; post-deepMerge second-pass re-filter กัน legacy JSONB leftover |
| BR-7 | Effective = `(pkg ∪ addon ∪ override_enabled) − override_disabled` | PASS | `permissionResolver.service.ts:99–140` |
| BR-8 | comp_permission JSONB เก็บเต็ม — filter ตอนอ่าน | PASS | Code ไม่มี mutation ของ stored JSONB |

---

## Section 2 — Filter Completeness across Endpoints

| Endpoint / Path | Service / Code | Filter Applied | Status |
|----------------|----------------|----------------|--------|
| `GET /api/external/auth/permissions/:uuid` (W1) | `permissions.service.ts:39` | `resolveUserEffectivePermissionService(userUuid, cache)` | PASS |
| `/auth/user-info` via `getEmployeePermissionService` (W3.5 Fix 1) | `compPermissionMapping.service.ts:131–159` | baseline pre-filter + post-deepMerge re-filter | PASS |
| `getCompPermissionByUuidService` (W2) | `compPermission.service.ts:153–165` | `resolveCompanyEffectiveMenuKeysService` → `filterPermissionByMenuKeysService` | PASS |
| `getCompPermissionByUuidController` deepMerge baseline (W3.5 Fix 2) | `compPermission.controller.ts:179–184` | `filteredDefault = filterPermissionByMenuKeysService(structuredClone(PermissionDefault), allowedKeys)` | PASS |
| `GET /comp-permission/default?companyUuid=` (W3.5 Fix 3) | `compPermission.controller.ts:128–135` | filter both defaults เมื่อ companyUuid present | PASS |
| `getCompPermissionListService` (list) | unchanged | ไม่ return permission JSONB (per checklist P4.3 confirm) | PASS (no-op correct) |
| `getCompPermissionByIdService` | unchanged | select ไม่รวม permission field | PASS (no-op correct) |

ทุก 5 read path ที่ return permission JSONB ถูก filter แล้ว ตรงกับ FR-5 requirement.

---

## Section 3 — Checklist Completeness

| Item | Tick Status | Code Verified |
|------|------------|---------------|
| P4.1 controller thin (resolver ใน service) | ✅ done | `permissions.controller.ts` — ไม่มี resolver call, thin try/catch |
| P4.2 `getPermissionsService` resolver wired | ✅ done | `permissions.service.ts:39` |
| P4.3 `getCompPermissionByUuidService` filter | ✅ done | `compPermission.service.ts:153–165` |
| P4.4 create/update save validation | ✅ done | `assertPermissionKeysWithinCompanyMenusService:41–68` |
| W3.5.1 Fix 1 `getEmployeePermissionService` | ✅ done | `compPermissionMapping.service.ts:111–168` |
| W3.5.2 Fix 2 baseline pre-filter controller | ✅ done | `compPermission.controller.ts:174–184` |
| W3.5.3 Fix 3 `/comp-permission/default` companyUuid | ✅ done | `compPermission.controller.ts:128–135` + Zod schema line 31 |
| W3.5.4 Fix 4 Owner STRICT (no extra code needed) | ✅ done | Transitively via Fix 1 + Fix 2 |
| F4.1 Frontend audit done | ✅ done | `f4-1-frontend-audit.md` |
| P4.5+P4.6 Integration test 25/25 | ✅ done | file: `permission-resolver.integration.spec.ts` |
| F4.2 API-level UAT smoke (IT-G/H/I/J/K/L) | ✅ done | embedded in 25 test cases |
| CR-H1 double resolver call fix | ✅ done | `getCompPermissionByUuidService(uuid, cache?)` signature + controller pass cache |
| CR-H2 DEF-001 unit test fix | ✅ done | `permissions.service.test.ts` — mock + 5/5 PASS |
| CR-M1 unnecessary resolver for `{}` | ✅ done | `desiredKeys.length === 0 → return` at line 59 BEFORE resolver call |
| CR-M2 double DB read update | ✅ done | `updateCompPermissionByUuidService(uuid, data, existingCompanyId?)` + controller pass `compPermission.companyId` |
| CR-L1 logger TODO comment | ✅ done | `permissions.service.ts:45` comment added |
| CR-L2 `z.any()` → `z.unknown()` | ✅ done | `compPermission.ts` lines 14,15,37,38,137,138 — verified grep clean |
| CR-L3 | ☐ DEFERRED | acknowledged in checklist (25/25 still pass) |

---

## Section 4 — SSD ↔ Code Consistency

| SSD Spec | Code | Status |
|----------|------|--------|
| §4.5 `GET /api/external/auth/v1/permissions/:userUuid` — Phase 4 adds resolver filter | `permissions.service.ts:39` call chain | PASS |
| §6.2 Resolve User Permission Flow | Full chain matches SSD diagram (load mapping → companyId → resolver → filter → response) | PASS |
| §7 Error Handling: PERMISSION_DATA_NOT_FOUND → graceful `{}` | `permissions.service.ts:42–44` catch block | PASS |
| §7 Error Handling: PERMISSION_DATA_CORRUPTED → propagate | `permissions.service.ts:49` re-throw | PASS |
| §7 Error Handling: COMPANY_NOT_FOUND / SEED_PACKAGE_NOT_FOUND → propagate | `compPermissionMapping.service.ts:135–144` re-throw | PASS |
| Response shape backward compat (add `permission` field only) | `PermissionsResponse` interface: `userUuid`, `isAdmin`, `permission` — existing fields unchanged | PASS |
| `/comp-permission/default` backward compat (no companyUuid → full baseline) | `compPermission.controller.ts:128` guard `typeof companyUuid === 'string' && length > 0` | PASS |
| Zod schema `companyUuid` optional UUID | `compPermission.ts:31` — `z.string().trim().uuid().optional()` | PASS |

---

## Section 5 — Test Coverage Assessment

| Test File | Count | Result | Coverage |
|-----------|-------|--------|----------|
| `permission-resolver.integration.spec.ts` | 25 | 25/25 PASS | W1, W2, W3.5 Fix1/2/3, P4.4, IT-A→L |
| `permissions.service.test.ts` (unit) | 5 | 5/5 PASS | resolver mock, PERMISSION_DATA_NOT_FOUND graceful, CORRUPTED propagate, 404 case |
| `companyFeature.integration.spec.ts` (regression) | 17 | 17/17 PASS | Phase 3 no regression |
| **Total** | **47** | **47/47 PASS** | |

IT mapping vs testcase.md IT-A→IT-L:
- IT-A (toggle override → filter): covered via `W3.5 Fix 1 / IT-A` test
- IT-B (addon → menu surfaces): covered via `W3.5 Fix 1 / IT-B` test
- IT-C (override disabled → absent): covered via `W3.5 Fix 1 / IT-C` test
- IT-D (remove override → restore): covered via `W3.5 Fix 1 / IT-D` test
- IT-E (no override baseline): covered via `W3.5 Fix 1 / IT-E` test
- IT-F (shape + no error): covered via `W3.5 Fix 1 / IT-F` + W1 tests
- IT-G (owner STRICT): covered via `W3.5 Fix 1 / IT-G` test
- IT-H (non-owner parity): covered via `W3.5 Fix 1 / IT-H` test
- IT-I (default?companyUuid filter): covered via `W3.5 Fix 3 / IT-I` test
- IT-J (default no query backward compat): covered via `W3.5 Fix 3 / IT-J` test
- IT-K (admin role-edit filter): covered via `W2 + W3.5 Fix 2 / IT-K` test (×2)
- IT-L (save validation 400 + accept + empty + update): covered via `P4.4 / IT-L` (5 cases)

---

## Section 6 — CR Fix Verification

| Finding | Severity | Fix Status | Code Evidence |
|---------|---------|------------|---------------|
| H1 Double resolver call | High | FIXED | `getCompPermissionByUuidService(uuid, cache?)` at line 131; controller creates 1 cache, passes to service + reuse at line 179 |
| H2 DEF-001 stale unit test | High | FIXED | `permissions.service.test.ts` mock `resolveUserEffectivePermissionService` + 5 tests PASS |
| M1 Unnecessary resolver for `{}` | Medium | FIXED | `desiredKeys.length === 0` early-return at line 59, before resolver call at line 61 |
| M2 Double DB read in update | Medium | FIXED | `updateCompPermissionByUuidService(uuid, data, existingCompanyId?)` + controller passes `compPermission.companyId` |
| L1 logger.error for info-level path | Low | DOCUMENTED | TODO comment at `permissions.service.ts:45` |
| L2 `z.any()` → `z.unknown()` | Low | FIXED | 6 occurrences in `compPermission.ts` all use `z.unknown()` |
| L3 IT-J inline logic | Low | DEFERRED | Documented in checklist; 25/25 tests still pass |

ข้อสังเกตเพิ่มเติมจาก BA: M1 fix ใช้ `collect({})` → `extractMenuKeysFromPermissionJsonbService({})` → loop เหนือ empty object → คืน `[]` → `desiredKeys.length === 0 → return` ทำงานถูกต้อง แต่ test case `IT-L: empty {} JSONB` ที่ comment ว่า "Resolver may or may not be called" ยัง ambiguous (CR-M1 fix ทำ behavior ชัดเจนแล้วว่า NOT called) — test assertion ควร `expect(mockResolveMenuKeys).not.toHaveBeenCalled()` เพื่อ lock behavior

---

## Section 7 — Backward Compatibility

| Change | Impact | Status |
|--------|--------|--------|
| `getCompPermissionByUuidService` เพิ่ม `cache?: ResolverCache` | Optional param — caller ไม่ส่งก็ fallback สร้าง `new Map()` ใหม่ | PASS — `compPermissionMapping.controller.ts:64,178` ยังเรียกโดยไม่ส่ง cache ได้ |
| `updateCompPermissionByUuidService` เพิ่ม `existingCompanyId?: string` | Optional param — ไม่ส่ง → lookup DB ตามเดิม | PASS |
| `/comp-permission/default` รับ `companyUuid` query | Optional — old client ที่ไม่ส่ง → คืน full PermissionDefault | PASS — guard `typeof companyUuid === 'string' && companyUuid.length > 0` |
| `/api/external/auth/permissions/:uuid` เพิ่ม `permission` field | Additive only — existing fields `userUuid`, `isAdmin` unchanged | PASS |
| Zod schema `permission: z.record(z.string(), z.unknown())` | `unknown` ⊂ `object` — downstream interface `CreateCompPermissionInterface.permission: object` compatible | PASS |

---

## Section 8 — Owner Case (Q-A=STRICT) Verification

**Decision:** Q-A=STRICT — owner ของ company ที่ feature ปิด → ไม่เห็น menu ของ feature นั้น

**Implementation verified:**

1. `getEmployeePermissionService` (`compPermissionMapping.service.ts:131–159`):
   - `resolveCompanyEffectiveMenuKeysService(companyId, cache)` → `allowedKeys`
   - `basePermission = filterPermissionByMenuKeysService(structuredClone(PermissionDefault), allowedKeys)`
   - `deepMerge(basePermission, p.permission, isOwner)` — baseline filtered ก่อน → owner deepMerge bypass override per-leaf แต่ baseline คือ subset ของ company-enabled menus เท่านั้น
   - Post-deepMerge second-pass: `filterPermissionByMenuKeysService(permission, allowedKeys)` — กัน leftover จาก legacy JSONB
   - **Result:** owner ไม่เห็น menu ของ feature ที่ company ปิด ✅

2. `getCompPermissionByUuidController` (`compPermission.controller.ts:180–184`):
   - `filteredDefault = filterPermissionByMenuKeysService(structuredClone(PermissionDefault), allowedKeys)`
   - `deepMerge(filteredDefault, permission ?? {}, rest.isOwner)` — ใช้ filtered baseline
   - **Result:** owner role-edit JSONB ก็ไม่มี key นอก company effective menus ✅

**Test evidence:** IT-G test — owner isOwner=true, allowedKeys=`Set(['dashboard'])` only, role JSONB carries `payroll.runPayroll` → result `payroll.runPayroll` NOT in leaves ✅

---

## Section 9 — Outstanding / Notes (Non-Critical)

| # | Severity | Description | Suggested action |
|---|----------|-------------|-----------------|
| N1 | Non-critical | `IT-J` test (`permission-resolver.integration.spec.ts:632`) inlines controller branch logic instead of calling actual controller code (CR-L3 deferred) | Track for Phase 5; refactor to call controller via mock req/res |
| N2 | Non-critical | `IT-L: empty {} JSONB` test comment "Resolver may or may not be called" misleading — after CR-M1 fix, resolver is definitively NOT called. Assertion should add `expect(mockResolveMenuKeys).not.toHaveBeenCalled()` | Fix test assertion in next test maintenance wave |
| N3 | Non-critical | `logger.error` with `level: 'info'` context at `permissions.service.ts:46` — TODO documented for Phase 5 logger refactor | Resolve when diagnostic logger gains `info` level |

---

## Section 10 — Recommendation (Push Readiness)

**Verdict: PASS-with-notes**

Phase 4 implementation:
- ครอบคลุม FR-5 ครบทุก acceptance criteria
- Q-A=STRICT enforced ทั้ง 2 paths (getEmployeePermissionService + getCompPermissionByUuidController)
- Q-B backward compat ผ่าน Zod optional + controller guard
- Q-C no-migration ป้องกันด้วย post-deepMerge second-pass filter
- CR findings H1+H2+M1+M2+L1+L2 fixed ครบ
- 47/47 tests pass (25 integration + 5 unit + 17 regression)
- Backward compat: additive-only response field + optional signature changes

**พร้อม push:** YES — ไม่มี blocker. 3 notes ข้างต้นเป็น cosmetic/test-quality ที่ไม่กระทบ correctness หรือ security.

**Post-push recommendations (Phase 5 backlog):**
- Fix N2: tighten IT-L empty `{}` assertion
- Fix N1: refactor IT-J to call actual controller
- N3: upgrade logger when `info` level available
