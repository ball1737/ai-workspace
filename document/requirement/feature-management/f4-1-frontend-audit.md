# F4.1 Frontend Permission UI Audit

**Date:** 2026-05-06
**Auditor:** developer agent (read-only audit; no code change)
**Scope:** Phase 4 Wave 3 — does Phase 4 server-side filter (P4.3+P4.4 + P4.2) reach the user-visible UI for "menu of company-disabled feature"?

---

## Summary

- **Repos surveyed:** `happywork-frontend` (main user app — owns runtime nav + role-edit UI), `happywork-sale-cms` (admin tool — owns master feature/menuKeys editing). No separate `happywork-backend-cms`.
- **Render source:** **HYBRID** — frontend always merges incoming `permission` JSONB on top of a `PermissionDefault` tree fetched from backend (`GET /comp-permission/default`). The `default` tree is the **unfiltered global** `PermissionDefault` constant.
- **Phase 4 filter effect:**
  - Runtime nav (`useNavData → buildMenuFromPermission`) iterates **`userInfo.permission`** keys → would benefit from filter — **BUT** `userInfo.permission` is built by `getEmployeePermissionService` which **also `deepMerge`s `PermissionDefault`** (auth.service.ts:1132 + compPermissionMapping.service.ts:106-123). This path bypasses Phase 4 P4.2 filtering. Result: **filter has NO effect on runtime menu visibility today.**
  - Role edit UI (`/dashboard/user-setting/edit`) merges `default` (full tree from `/comp-permission/default`) with the saved JSONB. Phase 4 P4.3 service filter on `GET /comp-permission/:uuid` is **overlaid back** by both backend controller `deepMerge(PermissionDefault, …)` AND frontend `mergePermissions(default, specific)` → filter is fully neutralized.
- **Hardcoded menu list:** **2 hardcoded sources**: (a) backend `PermissionDefault`/`PermissionMobileDefault` returned via `/comp-permission/default`; (b) frontend `MENU_REGISTRY` (icon + path mapping) in `features/layouts/configs/menu-registry.tsx` (and `MOBILE_MENU_REGISTRY` in `config-navigation-mobile.tsx`).
- **Owner case impact:** `isOwner=true` paths in `deepMerge` set every action `true` for every leaf — owners see **all menus regardless of company-disabled features**. (Confirmed at compPermissionMapping.service.ts:111-115 and compPermission.controller.ts:147-148.)
- **Recommendation:** **Frontend changes alone are NOT sufficient.** Phase 4 must extend P4.2 to either (i) make `/auth/user-info`'s permission attachment go through the resolver-aware filter, or (ii) move the `deepMerge(PermissionDefault, …)` step out of `getEmployeePermissionService` + `getCompPermissionByUuidController` and let services emit pre-filtered JSONB. Frontend `mergePermissions(default, specific)` and `getPermissionDefault` likewise need filter awareness OR removal.
- **Report path:** `/Users/ball/Desktop/ai-space/ai-workspace/document/requirement/feature-management/f4-1-frontend-audit.md`

---

## Findings

### Section 1 — Files surveyed

