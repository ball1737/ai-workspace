---
feature: type-safety-any-cleanup
type: feature-summary
created: 2026-04-29
updated: 2026-04-29
owner: SA (initial), Lead (ongoing updates)
status: implemented
---

# Feature Summary — Type Safety Any Cleanup

> สรุปการยกระดับ type safety ของ `mof-backend/src` โดยรอบนี้คงไว้เฉพาะ model type exports และตั้ง lint เป็น warning เพื่อไม่กระทบ behavior เดิม

---

## 1. Current State (สถานะปัจจุบัน)

- `mof-backend` ใช้ Express 5 + TypeScript strict + Objection.js/Knex + Zod v4
- Baseline ก่อนเริ่มงาน: `npm run lint -- --no-cache` ผ่าน และ `npx tsc --noEmit --pretty false` ผ่าน
- `mof-backend/src` มี TypeScript source 376 ไฟล์ และพบ explicit `any` 175 nodes ใน 66 ไฟล์
- Objection model มี 72 ไฟล์ / 73 classes และยังไม่มี class-specific exported model type alias
- Full Jest baseline ก่อน refactor มี existing failures 4 suites / 29 tests; ต้องแยกจาก regression ของงานนี้

## 1.1 Implementation Result (2026-04-29)

- Objection model 73 ไฟล์ / 74 exported classes มี exported `ModelObject<...>` type ครบ 74 ตัว (รวม `UserFavorite`)
- ถอย explicit `any` cleanup ออกตามคำขอล่าสุด เพื่อไม่กระทบ behavior เดิมที่ทำงานได้อยู่
- ไม่ใช้ shared helper `DbQueryRow` / `src/types/common.ts` แล้ว
- เปิด `@typescript-eslint/no-explicit-any` เป็น `warn`
- `npm run lint -- --no-cache` ผ่าน
- `npx tsc --noEmit --pretty false` ผ่าน
- ไม่ได้รัน full Jest ซ้ำหลังถอย cleanup; รอบก่อนหน้ามี baseline fail 4 suites / 29 tests

## 2. Planned Changes (สิ่งที่จะทำ)

- เพิ่ม exported type 1 ตัวต่อ Objection model class เช่น `SystemUserType`
- อนุญาต explicit `any` ใน production source ชั่วคราว
- ESLint `@typescript-eslint/no-explicit-any` เป็น warning เพื่อใช้ติดตาม debt ต่อ

## 3. ⚠️ Files Reviewed (ไฟล์ที่ SA อ่านครบแล้ว)

### Folders covered

- `mof-backend/src/`

### Files read

