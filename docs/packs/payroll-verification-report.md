# GET /businesses/{business-id}/payroll-verification-report

## Proposed authorization predicate
Verified user identity has `ReportsPayroll` for the requested business/company-file and may read the payroll-verification report.

## Caller inventory
- Reporting Web’s AU Payroll Verification route requires `reportsPayrollVerification`: `reporting-web/src/components/Reports/reportDetailsEnums.js` (`PAYROLL_VERIFICATION`).
- Reporting Backends exposes `GET /bff/payrollVerification/:businessId`: `reporting-backends/bff/src/payrollVerification/payrollVerificationRouter.js`.
- Its shared service targets `/businesses/{businessId}/payroll-verification-report`: `reporting-backends/shared/src/payrollVerification/payrollVerificationService.js`; `ReportRouterName[PAYROLL_VERIFICATION]` resolves to that feature key in `reporting-backends/shared/src/common/commonEnums.js`.
- `reporting-backends/bff/src/exportReport/exportReportResources.js` also wires `PAYROLL_VERIFICATION` to that shared report service; no separate feature gate was located for that shared path.
- The Protected API controller is authenticated and exposes `GET Report/Payroll/PayrollVerification`: `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Reporting/Payroll/PayrollVerification/PayrollVerificationController.cs`.

## Tenant identifier propagation
The BFF copies `:businessId` into context (`reporting-backends/bff/src/common/routerCallback.js`). The Platform adapter obtains the first route capture and passes it as the Protected API company-file path segment (`sme-platform-api/src/feature/reports/payroll/payroll-verification/adapter-config.lua`).

## UI/BFF eligibility-gate check
| Check | Observed value | Source evidence | Classification |
| --- | --- | --- | --- |
| Business context used for `current-user` and roles | `businessId` is used for current-user, roles, subscription, and business calls | `reporting-backends/bff/src/enabledFeatures/enabledFeaturesService.js` | Matched |
| User role IDs mapped to business roles and entitlements | Role IDs join against business roles and collect `SystemEntitlement` | `reporting-backends/bff/src/business/transformers/buildUserEntitlements.js` | Matched |
| Entitlement-to-`enabledFeatures` calculation and filters | `ReportsPayroll` yields `reportsPayrollVerification` and is later filtered by region/subscription | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`, `enabledFeaturesTransformer.js` | Alias |
| Web widget, SPA route, or BFF route gate | AU route requires `reportsPayrollVerification`, but the BFF callback only builds context and calls the service; the shared export resource also reaches that service without a located feature gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-web/src/index.js`, `reporting-backends/bff/src/common/routerCallback.js`, `exportReportResources.js` | Partially gated |
| Endpoint feature alignment | Platform feature is a PermissionMap key mapped to `ReportsPayroll` | `sme-platform-api/src/feature/reports/payroll/payroll-verification/adapter-config.lua`; `accountright-protected-api/.../PermissionMap.cs` | Matched |

**UI/BFF classification:** `Partially gated`

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | `payroll-verification-report`; currently bypasses access control | `sme-platform-api/src/feature/reports/payroll/payroll-verification/adapter-config.lua` | Matched |
| Protected API `PermissionMap.cs` entitlement | `payroll-verification-report` → `ReportsPayroll` | `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/PermissionMap.cs` | Matched |
| BFF feature calculation | `ReportsPayroll` → `reportsPayrollVerification` | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js` | Alias |
| Web UI/route gate | AU Payroll Verification SPA route requires `reportsPayrollVerification`; shared BFF export path has no located equivalent gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-backends/bff/src/exportReport/exportReportResources.js` | Partially gated |

## Findings
1. Removing the flag activates a `CanPerformAction(read:payroll-verification-report)` check against the forwarded user token (`sme-platform-api/src/route/route.lua`, `security/permission-verifier.lua`).
2. The Reporting Web route is gated, but the BFF route itself does not re-check `enabledFeatures`, and a shared export-resource caller has no located equivalent gate. Downstream Platform enforcement is therefore required for that path.
3. Targeted SME Web evidence shows dashboard deep-links only; a full-tree endpoint-path scan timed out, so absence of another caller is unverified.

## Verdict
`Insufficient evidence`

## Residual risk
Complete the SME Web caller inventory and verify the shared export caller. Then release only with integration coverage for authorised, unauthorised, and wrong-business users, including both direct permission-verifier and permission-header deployment modes.