| Path | Purpose | Render source |
|------|---------|---------------|
| `happywork-frontend/src/app/dashboard/user-setting/create/components/permission-table/index.tsx` | Renders permission tree as table (read/create/update/delete/export checkboxes) | Receives `value: PermissionGroup` prop; pure renderer driven by whatever object is passed |
| `happywork-frontend/src/app/dashboard/user-setting/create/components/permission-table/utils.ts` | `getPermissionDefault()` → `GET /comp-permission/default`; `getFilteredPermissionGroupList` keyword filter | Backend hardcoded `PermissionDefault` |
| `happywork-frontend/src/app/dashboard/user-setting/create/components/permission-table/hooks.ts` | `useDefaultPermission`, `useDefaultPermissionMobile` (React Query, `staleTime: Infinity`) | Same as above |
| `happywork-frontend/src/app/dashboard/user-setting/create/components/step-permission.tsx` | Wrapper for create/edit role; calls `mergePermissions(default, specific)` (lines 77-101, 121, 157) | **Hybrid** — `default` (from `/comp-permission/default`) overlaid by saved JSONB |
| `happywork-frontend/src/app/dashboard/user-setting/create/components/form/utils.ts` | `getUserRole(uuid)` → `GET /comp-permission/:uuid` then derives form values | Saved JSONB from server (already deepMerged on backend by `getCompPermissionByUuidController`) |
| `happywork-frontend/src/features/layouts/configs/menu-registry.tsx` | Static icon + path + `permissionKey` registry for sidebar (file ~250+ lines, ~15 top-level groups) | **Hardcoded** registry; iterates `userInfo.permission` keys to filter |
| `happywork-frontend/src/features/layouts/configs/config-navigation.tsx` (lines 26-115) | `buildMenuFromPermission(userInfo?.permission, MENU_REGISTRY, t)` — runtime nav builder | Reads `userInfo.permission` JSONB to gate visibility (`read === true`) |
| `happywork-frontend/src/features/layouts/configs/config-navigation-mobile.tsx` (line 436) | Mobile nav variant with separate `MOBILE_MENU_REGISTRY` | Same pattern, separate hardcoded registry |
| `happywork-frontend/src/features/dashboard/store/userInfo.ts` (line 27) | Zustand store; `GET /auth/user-info` populates `userInfo.permission` | Backend computes via `getEmployeePermissionService → deepMerge(PermissionDefault, …)` — **NOT** filtered by Phase 4 P4.2 |
| `happywork-frontend/src/types/stores/userInfo.type.ts` (lines 165-247) | Static type listing **every** known top-level + nested permission key | TypeScript shape only; not a runtime list |
| `happywork-frontend/src/hooks/use-permission.ts` | `usePermission()` reads `userInfo.permission` for action gating | Same upstream as runtime nav |
| `happywork-sale-cms/src/components/feature-management/menu-keys-select.tsx` | Tree-checkbox component for editing `feature.menuKeys` (admin) — receives `availableKeys: string[]` prop | Admin-only; consumes `availableMenuKeys` returned by Phase 2 backend (full PermissionDefault leaf set) — out of Phase 4 scope |
| `happywork-sale-cms/src/sections/feature-management/feature-detail-view.tsx` (line 273) | Caller of MenuKeysSelect for feature CRUD | Same as above |
| `happywork-sale-cms/src/sections/package-management/components/package-effective-menus-preview.tsx` (line 83) | Preview package effective menus (sale-cms admin) | Out of Phase 4 user-visible scope |

### Section 2 — Permission UI flow

**Two distinct UIs touch permission JSONB. Both depend on the same hardcoded default, both chain through `deepMerge(PermissionDefault, …)`.**

#### 2.1 Runtime nav (every authenticated user, every page load)
```
Login → /auth/user-info
  └─ auth.service.ts:1132  getEmployeePermissionService(employeeId)
       └─ compPermissionMapping.service.ts:106-123
            ├─ permissions = getCompPermissionMappingByEmployeeId(...)   // raw JSONB rows
            ├─ permission = PermissionDefault                            // hardcoded full tree
            └─ permissions.forEach(p => permission = deepMerge(permission, p.permission, isOwner))
       └─ returns { permission, permissionMobile, isOwner, roles }
       
Frontend: useUserInfoStore.userInfo.permission (zustand)
  └─ config-navigation.tsx  useNavData()
       └─ buildMenuFromPermission(userInfo.permission, MENU_REGISTRY, t)
            ├─ if !permission → showAll = true (every registry entry)
            └─ else → iterate Object.keys(permission), filter by registry, sort by sequence
                       → child.read === true ? show : hide
```
**Phase 4 P4.2 filter status: NOT applied.** P4.2 only wired `resolveUserEffectivePermissionService → filterPermissionByMenuKeysService` into `/external/auth/permissions` (`permissions.service.ts`). The legacy `/auth/user-info` path used by `happywork-frontend` is untouched.

#### 2.2 Role edit UI (admin only — `/dashboard/user-setting/edit?uuid=…`)
```
Page load
 ├─ useUserRole(uuid)             →  GET /comp-permission/:uuid
 │     └─ Backend: getCompPermissionByUuidService (Phase 4 P4.3 filters JSONB) ✅
 │     └─ Backend: controller deepMerge(PermissionDefault, filtered, isOwner) ❌  ← undoes the filter
 │
 └─ useDefaultPermission()         →  GET /comp-permission/default
       └─ Backend: getCompPermissionDefaultListController returns RAW PermissionDefault   ← never filtered
                   (compPermission.controller.ts:111-127)

Render
 └─ step-permission.tsx
      └─ mergePermissions(default, specific)   // line 77-101 — frontend hybrid
           → JSON.parse(JSON.stringify(default))   → walk into specific keys → overwrite
      → PermissionTable value=mergedPermissions
```
**Phase 4 P4.3 filter status: filtered, then re-overlaid by 2 different `default` injections.** Result for admin editing a role on a company that has feature X disabled: the menu under feature X **still appears** in the table (because `PermissionDefault` re-injects it). Admin can tick boxes; on Save, Phase 4 P4.4 will reject the request with `INVALID_MENU_KEY_FOR_COMPANY 400`. UX gap.

#### 2.3 Role view (no edit) — same flow as 2.2 with `disabled={true}`. Same gap.