| Path                                                                                                     | Read date  | Note                                              |
| -------------------------------------------------------------------------------------------------------- | ---------- | ------------------------------------------------- |
| `mof-backend/src/adapters/email/index.ts`                                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/email/smtp.adapter.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/postgresql/index.ts`                                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/postgresql/knex-postgresql.adapter.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/postgresql/postgresql.adapter.interface.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/redis/index.ts`                                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/redis/ioredis.adapter.ts`                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/redis/redis.adapter.interface.ts`                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/storage/index.ts`                                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/storage/s3-storage.adapter.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/adapters/storage/storage.adapter.interface.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/index.routes.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/activityLog/activityLog.controller.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/activityLog/activityLog.routes.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/auth/auth.controller.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/auth/auth.routes.ts`                                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/calendarEvent/calendarEvent.controller.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/calendarEvent/calendarEvent.routes.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/documentLibrary/documentLibrary.controller.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/documentLibrary/documentLibrary.routes.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/expensePositionRate/expensePositionRate.controller.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/expensePositionRate/expensePositionRate.routes.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/fiscalYear/fiscalYear.controller.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/fiscalYear/fiscalYear.routes.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/index.routes.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionCalendar/inspectionCalendar.controller.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionCalendar/inspectionCalendar.routes.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionInvitation/inspectionInvitation.controller.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionInvitation/inspectionInvitation.routes.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionPlan/inspectionPlan.controller.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionPlan/inspectionPlan.form.controller.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionPlan/inspectionPlan.routes.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionResult/inspectionResult.controller.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionResult/inspectionResult.routes.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZone/inspectionZone.controller.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZone/inspectionZone.routes.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneAssignment/inspectionZoneAssignment.controller.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneAssignment/inspectionZoneAssignment.routes.ts`                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.controller.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.routes.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.controller.ts`         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.routes.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneRegion/inspectionZoneRegion.controller.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspectionZoneRegion/inspectionZoneRegion.routes.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspector/inspector.controller.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/inspector/inspector.routes.ts`                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/loanAgreement/loanAgreement.controller.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/loanAgreement/loanAgreement.routes.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/orgPosition/orgPosition.controller.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/orgPosition/orgPosition.routes.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/orgUnit/orgUnit.controller.ts`                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/orgUnit/orgUnit.routes.ts`                                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/personRegistry/personRegistry.controller.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/personRegistry/personRegistry.routes.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/province/province.controller.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/province/province.routes.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/provinceContact/provinceContact.controller.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/provinceContact/provinceContact.routes.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/provinceProfile/provinceProfile.controller.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/provinceProfile/provinceProfile.routes.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/region/region.controller.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/region/region.routes.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/role/role.controller.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/role/role.routes.ts`                                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/systemUser/systemUser.controller.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/systemUser/systemUser.routes.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/task/task.controller.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/task/task.routes.ts`                                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/travelApproval/travelApproval.controller.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/travelApproval/travelApproval.routes.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/travelReport/travelReport.controller.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/travelReport/travelReport.routes.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/watermarkTemplate/watermarkTemplate.controller.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/api/v1/watermarkTemplate/watermarkTemplate.routes.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/app.ts`                                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/database.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/dotenv.ts`                                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/environment.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/index.ts`                                                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/redis.ts`                                                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/s3.ts`                                                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/sentry.ts`                                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/server.ts`                                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/service.ts`                                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/configs/swagger.ts`                                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/constants/country.ts`                                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/constants/errorCode.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/constants/general.ts`                                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/constants/log.ts`                                                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/index.ts`                                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/base.model.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/_template.model.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/calendarEvent.model.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/documentLibraryFile.model.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/documentLibraryNode.model.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/expensePositionRate.model.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/fiscalYear.model.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/fiscalYearCarryForwardDocument.model.ts`                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/fiscalYearTenPercentExpenseItem.model.ts`               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEvent.model.ts`                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEventAttachment.model.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEventMember.model.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEventProvince.model.ts`               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEventSchedule.model.ts`               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionCalendarEventType.model.ts`                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionInvitation.model.ts`                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionInvitationInvitee.model.ts`                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlan.model.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanFormDispatch.model.ts`                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanFormRecipient.model.ts`                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanQuestion.model.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanRecipientRevision.model.ts`               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanRecipientSectionResponse.model.ts`        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanRevisionHistory.model.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanSection.model.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanSignedDocument.model.ts`                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionPlanTableData.model.ts`                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionResult.model.ts`                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionResultAttachment.model.ts`                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionResultRevisionHistory.model.ts`               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionResultSignedDocument.model.ts`                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneAssignment.model.ts`                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneCentralGroup.model.ts`                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneCentralGroupUnit.model.ts`                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneComplaintGroup.model.ts`                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneComplaintGroupUnit.model.ts`              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneRegion.model.ts`                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneRegionProvince.model.ts`                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneSignedDocumentBatch.model.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneSignedDocumentFile.model.ts`              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/inspectionZoneSignedDocumentRecipient.model.ts`         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/loanAgreement.model.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/loanAgreementReturn.model.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/loanAgreementReturnAttachment.model.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/myTask.model.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/orgPosition.model.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/orgUnit.model.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/personRegistry.model.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/province.model.ts`                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/provinceContact.model.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/provinceProfile.model.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/provinceProfileAttachment.model.ts`                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/region.model.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/role.model.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/rolePermission.model.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/systemUser.model.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/systemUserRole.model.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelApprovalExpense.model.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelApprovalExpenseRow.model.ts`                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelApprovalExpenseSection.model.ts`                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelApprovalVehicleRequest.model.ts`                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReport.model.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportAttachment.model.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportExpenseForm.model.ts`                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportExpenseFormRow.model.ts`                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportPaymentRow.model.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportReceiptCertificate.model.ts`                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/travelReportSignedDocument.model.ts`                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/data/watermarkTemplate.model.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/log/logsAccessCredential.model.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/log/logsActivity.model.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/postgresql/models/log/logsAuthLogin.model.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/index.ts`                                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/_clear.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/_delete.ts`                                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/_lists.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/_read.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/_write.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/hash.ts`                                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/database/redis/models/index.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/doc/openapi.ts`                                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/auth.middleware.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/cors.middleware.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/errorHandlers.middleware.ts`                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/faviconHandler.middleware.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/logResponse.middleware.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/rateLimiter.middleware.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/requestIp.middleware.ts`                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/resolveDispatchRecipient.middleware.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/resolveInspectionResultByToken.middleware.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/responseHandler.middleware.ts`                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/sentryUser.middleware.ts`                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/swaggerAuth.middleware.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/uploadFile.middleware.ts`                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/uploadFiles.middleware.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/userAgent.middleware.ts`                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/validateData.middleware.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/middlewares/validator.middleware.ts`                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/middleware/rateLimiter/rateLimiter.interface.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/middleware/rateLimiter/rateLimiter.repository.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/activityLog/activityLog.export.service.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/activityLog/activityLog.interface.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/activityLog/activityLog.repository.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/activityLog/activityLog.service.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/auth/auth.interface.ts`                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/auth/auth.mailer.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/auth/auth.repository.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/auth/auth.service.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/auth/auth.thaid.service.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/calendarEvent/calendarEvent.export.service.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/calendarEvent/calendarEvent.import.service.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/calendarEvent/calendarEvent.interface.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/calendarEvent/calendarEvent.repository.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/calendarEvent/calendarEvent.service.ts`                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/documentLibrary/documentLibrary.interface.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/documentLibrary/documentLibrary.repository.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/documentLibrary/documentLibrary.service.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/expensePositionRate/expensePositionRate.export.service.ts`                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/expensePositionRate/expensePositionRate.import.service.ts`                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/expensePositionRate/expensePositionRate.interface.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/expensePositionRate/expensePositionRate.repository.ts`                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/expensePositionRate/expensePositionRate.service.ts`                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/fiscalYear/fiscalYear.export.service.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/fiscalYear/fiscalYear.import.service.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/fiscalYear/fiscalYear.interface.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/fiscalYear/fiscalYear.repository.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/fiscalYear/fiscalYear.service.ts`                                            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/importExcel/importExcel.helper.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionCalendar/inspectionCalendar.export.service.ts`                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionCalendar/inspectionCalendar.interface.ts`                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionCalendar/inspectionCalendar.mailer.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionCalendar/inspectionCalendar.repository.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionCalendar/inspectionCalendar.service.ts`                            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionInvitation/inspectionInvitation.interface.ts`                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionInvitation/inspectionInvitation.repository.ts`                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionInvitation/inspectionInvitation.service.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.export.service.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.form.service.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.interface.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.mailer.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.recipient.repository.ts`                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.recipient.service.ts`                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.repository.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionPlan/inspectionPlan.service.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionResult/inspectionResult.interface.ts`                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionResult/inspectionResult.mailer.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionResult/inspectionResult.repository.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionResult/inspectionResult.service.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZone/inspectionZone.export.service.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneAssignment/inspectionZoneAssignment.interface.ts`              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneAssignment/inspectionZoneAssignment.mailer.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneAssignment/inspectionZoneAssignment.report.service.ts`         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneAssignment/inspectionZoneAssignment.repository.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneAssignment/inspectionZoneAssignment.service.ts`                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.import.service.ts`     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.interface.ts`          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.repository.ts`         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.service.ts`            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.import.service.ts` | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.interface.ts`      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.repository.ts`     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.service.ts`        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneRegion/inspectionZoneRegion.import.service.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneRegion/inspectionZoneRegion.interface.ts`                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneRegion/inspectionZoneRegion.repository.ts`                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspectionZoneRegion/inspectionZoneRegion.service.ts`                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspector/inspector.interface.ts`                                            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspector/inspector.repository.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/inspector/inspector.service.ts`                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/loanAgreement/loanAgreement.interface.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/loanAgreement/loanAgreement.repository.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/loanAgreement/loanAgreement.service.ts`                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgPosition/orgPosition.interface.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgPosition/orgPosition.repository.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgPosition/orgPosition.service.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgScope/orgScope.interface.ts`                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgScope/orgScope.repository.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgScope/orgScope.service.ts`                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgUnit/orgUnit.export.service.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgUnit/orgUnit.import.service.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgUnit/orgUnit.interface.ts`                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgUnit/orgUnit.repository.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/orgUnit/orgUnit.service.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/personRegistry/personRegistry.export.service.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/personRegistry/personRegistry.import.service.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/personRegistry/personRegistry.interface.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/personRegistry/personRegistry.repository.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/personRegistry/personRegistry.service.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/province/province.interface.ts`                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/province/province.repository.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/province/province.service.ts`                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceContact/provinceContact.export.service.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceContact/provinceContact.import.service.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceContact/provinceContact.interface.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceContact/provinceContact.repository.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceContact/provinceContact.service.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceProfile/provinceProfile.import.service.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceProfile/provinceProfile.interface.ts`                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceProfile/provinceProfile.repository.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/provinceProfile/provinceProfile.service.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/region/region.interface.ts`                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/region/region.repository.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/region/region.service.ts`                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/role/role.interface.ts`                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/role/role.repository.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/role/role.service.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/systemUser/systemUser.export.service.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/systemUser/systemUser.interface.ts`                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/systemUser/systemUser.repository.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/systemUser/systemUser.service.ts`                                            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/task/task.interface.ts`                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/task/task.repository.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/task/task.service.ts`                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelApproval/travelApproval.interface.ts`                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelApproval/travelApproval.repository.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelApproval/travelApproval.service.ts`                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelReport/travelReport.interface.ts`                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelReport/travelReport.repository.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/travelReport/travelReport.service.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/watermarkTemplate/watermarkTemplate.interface.ts`                            | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/watermarkTemplate/watermarkTemplate.repository.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/modules/v1/watermarkTemplate/watermarkTemplate.service.ts`                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/helpers/defineRequestSchema.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/activityLog/activityLog.schema.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/auth/auth.schema.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/calendarEvent/calendarEvent.schema.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/documentLibrary/documentLibrary.schema.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/expensePositionRate/expensePositionRate.schema.ts`                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/fiscalYear/fiscalYear.schema.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionCalendar/inspectionCalendar.schema.ts`                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionInvitation/inspectionInvitation.schema.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionPlan/inspectionPlan.schema.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionPlan/inspectionPlanForm.schema.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionResult/inspectionResult.schema.ts`                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionZone/inspectionZone.schema.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionZoneAssignment/inspectionZoneAssignment.schema.ts`                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionZoneCentralGroup/inspectionZoneCentralGroup.schema.ts`             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionZoneComplaintGroup/inspectionZoneComplaintGroup.schema.ts`         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspectionZoneRegion/inspectionZoneRegion.schema.ts`                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/inspector/inspector.schema.ts`                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/loanAgreement/loanAgreement.schema.ts`                                       | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/orgPosition/orgPosition.schema.ts`                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/orgUnit/orgUnit.schema.ts`                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/personRegistry/personRegistry.schema.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/province/province.schema.ts`                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/provinceContact/provinceContact.schema.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/provinceProfile/provinceProfile.schema.ts`                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/region/region.schema.ts`                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/role/role.schema.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/systemUser/systemUser.schema.ts`                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/task/task.schema.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/travelApproval/travelApproval.schema.ts`                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/travelReport/travelReport.schema.ts`                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/schemas/v1/watermarkTemplate/watermarkTemplate.schema.ts`                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/server.ts`                                                                              | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/templates/email/helpers.ts`                                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/types/archiver.d.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/typings/express/index.d.ts`                                                             | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/typings/express/validatedRequest.ts`                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/typings/ts-type.ts`                                                                     | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/HttpException.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/__examples__/error-type-safety.example.ts`                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/actorContext.ts`                                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/crypto.ts`                                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/dataEncryption.ts`                                                                | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/deviceInfo.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/diagnostic.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/error.ts`                                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/errorHandler.ts`                                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/file.ts`                                                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/healthMonitor.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/helper.ts`                                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/helpers/getValidated.ts`                                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/httpClient.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/jwt.ts`                                                                           | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/orgScopeQuery.ts`                                                                 | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/pdfWatermark.ts`                                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/registerRouterOpenApi.ts`                                                         | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/storagePath.ts`                                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/tempFileCleanup.ts`                                                               | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/timezone.ts`                                                                      | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/typeGuards.ts`                                                                    | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/userId.ts`                                                                        | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/uuid.ts`                                                                          | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/validateEnv.ts`                                                                   | 2026-04-29 | Static inventory + targeted AST/type-safety audit |
| `mof-backend/src/utils/zodToOpenApi.ts`                                                                  | 2026-04-29 | Static inventory + targeted AST/type-safety audit |

