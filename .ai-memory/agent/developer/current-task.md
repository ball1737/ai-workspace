---
agent: developer
updated: 2026-05-06
---

# Developer Agent — Current Task

> งานที่ Developer กำลังถืออยู่ — Lead เป็นคน assign
> Developer = full-stack agent (backend + frontend ใน session เดียว)

---

## Active Task

**Phase 4 Wave 3.5** — Audit Fix Wave: 4 fixes wiring resolver filter into endpoints F4.1 audit flagged
- Fix 1: `getEmployeePermissionService` (`/auth/user-info` hot path) — filter PermissionDefault baseline + JSONB
- Fix 2: `getCompPermissionByUuidController` — filter PermissionDefault baseline before deepMerge
- Fix 3: `getCompPermissionDefaultListController` — accept optional `companyUuid` query param + filter
- Fix 4: Owner case STRICT (Q-A) — covered transitively by Fix 1+2

Source: `document/requirement/feature-management/f4-1-frontend-audit.md` + Lead-confirmed Q-A=STRICT, Q-B=companyUuid optional, Q-C=no migration

(Phase 3 — done 2026-05-06; Phase 3 CR Fix Wave — done 2026-05-06; Phase 4 W1 — done 2026-05-06; Phase 4 W2 — done 2026-05-06; Phase 4 W3 audit — done 2026-05-06)

---

## Active Track

backend

---

## Files Read Memory (Session-scoped)