### Section 3 — Hardcoded menu lists

| # | Source | Path | Size | Impact |
|---|--------|------|------|--------|
| 1 | `PermissionDefault` constant (server) | `happywork-backend/src/constant/compPermission.ts` | ~20 top-level keys, ~70+ leaves | Returned as-is by `GET /comp-permission/default`; embedded into every `userInfo.permission` and every `getCompPermissionByUuid` response via `deepMerge`. **Single biggest blocker** for Phase 4 user-visible filtering. |
| 2 | `MENU_REGISTRY` (frontend) | `happywork-frontend/src/features/layouts/configs/menu-registry.tsx` | ~15 top-level groups | Pure UI metadata (icon + i18n key + route). Iterated against incoming `permission` keys — does NOT itself add menus, only renders those that are in the JSONB. **Does not bypass filter** if upstream JSONB is properly filtered. |
| 3 | `MOBILE_MENU_REGISTRY` (frontend) | `happywork-frontend/src/features/layouts/configs/config-navigation-mobile.tsx:44` | ~similar | Same nature as #2; safe. |
| 4 | `IUserInfo.permission` type (frontend) | `happywork-frontend/src/types/stores/userInfo.type.ts:165-247` | static enumeration | TypeScript-only; no runtime impact, but couples FE to current backend tree shape. |
| 5 | Sale-CMS `MenuKeysSelect` `availableKeys` prop | `happywork-sale-cms/src/components/feature-management/menu-keys-select.tsx:23,61` | varies | Receives full `availableMenuKeys` from Phase 2 backend; admin-only feature-CRUD UI; **not user-visible in Phase 4 scope** (master-data editing for super-admin). |

### Section 4 — deepMerge analysis

**Where deepMerge runs:**

| Location | File:Line | Effect |
|----------|-----------|--------|
| Login / refresh user info | `happywork-backend/src/modules/v1/admin/compPermissionMapping/compPermissionMapping.service.ts:106-123` | `permission = PermissionDefault → deepMerge(permission, role.permission, isOwner)` for every assigned role; result attached to `userInfo.permission`. |
| Admin reads single role | `happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts:147-148` | `deepMerge(PermissionDefault, permission ?? {}, isOwner)` — re-adds keys the Phase 4 service just filtered out. |
| Frontend role edit form | `happywork-frontend/src/app/dashboard/user-setting/create/components/step-permission.tsx:77-101, 121, 157` | `mergePermissions(default, specific)` — same outcome as backend deepMerge, but with `default` from a separate API call. |

**Effect of Phase 4 P4.3 filter (current Wave 2 service-level filtering) on user-visible behavior:**

- **Non-owner users (runtime nav):** None. The runtime nav reads `userInfo.permission`, which never goes through P4.2's `resolveUserEffectivePermissionService` — the deepMerge in `getEmployeePermissionService` re-injects every disabled menu. Sequence: filter neutralized **before** it ever reaches the client.
- **Non-owner users (admin role-edit table):** None visible. Even though `getCompPermissionByUuidService` filters keys (Wave 2), the controller re-`deepMerge`s `PermissionDefault` immediately, and the frontend separately fetches `/comp-permission/default` and merges again. Net: admin sees every menu in the table → can tick checkboxes for company-disabled features → save call rejected by Phase 4 P4.4.
- **Owner (`isOwner=true`):** Worse. `deepMerge`'s third arg short-circuits per-leaf override; for owners every action becomes `true` for every leaf in `PermissionDefault`. Owners see all menus across all features regardless of company entitlement. Confirmed in `helper.ts` deepMerge lines ~175-235 (per Files Read Memory) — when `isOwner=true`, override path is bypassed.

### Section 5 — Recommendations

#### Action items (require code change in a separate Phase 4 wave; out of Wave 3 scope)

1. **(Backend, MUST)** Wire the resolver-aware filter into `getEmployeePermissionService` (`compPermissionMapping.service.ts:106-123`) so `/auth/user-info` ships pre-filtered JSONB. Rough shape:
   - After per-role deepMerge loop, call `filterPermissionByMenuKeysService(permission, effectiveMenuKeys)` using the resolver cache (single resolver call per request).
   - For `permissionMobile` likewise.
2. **(Backend, MUST)** Decide policy for `getCompPermissionByUuidController.deepMerge(PermissionDefault, …)` (lines 147-148):
   - **Option A:** Remove deepMerge (let frontend render only what server returns). Cleanest; matches Phase 4 P4.4 invariant.
   - **Option B:** Replace with `deepMerge(filterPermissionByMenuKeysService(PermissionDefault, effectiveMenuKeys), permission, isOwner)` — preserves the "default tree shape" UX but trims to company-effective.