### Files explicitly excluded

| Path                                              | Reason for exclusion                                              |
| ------------------------------------------------- | ----------------------------------------------------------------- |
| `mof-backend/tests/`                              | Out of scope for v1 per user decision                             |
| `mof-backend/scripts/`                            | Out of scope for v1 per user decision                             |
| `mof-backend/dist/`                               | Build output / generated artifact                                 |
| `mof-backend/node_modules/`                       | Dependency folder                                                 |
| `mof-backend/src/database/postgresql/migrations/` | Migration files excluded by workspace rules unless migration task |

## 4. Architecture Decisions

| Decision                             | Alternatives considered                                 | Reason chosen                                                                 |
| ------------------------------------ | ------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Export one `XxxType` per model class | Insert/Patch aliases, manual full interfaces            | User clarified CRUD params should be function-owned; one model type is enough |
| Use shared JSON/plain object aliases | Continue `Record<string, any>`, module-local duplicates | Reduces unsafe object surfaces without changing runtime behavior              |
| Keep API contracts unchanged         | Rename response fields / introduce new wrappers         | This is a type-safety refactor, not an API change                             |

## 5. Risks & Mitigations

| Risk                                             | Likelihood | Impact                                     | Mitigation                                                                       |
| ------------------------------------------------ | ---------- | ------------------------------------------ | -------------------------------------------------------------------------------- |
| Knex raw/join rows lose exact typing             | Medium     | Compile errors or unsafe casts move around | Add local row interfaces near query mappers                                      |
| Express route overloads reject typed controllers | Medium     | Route compile failures                     | Use typed handler helper returning `RequestHandler` instead of route-level casts |
| Existing tests fail unrelated to refactor        | High       | Hard to validate final state               | Record baseline failures and rely on lint/typecheck plus targeted tests          |

## 6. Progress Log

| Date       | Event                  | Agent | Note                                           |
| ---------- | ---------------------- | ----- | ---------------------------------------------- |
| 2026-04-29 | Started implementation | Codex | Long-term workflow docs and memory initialized |

## 7. Final Outcome (กรอกเมื่อจบ feature)

- Delivered:
- Deviations from plan:
