---
feature: type-safety-any-cleanup
type: testcase
created: 2026-04-29
updated: 2026-04-29
owner: QA
status: executed
---

# Test Cases — Type Safety Any Cleanup

## 1. Test Strategy

| Test Type    | Coverage                | Tool                       | Owner   |
| ------------ | ----------------------- | -------------------------- | ------- |
| Static audit | model type exports      | TypeScript AST script / rg | Backend |
| Lint         | project lint rules      | npm run lint               | Backend |
| Typecheck    | compiler compatibility  | tsc --noEmit               | Backend |
| Integration  | existing behavior smoke | Jest targeted/full         | QA      |

## 2. Unit Test Cases

### UT-1: Model type audit

- File: generated command / no committed test required
- Description: Audit reports one exported `ModelObject<...>` type per model class
- Cases:
  - [x] model files scanned > 0
  - [x] exported model type count equals exported model class count

## 3. Integration Test Cases

### IT-1: Existing backend test suite

- Description: Run current Jest tests with runtime OpenAPI generation disabled
- Expected: No new failures beyond known baseline

## 6. Test Execution Log

| Run            | Date       | Type           | Pass                  | Fail                | Skip | Note                                                                                         |
| -------------- | ---------- | -------------- | --------------------- | ------------------- | ---- | -------------------------------------------------------------------------------------------- |
| baseline       | 2026-04-29 | lint/typecheck | 2                     | 0                   | 0    | Passed before implementation                                                                 |
| baseline       | 2026-04-29 | jest full      | 24 suites             | 4 suites / 29 tests | 0    | Existing failures before refactor                                                            |
| implementation | 2026-04-29 | static audit   | 2                     | 0                   | 0    | Explicit any = 0; model classes 73 / exported model types 73                                 |
| implementation | 2026-04-29 | lint           | 1                     | 0                   | 0    | `npm run lint -- --no-cache` passed                                                          |
| implementation | 2026-04-29 | typecheck      | 1                     | 0                   | 0    | `npx tsc --noEmit --pretty false` passed                                                     |
| implementation | 2026-04-29 | targeted jest  | 2 suites / 61 tests   | 3 suites / 9 tests  | 0    | Unit utils passed; integration failures match known baseline areas                           |
| implementation | 2026-04-29 | full jest      | 24 suites / 356 tests | 4 suites / 29 tests | 0    | `DISABLE_RUNTIME_OPENAPI=true npm test -- --runInBand --silent`; same baseline failure count |
| rollback-scope | 2026-04-29 | model audit    | 1                     | 0                   | 0    | Model files 73 / classes 74 / model types 74; explicit any cleanup intentionally reverted    |
| rollback-scope | 2026-04-29 | lint           | 1                     | 0                   | 0    | `npm run lint -- --no-cache` passed with 175 `no-explicit-any` warnings                      |
| rollback-scope | 2026-04-29 | typecheck      | 1                     | 0                   | 0    | `npx tsc --noEmit --pretty false` passed                                                     |

## 7. Remaining Baseline Failures

- `tests/integration/18-inspection-zone-document-workflow.test.ts`: 5 failures around signed document payload expectations and task list count.
- `tests/integration/19-inspection-plan-recipient.test.ts`: 20 failures cascading from dispatch returning 500.
- `tests/integration/00-auth.test.ts`: 3 failures after seeded admin session is no longer valid later in the suite.
- `tests/integration/02-system-users.test.ts`: 1 failure where inactive user creation receives 409.

No dedicated `documentLibrary` or validator middleware test suite exists yet; coverage was approximated through document workflow, route validation usage, and utility tests.