3. **(Backend, MUST)** Filter `getCompPermissionDefaultListController` by company effective menus (currently returns raw PermissionDefault — `compPermission.controller.ts:111-127`). Likely accept `companyUuid` query param OR derive from `req.user.companyId` and call resolver before responding. Without this, frontend `useDefaultPermission` keeps re-injecting unfiltered keys.
4. **(Backend, MUST)** Owner override semantics. Decision needed: should an owner's effective permission include menus of features the **company itself disabled**? Current `deepMerge` says yes (everything `true`). Phase 4 invariant says no (P4.4 will reject saves of those keys). Two choices to align:
   - Keep `isOwner=true` semantics ("owner gets all"), but constrain "all" to `effectiveMenuKeys` (resolver-filtered) instead of full `PermissionDefault`.
   - Drop the `isOwner` shortcut entirely and rely on stored JSONB.
5. **(Frontend, SHOULD)** After backend changes (1)+(3), `mergePermissions(default, specific)` becomes safe (because `default` is already trimmed). No code change required. Optionally simplify `step-permission.tsx` to drop the merge once backend always returns the union.
6. **(Frontend, NICE-TO-HAVE)** `MENU_REGISTRY` already gates by `userInfo.permission` keys — once (1) lands, hidden features disappear automatically. No FE change needed for the runtime nav.

#### Open questions (need Lead / SA decision before code change)

- **Q-A: Owner policy.** Does an owner who has a feature **administratively disabled by their company override** still see that feature's menus? (Recommendation: **No** — keep parity with non-owner; aligns with P4.4 reject behavior.)
- **Q-B: Default tree endpoint scope.** Should `/comp-permission/default` be retained as a "blank template for create role" use case (super-admin only), or replaced with `/comp-permission/default?companyUuid=…` returning effective tree per company? Affects role-create UX (admin currently sees full tree to tick → some ticks become invalid).
- **Q-C: Migration of existing role JSONBs.** Existing rows in `comp_permission.permission` JSONB may contain keys for features that are now disabled at company level. P4.4 will reject **updates** that include those keys, but reads still flow through deepMerge. Should we bulk-prune existing JSONBs, or rely on per-read filter only?

---

## Decision matrix สำหรับ Lead

| Scenario | Phase 4 server filter applies? | UI matches Phase 4 invariant? | Action needed |
|----------|--------------------------------|-------------------------------|---------------|
| User logs in → sees runtime sidebar | **No** — `getEmployeePermissionService` deepMerges raw `PermissionDefault` | No (sees disabled features) | Backend change in `compPermissionMapping.service.ts` (Recommendation 1) |
| Admin opens role edit page (non-owner role) | Partial — service filters, controller `deepMerge` re-adds | No (sees disabled features as ticked options) | Backend change in `compPermission.controller.ts:147-148` (Recommendation 2) + `/comp-permission/default` filter (Recommendation 3) |
| Admin opens role edit page (owner role) | None — `isOwner=true` short-circuits override | No (sees + can tick everything) | Decide owner policy (Q-A) → backend change in `deepMerge` semantics or `getEmployeePermissionService` (Recommendation 4) |
| Admin saves edited role for non-owner | Yes — Phase 4 P4.4 rejects 400 | Yes, but UX shows error AFTER user ticks invalid | Tighten UI before save: depends on Recommendation 3 |
| External /api/external/auth/permissions consumer | Yes — Phase 4 P4.2 filter wired | Yes | None (verified Wave 1 done) |
| Sale-CMS super-admin editing feature.menuKeys | Out of scope (master data, pre-Phase 4) | N/A | None |

---

## Verification artifacts (no code modified)

- `compPermission.controller.ts:147-148` — confirmed `deepMerge(PermissionDefault, permission ?? {}, rest.isOwner)` post-filter.
- `compPermission.controller.ts:111-127` — `getCompPermissionDefaultListController` returns raw constants.
- `compPermissionMapping.service.ts:106-123` — `getEmployeePermissionService` deepMerges `PermissionDefault` for `/auth/user-info`.
- `auth.service.ts:1132,1170` — `userInfo.permission` plumbed from `getEmployeePermissionService`.
- `step-permission.tsx:77-101,121,157` — frontend `mergePermissions(default, specific)`.
- `permission-table/utils.ts:4-13` — `getPermissionDefault → GET /comp-permission/default`.
- `permission-table/hooks.ts:4-26` — React Query with `staleTime: Infinity` (cached for session).
- `config-navigation.tsx:26-115` — `buildMenuFromPermission` reads `userInfo.permission` keys.
- `userInfo.ts:27` — `GET /auth/user-info` populates `userInfo.permission`.
- `menu-registry.tsx:56` — static `MENU_REGISTRY`; pure metadata, not a menu list source.
