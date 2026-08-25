# GET /businesses/{business-id}/payroll-activity-report

## Proposed authorization predicate
Verified user identity has `ReportsPayroll` for the requested business/company-file and may read the payroll-activity report.

## Caller inventory
- Reporting Web’s AU Payroll Activity route requires `reportsPayrollActivity`: `reporting-web/src/components/Reports/reportDetailsEnums.js` (`PAYROLL_ACTIVITY`).
- Reporting Backends exposes `GET /bff/payrollActivity/:businessId`: `reporting-backends/bff/src/payrollActivity/payrollActivityRouter.js`.
- The service requests this report and also requests pay-items, STP status, and Leave Balance Summary for the same business: `reporting-backends/bff/src/payrollActivity/payrollActivityService.js`.
- `reporting-backends/bff/src/businessReport/businessReportResources.js` wires `PAYROLL_ACTIVITY` to the same service; no separate feature gate was located for this shared path.
- The Protected API controller is authenticated and exposes `GET Report/Payroll/PayrollActivity`: `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Reporting/Payroll/PayrollActivity/PayrollActivityController.cs`.

## Tenant identifier propagation
The BFF callback derives `businessId` from the route parameter (`reporting-backends/bff/src/common/routerCallback.js`). Payroll Activity and all its fan-out requests interpolate that same context value; the Platform adapter forwards route capture group 1 as the Protected API company-file id (`sme-platform-api/src/feature/reports/payroll/payroll-activity/adapter-config.lua`).

## UI/BFF eligibility-gate check
| Check | Observed value | Source evidence | Classification |
| --- | --- | --- | --- |
| Business context used for `current-user` and roles | `businessId` is used for both calls | `reporting-backends/bff/src/enabledFeatures/enabledFeaturesService.js` | Matched |
| User role IDs mapped to business roles and entitlements | Role IDs join against business roles and collect `SystemEntitlement` | `reporting-backends/bff/src/business/transformers/buildUserEntitlements.js` | Matched |
| Entitlement-to-`enabledFeatures` calculation and filters | `ReportsPayroll` yields `reportsPayrollActivity` and is filtered by region/subscription | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`, `enabledFeaturesTransformer.js` | Alias |
| Web widget, SPA route, or BFF route gate | AU route requires `reportsPayrollActivity`, but the BFF callback invokes the service without an `enabledFeatures` check and the shared business-report resource has no located equivalent gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-web/src/index.js`, `reporting-backends/bff/src/common/routerCallback.js`, `businessReportResources.js` | Partially gated |
| Endpoint feature alignment | Platform feature is a PermissionMap key mapped to `ReportsPayroll` | `sme-platform-api/src/feature/reports/payroll/payroll-activity/adapter-config.lua`; `accountright-protected-api/.../PermissionMap.cs` | Matched |

**UI/BFF classification:** `Partially gated`

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | `payroll-activity-report`; currently bypasses access control | `sme-platform-api/src/feature/reports/payroll/payroll-activity/adapter-config.lua` | Matched |
| Protected API `PermissionMap.cs` entitlement | `payroll-activity-report` → `ReportsPayroll` | `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/PermissionMap.cs` | Matched |
| BFF feature calculation | `ReportsPayroll` → `reportsPayrollActivity` | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js` | Alias |
| Web UI/route gate | AU Payroll Activity SPA route requires `reportsPayrollActivity`; the shared BFF business-report path has no located equivalent gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-backends/bff/src/businessReport/businessReportResources.js` | Partially gated |

## Findings
1. The report’s BFF service also calls `entitlements-balance-summary-report`. Its current missing PermissionMap mapping is an indirect rollout blocker: the activity report’s fan-out request will fail once that leave-balance route is protected (`reporting-backends/bff/src/payrollActivity/payrollActivityService.js`).
2. The Reporting Web route is gated, but the BFF route and shared business-report resource do not have a located `enabledFeatures` check. The new Platform enforcement is therefore required for these callers.
3. Targeted SME Web evidence shows dashboard deep-links only; a full-tree endpoint-path scan timed out, so absence of another caller is unverified.

## Verdict
`User-visible impact`

## Residual risk
Do not remove the Payroll Activity flag in the same batch as an unmapped Leave Balance Summary route. First add the exact `entitlements-balance-summary-report` → `ReportsPayroll` mapping and retest the aggregate; otherwise normal entitled Payroll Activity users receive a failed report. Complete the SME Web caller inventory and verify the shared business-report path before broader rollout.