| Path | Read on | Edited | Key takeaways |
|------|---------|--------|---------------|
| happywork-backend/src/modules/v2/sale-dashboard/packageCrud/packageCrud.service.ts | 2026-05-06 | no | Reference pattern: pre-fetch + N+1 prevention with batch repos; getPackageEffectiveMenusService close to resolver semantics |
| happywork-backend/src/modules/v2/sale-dashboard/packageCrud/packageCrud.repository.ts | 2026-05-06 | no | Reference patterns: buildBaseQuery filtering active/inactive, batch repos with WHERE IN + GROUP BY, getPackageFeaturesRepository JOIN comp_package_features |
| happywork-backend/src/database/postgresql/models/data/compCompanyFeatures.model.ts | 2026-05-06 | no | Override table model — overrideStatus 'enabled'/'disabled', reason nullable, FK to comp_features (Q4=B RESTRICT) |
| happywork-backend/src/database/postgresql/models/data/compCompanyAddons.model.ts | 2026-05-06 | no | Per-company addons replaces JSONB; has expiresAt nullable; Q3=A semantics filter at runtime |
| happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts | 2026-05-06 | no | Has menuKeys: string[] JSONB attribute; statusType filter at active/inactive |
| happywork-backend/src/database/postgresql/models/data/compPackages.model.ts | 2026-05-06 | no | Standard model with id/uuid; needs lookup by code='seed' for fallback |
| happywork-backend/src/database/postgresql/models/data/compCompanies.model.ts | 2026-05-06 | no | packageId is optional (Phase 2); Resolver fallback when null |
| happywork-backend/src/database/postgresql/models/data/compPermission.ts | 2026-05-06 | no | comp_permission has companyId + permission JSONB + permissionMobile JSONB |
| happywork-backend/src/database/postgresql/models/data/compPermissionMapping.ts | 2026-05-06 | no | comp_permission_mapping links permission_id to user_id/employee_id/department_id/group_id (NOT directly to userUuid) |
| happywork-backend/src/database/postgresql/models/data/compPackageFeatures.model.ts | 2026-05-06 | no | Composite PK (packageId, featureId), no soft-delete, no audit |
| happywork-backend/src/database/postgresql/models/data/compAddonFeatures.model.ts | 2026-05-06 | no | Composite PK (addonId, featureId), no soft-delete |
| happywork-backend/src/database/postgresql/models/data/compAddons.model.ts | 2026-05-06 | no | Standard addon model |
| happywork-backend/src/modules/v1/externalAuth/externalAuth.repository.ts | 2026-05-06 | no | findEmployeeByUserUuidRepository: 2-strategy lookup (Employee.sso_user_uuid OR auth_users → employee.user_id); returns companyId. Pattern to reuse for getUserPermissionJsonbRepository |
| happywork-backend/src/modules/v1/externalAuth/permissions.service.ts | 2026-05-06 | no | Existing permissions.service uses findEmployeeByUserUuid + checkIsAdmin; no JSONB filtering yet |
| happywork-backend/src/constant/compPermission.ts | 2026-05-06 | no | PermissionDefault tree, leaf detected by hasActionKey (read/create/update/delete/export); collectLeafPaths walker — same logic to reuse for filterPermissionByMenuKeys |
| happywork-backend/src/constant/general.ts | 2026-05-06 | no | StatusTypeEnumCode enum: ACTIVE='active' |
| happywork-backend/src/utils/errorMethods.ts | 2026-05-06 | no | errorNotFound, errorInternal, errorBadRequest helpers — return CustomError with msg/status/code |
| document/requirement/feature-management/ssd.md (§6.2, §7) | 2026-05-06 | no | Resolver flow + error scenarios; Q3=A addon expiry; Q4=B feature.status_type='active' filter; Seed fallback when company.package_id NULL |
| document/requirement/feature-management/backend.md (§2.5) | 2026-05-06 | no | Resolver service signatures: 4 functions, in-request memoization for N+1 prevention |
| document/requirement/feature-management/checklist.md (P3.7-P3.9) | 2026-05-06 | yes | P3.7-P3.9 ticked + sub-items expanded with actual function names + path sync note (sale-dashboard router) |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.interface.ts | 2026-05-06 | yes (created) | EffectiveFeatureIds, EffectiveMenuKeys, ResolverCache, CompanyOverrideRow, UserPermissionRow, PermissionJsonb types |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts | 2026-05-06 | yes (created) | 7 repo fns: company.packageId lookup + Seed fallback, package/addon feature ids batch, company overrides JOIN active, feature menu_keys via LATERAL unnest, user permission JSONB lookup |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts | 2026-05-06 | yes (created) | 4 public services with Option-C cache param; pure filterPermissionByMenuKeysService recursive walker; uses logger.error level:'warn' since logger.warn not exposed |
| happywork-backend/src/api/v2/sale-dashboard/feature/feature.routes.ts | 2026-05-06 | no | Phase 2 reference — middleware chain `validateSaleDashboardAccessToken → requireSuperAdmin → validateRequest`; leaf-path-first registration; tags pattern |
| happywork-backend/src/api/v2/sale-dashboard/feature/feature.controller.ts | 2026-05-06 | no | Reference: resolveActorUuid helper (req.user.uuid ?? userId), handleError + res.success pattern |
| happywork-backend/src/modules/v2/sale-dashboard/feature/feature.adapter.ts | 2026-05-06 | no | Reference: toMultilingual helper, strip db id, response with uuid only |
| happywork-backend/src/modules/v2/sale-dashboard/feature/feature.service.ts | 2026-05-06 | no | Reference: error handling + adapter pattern |
| happywork-backend/src/modules/v2/sale-dashboard/feature/feature.interface.ts | 2026-05-06 | no | Reference: Zod schema shape `{body, query, params}` + types |
| happywork-backend/src/modules/v2/sale-dashboard/feature/feature.repository.ts | 2026-05-06 | no | Reference: ACTIVE_STATUSES const, buildBaseQuery, batch repo with WHERE IN + GROUP BY |
| happywork-backend/src/middlewares/requireSuperAdmin.middleware.ts | 2026-05-06 | no | role==='super_admin' check; throws AppError 403 |
| happywork-backend/src/middlewares/saleDashboardAuth.middleware.ts (lines 60-90) | 2026-05-06 | no | Sets req.user with uuid, userId, saleId, role, email; via SaleDashboardUsers query |
| happywork-backend/src/api/v2/sale-dashboard/sale-dashboard.routes.ts | 2026-05-06 | yes | Phase 3 mount at `/companies` registered after `/features`/`/packages`/`/addons`, before catch-all `/` mounts |
| happywork-backend/src/api/v2/sale-dashboard/packageCrud/packageCrud.routes.ts | 2026-05-06 | no | Reference: leaf-path-first registration order (specific `:uuid/features` before `/:uuid` generic) |
| happywork-backend/src/modules/v2/sale-dashboard/payments/payments.repository.ts (lines 120-135) | 2026-05-06 | no | Existing `getCompanyByUuidRepository` returns shape with name string; companyFeature uses own lookup returning packageId for resolver |
| happywork-backend/src/database/postgresql/models/data/compFeatures.model.ts | 2026-05-06 | no (re-confirmed) | sortOrder field exists for ordering; jsonAttributes ['name','description','menuKeys'] |
| document/requirement/feature-management/ssd.md (§3.3 + §4.4 + §7) | 2026-05-06 | no | CompanyFeatureItem shape line 302-309; API spec (note path stale `/v2/companies/...`); error scenarios incl. concurrent toggle row 8 |
| document/requirement/feature-management/backend.md (§2.4 lines 280-322) | 2026-05-06 | no | companyFeature.service signatures + source decision logic (override > package > addon > default-disabled) |
| document/requirement/feature-management/checklist.md (P3.1-P3.6) | 2026-05-06 | yes | P3.1-P3.6 ticked + sub-items expanded with actual function names + Q9=A path sync note |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.interface.ts | 2026-05-06 | yes (created) | Zod schemas (3) + CompanyFeatureItem + OverrideRowDb + OverrideStatusType + CompanyFeatureSource + CompanyFeatureListResponse |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.repository.ts | 2026-05-06 | yes (created) | 6 repo fns: getCompanyByUuid, listActiveFeatures, getFeatureByUuidForOverride, getOverridesByCompany (JOIN active filter), upsertOverride (raw ON CONFLICT), removeOverride (hard delete) |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.adapter.ts | 2026-05-06 | yes (created) | Pure computeCompanyFeatureItem + toCompanyFeatureItems; source priority override > package > addon > default-disabled |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts | 2026-05-06 | yes (created) | 3 public services + buildCompanyFeatureList (4 parallel batches via Promise.all + addon-feature-ids sequential 1 hop); Seed fallback; race handling 23505/40001/40P01 → 409 |
| happywork-backend/src/api/v2/sale-dashboard/companyFeature/companyFeature.controller.ts | 2026-05-06 | yes (created) | 3 controllers; resolveActorUuid; res.success + handleError |
| happywork-backend/src/api/v2/sale-dashboard/companyFeature/companyFeature.routes.ts | 2026-05-06 | yes (created) | 3 routes mounted at /companies; specific paths first; tags 'Company Features - V2 Sale Dashboard' |
| happywork-sale-cms/src/types/store/sale-dashboard-feature-management.ts | 2026-05-06 | no | Phase 2 reference for Root state shape pattern + Multilingual reuse |
| happywork-sale-cms/src/store/services/feature-management-request.ts | 2026-05-06 | no | Phase 2 reference for sd_method_GET/PUT/DELETE wrappers + JSDoc URL pattern |
| happywork-sale-cms/src/store/services/package-management-request.ts | 2026-05-06 | no | Phase 2 closeout pattern: JSDoc uses `/api/v2/sale-dashboard/...` (post URL fix) — followed for new file |
| happywork-sale-cms/src/store/actions/sale-dashboard-feature-management.ts | 2026-05-06 | no | Phase 2 reference: createActions REQUEST/SUCCESS/FAILURE triplet + cleanup helpers |
| happywork-sale-cms/src/store/reducers/sale-dashboard-feature-management.ts | 2026-05-06 | no | Phase 2 reference: handleActions + pickErrorMessage + useCallback hook pattern (CR fix M1 with eslint-disable) |
| happywork-sale-cms/src/store/sagas/sale-dashboard-feature-management.ts | 2026-05-06 | no | Phase 2 reference: takeLatest watcher + RequestAction generic type |
| happywork-sale-cms/src/store/reducers/sale-dashboard-package-config-feature.ts | 2026-05-06 | no | Legacy slice using `sdGetFeaturesRequest`/`sdToggleFeatureRequest` — explains why Wave 3 actions use `*CompanyFeature*` naming to avoid collision |
| happywork-sale-cms/src/store/services/sale-dashboard.ts | 2026-05-06 | yes | URL builders centralized; legacy `companyFeatures(id)` at line ~186 targets `/company-settings/...` (collision avoided via `saleDashboard*` prefix); added 2 new builders for Phase 3 |
| happywork-sale-cms/src/store/services/sale-dashboard-request.ts | 2026-05-06 | no | sd_method_* axios wrappers; SuccessHandler shape `{success, data, status, code, msg}` — `data` is unwrapped from API envelope |
| happywork-sale-cms/src/types/store/index.ts | 2026-05-06 | yes | Registered _SaleDashboardCompanyFeatures namespace + RootState slot |
| happywork-sale-cms/src/store/reducers/index.ts | 2026-05-06 | yes | Registered saleDashboardCompanyFeatures reducer slot |
| happywork-sale-cms/src/store/sagas/index.ts | 2026-05-06 | yes | Registered watchSaleDashboardCompanyFeatures in rootSaga |
| happywork-sale-cms/src/store/actions/index.ts | 2026-05-06 | yes | Re-exports sale-dashboard-company-features actions |
| happywork-sale-cms/src/types/store/sale-dashboard-company-features.ts | 2026-05-06 | yes (created) | F3.1 — types: CompanyFeatureSource union, CompanyFeatureItem, request/response payloads, Root state with togglingFeatureUuid lock |
| happywork-sale-cms/src/store/services/company-features-request.ts | 2026-05-06 | yes (created) | F3.2 — 3 typed wrappers via sd_method_GET/PUT/DELETE; uses saleDashboardCompanyFeatures + saleDashboardCompanyFeatureDetail builders |
| happywork-sale-cms/src/store/actions/sale-dashboard-company-features.ts | 2026-05-06 | yes (created) | F3.3 actions — 3 triplets (get/set/remove) + 2 cleanup; failure actions for set/remove also carry featureUuid |
| happywork-sale-cms/src/store/reducers/sale-dashboard-company-features.ts | 2026-05-06 | yes (created) | F3.3 reducer + F3.4 useSaleDashboardCompanyFeatures hook; replaceRowByFeatureUuid immutable map; per-row togglingFeatureUuid lock; useCallback wrappers |
| happywork-sale-cms/src/store/sagas/sale-dashboard-company-features.ts | 2026-05-06 | yes (created) | F3.3 saga — takeLatest GET, takeEvery set/remove (rapid multi-row toggle never cancels each other) |
| document/requirement/feature-management/frontend.md (§3 + §1.4 lines 100-372) | 2026-05-06 | no | Confirmed Q11=A naming; hook spec; tab section name `feature-tab/company-features-tab-view.tsx` (Wave 4) |
| document/requirement/feature-management/ssd.md (§3.3 + §4.4) | 2026-05-06 | no | CompanyFeatureItem shape; PUT/DELETE return single item (not list-wrapped); URL `/api/v2/sale-dashboard/companies/:uuid/features` (path corrected from stale `/v2/companies/...`) |
| document/requirement/feature-management/checklist.md (F3.1-F3.4) | 2026-05-06 | yes | F3.1-F3.4 ticked + sub-items expanded with naming-decision justification + technical notes |
| happywork-sale-cms/src/sections/client-management/company/list-view.tsx | 2026-05-06 | yes | Existing client detail view; Tabs at line ~350; added `features` Tab + conditional CompanyFeaturesTabView render — uses `useParams().uuid` |
| happywork-sale-cms/src/components/feature-management/feature-source-badge.tsx | 2026-05-06 | no | Phase 2 reusable badge — uses `companyFeatures.source.*` keys; size + color map by source; consumed unchanged in row component |
| happywork-sale-cms/src/locales/use-locales.ts + config-lang.ts | 2026-05-06 | no | `useLocales()` returns `{t, currentLang}`; `getBilingualText(name, currentLang.value)` for multilingual rendering |
| happywork-sale-cms/src/layouts/stores/toastStore.ts | 2026-05-06 | no | Toast API: `toast({message, type})` (NOT `{title, description, status}`) — fixed view to match |
| happywork-sale-cms/src/types/store/sale-dashboard-company-features.ts | 2026-05-06 | no | Wave 3 types reused — `CompanyFeatureItem`, `CompanyFeatureSource`, payloads |
| happywork-sale-cms/src/store/reducers/sale-dashboard-company-features.ts | 2026-05-06 | no | Wave 3 hook `useSaleDashboardCompanyFeatures()` exposes list + togglingFeatureUuid + dispatchers (all useCallback-wrapped) |
| happywork-sale-cms/src/locales/langs/en.json | 2026-05-06 | yes | Added `client_management.features` + expanded `companyFeatures.{filter,row,dialog,error,success}` (37 cF keys total) |
| happywork-sale-cms/src/locales/langs/th.json | 2026-05-06 | yes | Same TH parity (37 cF keys) — verified via JSON-flatten diff |
| happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx | 2026-05-06 | yes (created) | F3.5 orchestrator — load on mount + clearList cleanup; counts + filteredList useMemo; toast on listError/toggleError + clearError |
| happywork-sale-cms/src/sections/client-management/feature-tab/components/company-feature-list.tsx | 2026-05-06 | yes (created) | F3.6 — Skeleton×4 loading, no_features empty, maps to row |
| happywork-sale-cms/src/sections/client-management/feature-tab/components/company-feature-row.tsx | 2026-05-06 | yes (created) | F3.7 — bilingual name + FeatureSourceBadge + Switch.onClick intercept |
| happywork-sale-cms/src/sections/client-management/feature-tab/components/company-feature-toggle-confirm-dialog.tsx | 2026-05-06 | yes (created) | F3.8 — hasOverride 3-button mode vs 2-button; reason 0/500 counter; reset on open |
| happywork-sale-cms/src/sections/client-management/feature-tab/components/company-feature-source-filter.tsx | 2026-05-06 | yes (created) | F3.9 — Select 6 options with count suffix; types exported for parent |
| document/requirement/feature-management/checklist.md (F3.5-F3.10) | 2026-05-06 | yes | Ticked F3.5–F3.10 with sub-item expansion; F3.11 left for QA |
| document/requirement/feature-management/cr-phase3.md | 2026-05-06 | no | CR report — verdict PASS-with-fixes; 9 findings (0/2/3/4); BLOCKED on H1+H2 |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.repository.ts | 2026-05-06 | yes | CR-H1 — removed `getOverridesByCompanyRepository` (duplicate of resolver's `getCompanyOverridesRepository`); kept upsert + remove |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.interface.ts | 2026-05-06 | yes | CR-H1 — `OverrideRowDb` now type alias of `CompanyOverrideRow` from resolver.interface |
| happywork-backend/src/modules/v2/sale-dashboard/companyFeature/companyFeature.service.ts | 2026-05-06 | yes | CR-H1+M1+M3 — uses `getCompanyOverridesRepository` from resolver repo; uses `resolveEffectivePackageIdService`; removed `'23505'` from race-catch |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.repository.ts | 2026-05-06 | yes | CR-M2+L4 — added `jsonb_typeof(f.menu_keys) = 'array'` guard; `select('packageId')` only (no `id`) |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts | 2026-05-06 | yes | CR-M3 — extracted `resolveEffectivePackageIdService` (cached via `packageId:{companyId}`); inner Seed-fallback now reuses it |
| happywork-sale-cms/src/store/sagas/sale-dashboard-company-features.ts | 2026-05-06 | yes | CR-H2 — removed 3× `console.error` worker logs + `console.log` watcher banner; failure actions still dispatched |
| happywork-sale-cms/src/sections/client-management/feature-tab/company-features-tab-view.tsx | 2026-05-06 | yes | CR-L2 — `wasSubmittingRef` + new `useEffect` auto-closes dialog when `togglingFeatureUuid` clears for `targetItem`; confirm handlers no longer close synchronously |
| document/requirement/feature-management/checklist.md (Phase 3 CR Fix Wave) | 2026-05-06 | yes | Appended new section listing 7 fixed + 2 deferred items with rationale + verification log |
| happywork-backend/src/modules/v1/externalAuth/permissions.service.ts | 2026-05-06 | yes | P4.2 — added resolver call with per-request ResolverCache; PERMISSION_DATA_NOT_FOUND → empty {} graceful fallback; other resolver errors propagate |
| happywork-backend/src/modules/v1/externalAuth/permissions.interface.ts | 2026-05-06 | yes | P4.2 — added `permission: Record<string, unknown>` field with JSDoc explaining empty-fallback semantics |
| happywork-backend/src/api/external/auth/permissions.controller.ts | 2026-05-06 | no | P4.1 — confirmed already has try/catch + handleError + res.success; no change required (response shape extends automatically via PermissionsResponse) |
| happywork-backend/src/utils/errorMethods.ts | 2026-05-06 | no | CustomError shape: `code?: string \| number` — confirms `(error as {code?: string}).code` matches resolver's errorNotFound 'PERMISSION_DATA_NOT_FOUND' |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.interface.ts | 2026-05-06 | no | Re-confirmed ResolverCache = Map<string, unknown>; PermissionJsonb = Record<string, unknown> — matched permission field type |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts | 2026-05-06 | no | Re-confirmed `resolveUserEffectivePermissionService(userUuid, cache?)` signature + error codes (PERMISSION_DATA_NOT_FOUND, PERMISSION_DATA_CORRUPTED) |
| document/requirement/feature-management/checklist.md (P4.1-P4.2) | 2026-05-06 | yes | Ticked P4.1 + P4.2 + sub-items with controller-untouched rationale + edge-case fallback note |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts | 2026-05-06 | yes | Wave 2 P4.3+P4.4 — added resolver imports + `assertPermissionKeysWithinCompanyMenusService` helper; `createCompPermissionService` + `updateCompPermissionByUuidService` ทำ pre-validate (reject INVALID_MENU_KEY_FOR_COMPANY 400); `getCompPermissionByUuidService` filter `permission`+`permissionMobile` JSONB ผ่าน `filterPermissionByMenuKeysService`; CustomError code preservation in catch |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.repository.ts | 2026-05-06 | no | Confirmed `getCompPermissionList` repo selects `id, uuid, createdAt, updatedAt, statusType, description, name` only — NO `permission`/`permissionMobile` → list endpoint ไม่ต้อง filter; filter contract มี `companyId` (single-company assumption) |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.interface.ts | 2026-05-06 | no | `UpdateCompPermissionInterface` ไม่มี `companyId` field → service must lookup companyId via repo before validation; `CreateCompPermissionInterface` มี `companyId` พร้อม |
| happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts | 2026-05-06 | no | Controller บน update path เรียก `getCompPermissionByUuidService` ก่อน update — Wave 2 service add JSONB filter to that response. ห้าม recursive call: `updateCompPermissionByUuidService` ใช้ `getCompPermissionByUuid` (repo, raw) ไม่ใช่ service. Controller `deepMerge(PermissionDefault, ...)` ทีหลัง — ดู Open Question ใน final report |
| happywork-backend/src/database/postgresql/models/data/compPermission.ts | 2026-05-06 | no | `permission?: object` + `permissionMobile?: object` — Objection auto-deserializes JSONB → no JSON.parse needed |
| happywork-backend/src/utils/helper.ts (deepMerge lines 175-235) | 2026-05-06 | no | deepMerge starts จาก `structuredClone(base)` — ทุก key ของ PermissionDefault ปรากฏใน result เสมอ; isOwner=true → bypasses override → controller deepMerge เป็น scope ของ frontend audit (F4.1) |
| happywork-backend/src/constant/compPermission.ts (lines 270-324) | 2026-05-06 | no | Existing `collectLeafPaths` + `getAvailableMenuKeys()` — semantics ตรงกับ `extractMenuKeysFromPermissionJsonbService` ใหม่ (operates บน arbitrary JSONB tree); ใช้ ACTION_KEYS / NON_CHILD_KEYS แบบเดียวกันใน resolver (consistent leaf-detection rule) |
| happywork-backend/src/utils/errorMethods.ts | 2026-05-06 | no (re-confirmed) | `errorBadRequest(msg, code, data)` รองรับ `data` field — ใช้ส่ง `invalidKeys` list ใน 400 response |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts | 2026-05-06 | yes | Wave 2 — added `extractMenuKeysFromPermissionJsonbService(jsonb)` (collect leaf paths) co-located กับ `filterPermissionByMenuKeysService` — ใช้ ACTION_KEYS/NON_CHILD_KEYS/hasActionKey เดียวกัน |
| happywork-backend/src/modules/v1/externalAuth/permissions.service.ts | 2026-05-06 | no (re-confirmed) | Wave 1 reference pattern — per-request `new Map<string, unknown>()` cache + try/catch around resolver call → reused ใน compPermission service |
| document/requirement/feature-management/checklist.md (P4.3-P4.4) | 2026-05-06 | yes | Ticked P4.3 + P4.4 + sub-items with path-correction note (P4.4 ทำที่ compPermission ไม่ใช่ compPermissionMapping) + single-company assumption note |
| document/requirement/feature-management/f4-1-frontend-audit.md | 2026-05-06 | no | F4.1 audit report — 4 fixes spec: (1) getEmployeePermissionService filter baseline + JSONB; (2) getCompPermissionByUuidController filter PermissionDefault before deepMerge; (3) getCompPermissionDefaultListController accept companyUuid query; (4) Owner=STRICT covered transitively. Q-A=STRICT, Q-B=companyUuid optional, Q-C=no migration |
| happywork-backend/src/modules/v1/admin/compPermissionMapping/compPermissionMapping.service.ts | 2026-05-06 | yes | W3.5 Fix 1 — getEmployeePermissionService filter PermissionDefault baseline (structuredClone + filterPermissionByMenuKeysService) ก่อน deepMerge; pick companyId จาก permission row; post-merge re-filter pass ใช้ allowedKeys ที่ resolved แล้ว (no resolver re-call); Q-A=STRICT enforced |
| happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts | 2026-05-06 | yes | W3.5 Fix 2 — getCompPermissionByUuidController filter PermissionDefault + PermissionMobileDefault baseline ก่อน deepMerge (ใช้ rest.companyId จาก service); W3.5 Fix 3 — getCompPermissionDefaultListController รับ companyUuid query optional → getCompanyByUuidService → filter |
| happywork-backend/src/schemas/compPermission.ts | 2026-05-06 | yes | W3.5 Fix 3 — getCompPermissionDefaultListSchema.query รับ companyUuid optional UUID (backward compat) |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts | 2026-05-06 | no (re-confirmed) | W3.5 confirmed: getCompPermissionByUuidService คืน companyId ใน rest (ใช้ใน controller filter); Wave 2 service-level filter ของ saved JSONB ยังคงทำงาน |
| happywork-backend/src/modules/v1/admin/compPermissionMapping/compPermissionMapping.repository.ts | 2026-05-06 | no | getCompPermissionMappingByEmployeeId JOIN compPermission + select compPermission.* — return rows มี companyId field (ใช้ใน W3.5 Fix 1) |
| happywork-backend/src/database/postgresql/models/data/compPermission.ts | 2026-05-06 | no (re-confirmed) | CompPermissionType มี companyId: string field — confirmed for W3.5 Fix 1 row.companyId access |
| happywork-backend/src/modules/v1/admin/companies/companies.service.ts | 2026-05-06 | no | getCompanyByUuidService (line 144) — throws AppError 400 if not found; returns CompCompaniesType (with id field) — used in W3.5 Fix 3 |
| happywork-backend/src/modules/v1/admin/companies/companies.repository.ts | 2026-05-06 | no | getCompanyByUUID returns full company row including id — used by getCompanyByUuidService |
| happywork-backend/src/constant/compPermission.ts (PermissionDefault export) | 2026-05-06 | no (re-confirmed) | PermissionDefault + PermissionMobileDefault env-aware export (Prod/Dev variants); structurally Record<string, unknown> — cast as PermissionJsonb safe |
| document/requirement/feature-management/cr-phase4.md | 2026-05-06 | no | CR Phase 4 report — verdict PASS-with-fixes; 7 findings (0/2/2/3); BLOCKED on H1+H2 — task source |
| happywork-backend/src/api/v1/admin/compPermissionMapping/compPermissionMapping.controller.ts (lines 55-90, 170-195) | 2026-05-06 | no | Other callers of getCompPermissionByUuidService — read only `permission.id` + `permission.companyId`; no permission/permissionMobile usage → cache-share refactor doesn't affect them |
| happywork-backend/src/modules/v2/sale-dashboard/permissionResolver/permissionResolver.service.ts | 2026-05-06 | no (re-confirmed) | Cache-key naming `menuKeys:{companyId}` reused across calls — confirms shared Map ทำให้ second call hits cache (H1 fix premise) |
| happywork-backend/src/utils/errorMethods.ts | 2026-05-06 | no (re-confirmed) | CustomError `code: string \| number` field — confirms test assertion `toMatchObject({ code: 'PERMISSION_DATA_CORRUPTED' })` matches errorInternal output |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.interface.ts | 2026-05-06 | no (re-confirmed) | CreateCompPermissionInterface + UpdateCompPermissionInterface use `permission: object` — Record<string, unknown> assignable; L2 z.unknown swap is type-safe |
| happywork-backend/src/api/v1/admin/compPermission/compPermission.controller.ts | 2026-05-06 | yes | CR-H1 — getCompPermissionByUuidController สร้าง cache ตัวเดียว ส่งให้ service + reuse; CR-M2 — updateCompPermissionByUuidController ส่ง compPermission.companyId เข้า service |
| happywork-backend/src/modules/v1/admin/compPermission/compPermission.service.ts | 2026-05-06 | yes | CR-H1 — getCompPermissionByUuidService รับ optional cache?: ResolverCache param; CR-M1 — desiredKeys.length===0 early-return ก่อน resolver call; CR-M2 — updateCompPermissionByUuidService รับ optional existingCompanyId param skip internal lookup |
| happywork-backend/src/modules/v1/externalAuth/permissions.service.ts | 2026-05-06 | yes | CR-L1 — เพิ่ม TODO comment ที่ logger.error level:'info' workaround (defer to Phase 5 logger refactor) |
| happywork-backend/src/schemas/compPermission.ts | 2026-05-06 | yes | CR-L2 — z.record(z.string(), z.any()) → z.unknown() ทั้ง 6 spot (lines 14,15,37,38,137,138) |
| happywork-backend/tests/unit/externalAuth/permissions.service.test.ts | 2026-05-06 | yes | CR-H2 — เพิ่ม jest.mock resolver + 2 new test cases (PERMISSION_DATA_NOT_FOUND graceful + PERMISSION_DATA_CORRUPTED propagate); 5/5 pass |
| document/requirement/feature-management/checklist.md (Phase 4 CR Fix Wave) | 2026-05-06 | yes | Appended new section listing 6 fixed + 1 deferred (L3) with verification log + files modified |

---

## Progress / Checklist

_(ลิงก์ไปที่ feature task-list — เช่น `document/requirement/feature-management/checklist.md` — เมื่อมี active task)_

---

## Recent Updates

| Date | Update |
|------|--------|
| 2026-05-05 | Developer agent created — merged from backend + frontend agents (per workspace refactor). Closed tasks below ลอกมาจาก archived backend/current-task.md + frontend/current-task.md เพื่อ preserve history |

---

## Closed Tasks (this feature)

> History ของ task ที่ปิดแล้วใน feature-management Phase 2 — รวมทั้ง backend + frontend tracks

### Backend Track — Closed Tasks (feature-management)

#### Status Value Migration — `'archived'` → `'inactive'` (Phase 2 modules)

**Status:** done (2026-05-05)
**Reason:** User สั่งให้ Phase 2 รับเฉพาะ `'active'` + `'inactive'` (ไม่ใช้ `'archived'`)
**Files modified (4):**
- `src/modules/v2/sale-dashboard/feature/feature.repository.ts` — `softDeleteFeatureRepository`: `ARCHIVED` → `INACTIVE`
- `src/modules/v2/sale-dashboard/packageCrud/packageCrud.repository.ts` — `softDeletePackageRepository` + 2 comments
- `src/modules/v2/sale-dashboard/packageCrud/packageCrud.service.ts` — 3 comment lines
- `src/modules/v2/sale-dashboard/addon/addon.repository.ts` — `softDeleteAddonRepository`

**Verification:** DB schema = VARCHAR(25) no constraint, no migration needed; TS compile clean; prettier conformant.

#### Move-to-SaleDashboard Refactor — 3 modules (`v2/admin/` → `v2/sale-dashboard/`)

**Status:** done (2026-05-05)
**Reason:** Frontend = `happywork-sale-cms` ใช้ sale dashboard token — Phase 2 ต้องอยู่ใน sale-dashboard scope
**URL change:** `/api/v2/{features,packages,addons}` → `/api/v2/sale-dashboard/{features,packages,addons}`
**Files moved:** 6 folders (api/v2/admin/{feature,packageCrud,addon} + modules/v2/admin/...) + 12 file edits + middleware swap (`validateAccessToken` → `validateSaleDashboardAccessToken` ใน 20 endpoints)

#### URL Fix — Drop `/admin` segment

**Status:** done (2026-05-05)
**Files (3):** routes ของ feature, packageCrud, addon — `basePath` ลบ `/admin` segment + 20 inline comment URL refs
**Verification:** TS clean; prettier conformant.

#### Wave B4 — Package UUID Update (`PATCH /:packageUuid/identifier`)

**Status:** done (2026-05-05)
**Files (5):** packageCrud.{interface, repository, service, controller, routes}
**Key addition:** Transactional update of `comp_packages.uuid` + `comp_companies.selected_package_uuid` (denormalized cache sync) + audit trigger snapshot

#### Wave B3 — Package CRUD + Addon module

**Status:** done (2026-05-05)
**Files created (12):** `packageCrud.{interface,repository,service,adapter,controller,routes}` + `addon.{interface,repository,service,adapter,controller,routes}`
**Mounted at:** `/api/v2/admin/packages` + `/api/v2/admin/addons` (later moved to sale-dashboard)

#### Wave B1 — Helper + Middleware

**Status:** done (2026-05-05)
**Files (2):**
- `src/constant/compPermission.ts` — added `getAvailableMenuKeys()` helper
- `src/middlewares/requireSuperAdmin.middleware.ts` (new)

#### Phase 1 — Schema Foundation (FEAT-101..117)

**Status:** done (2026-05-04)
**Files created (15):** 8 migrations + 7 Objection.js models
- M1-M8: comp_features, comp_packages, comp_package_features (Cartesian seed Q1=C), comp_addons, comp_addon_features, comp_company_features (FK RESTRICT Q4=B), comp_company_addons (+migrate from JSONB), alter comp_companies.package_id FK
- Models 1-7 with relationMappings + Type interfaces + jsonAttributes for JSONB

**Deviations flagged:** comp_packages seed = 7 records (not 8 as in SSD); comp_addons seed = 9 records (HappyBeacon + 4 tier×2 pairs)

---

### Frontend Track — Closed Tasks (feature-management)

#### Wire Value Migration — `"archived"` → `"inactive"` (3 enums + 3 Redux types + JSDoc)

**Status:** done (2026-05-05)
**Files (9):** 3 enums (FeatureStatus, PackageStatusTypeEnum, AddonStatus) + 3 Redux type aliases + 3 service request files (JSDoc)
**Verification:** TS clean; prettier conformant; grep `'archived'` ใน Phase 2 = 0 hits; i18n parity 53/53, 103/103, 89/89

#### Status Filter Label Fix — "Archived" → "Inactive" / "ไม่ใช้งาน" (3 pages)

**Status:** done (2026-05-05)
**Approach:** Option B — rename TS enum identifier `ARCHIVED` → `INACTIVE` (string literal `"archived"` คงเดิม) + rename i18n key `packageManagement.status.archived` → `inactive`
**Files (14):** 3 enums + 3 list-views + 2 forms + 2 form utils + 1 status-option + 1 detail-view + 1 table + 2 i18n

#### Move-to-SaleDashboard URL Update (Phase 2 builders)

**Status:** done (2026-05-05)
**Files:** `src/store/services/sale-dashboard.ts` (lines 100-128, 11 builders)
**Change:** Re-added `/sale-dashboard/` segment + section comments updated to reference saleDashboardV2Router

#### URL Fix — Drop `/admin` segment from Phase 2 builders

**Status:** done (2026-05-05)
**Files:** `src/store/services/sale-dashboard.ts` (lines 100-125, 11 builders)

#### Q8 Fix — Addon `billingInterval` value rename

**Status:** done (2026-05-05)
**Files (8):** type, enum, form util, form Select, table + detail-view, en/th i18n
**Change:** `'monthly' | 'yearly' | 'one_time'` → `'month' | 'year'` (drop `'one_time'` literal — `null` carries semantic, mirrors Package pattern)

#### Wave F6 — Page C Addon Management (F2.32-F2.34)

**Status:** done (2026-05-05)
**Files (19):** addon-management section (enum, Yup util, table, form, dialogs, conditional max-quantity sub-form, features section + edit dialog) + 4 views + 4 App Router pages with `<Can resource="addons">` guards
**i18n:** `addonManagement.*` 89 keys (en + th parity)

#### Wave F5 — Page B Package Management (F2.29-F2.31) + master_packages RBAC

**Status:** done (2026-05-05)
**Files (20):** rbac.ts (`master_packages` resource), package-management section (enum, form util, table, form, dialogs, features section + edit, effective menus preview), 4 views (detail mounts `<PackageUuidEditDialog>` + `router.replace`), 4 App Router pages
**i18n:** `packageManagement.*` ~80 keys (en + th parity) + `common.permission_denied_*`

#### Wave B4-FE — Package UUID Update Dialog

**Status:** done (2026-05-05)
**Files:** `package-uuid-edit-dialog.tsx` component + integration ใน detail-view

#### Wave F4 — Page A Feature Management

**Status:** done (early 2026-05-05)

#### Wave F3 — 3 Redux slices

**Status:** done (early 2026-05-05)
**Slices:** `sale-dashboard-feature-management`, `sale-dashboard-package-management`, `sale-dashboard-addon-management`

#### Wave F2 — Reusable components

**Status:** done
**Files:** 4 reusable components (MultilingualTextField, MenuTreePreview, ฯลฯ)

#### Wave F1 — RBAC + paths + navigation + i18n setup

**Status:** done

---

## Notes for Future Sessions

- Files Read Memory: ตอนเริ่ม session ใหม่ ต้อง revalidate (อ่านไฟล์ใหม่ 1 ครั้ง) — external change อาจเกิดได้
- ถ้าจะทำ task ต่อใน Phase 3 ของ feature-management → อ่าน Closed Tasks เพื่อเห็น context ที่ทำไปแล้ว
- Backward compat: archived `backend/current-task.md` + `frontend/current-task.md` ที่ `.ai-memory/agent/_archive/` เก็บ raw history (ถ้าต้อง audit รายละเอียดที่นี่ไม่มี)
