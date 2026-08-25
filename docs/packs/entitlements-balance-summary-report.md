# GET /businesses/{business-id}/entitlements-balance-summary-report

## Proposed authorization predicate
Verified user identity has `ReportsPayroll` for the requested business/company-file and may read the leave-balance summary report.

## Caller inventory
- Reporting Web’s Leave Balance Summary report is an AU route gated by `reportsLeaveBalance`: `reporting-web/src/components/Reports/reportDetailsEnums.js` (`LEAVE_BALANCE_SUMMARY`).
- Its BFF route is `GET /bff/leaveBalanceSummary/:businessId`: `reporting-backends/bff/src/leaveBalanceSummary/leaveBalanceSummaryRouter.js`.
- The BFF service calls `/businesses/${businessId}/entitlements-balance-summary-report`: `reporting-backends/bff/src/leaveBalanceSummary/leaveBalanceSummaryService.js`.
- Payroll Activity and Payroll Register also fan out to this same BFF service URL: `reporting-backends/bff/src/payrollActivity/payrollActivityService.js`, `reporting-backends/bff/src/payrollRegister/payrollRegisterService.js`.
- `reporting-backends/bff/src/businessReport/businessReportResources.js` also wires `LEAVE_BALANCE_SUMMARY` to this service for report-pack/PDF processing.
- SME Web is deep-link only; SME Web BFF has no matching caller: `sme-web/src/modules/dashboard/types/ReportsDetail.js`, `DashboardPayrollReportsSelectors.js`; full `sme-web-bff/src` search found no matching terms.

## Tenant identifier propagation
`getRouterCallback` takes `request.params.businessId` into the BFF context (`reporting-backends/bff/src/common/routerCallback.js`), and the service places that same value in the SME Platform API URL. The Platform route gets regex group 1 and targets `/{businessId}/Report/Payroll/EntitlementsBalanceSummary` (`sme-platform-api/src/feature/reports/payroll/entitlements-balance-summary/adapter-config.lua`).

## UI/BFF eligibility-gate check
| Check | Observed value | Source evidence | Classification |
| --- | --- | --- | --- |
| Business context used for `current-user` and roles | `businessId` is used for both calls | `reporting-backends/bff/src/enabledFeatures/enabledFeaturesService.js` | Matched |
| User role IDs mapped to business roles and entitlements | `RoleIds` are matched to returned roles and their `SystemEntitlement` values | `reporting-backends/bff/src/business/transformers/buildUserEntitlements.js` | Matched |
| Entitlement-to-`enabledFeatures` calculation and filters | `ReportsPayroll` produces `reportsLeaveBalance`; region/subscription filters follow | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`, `enabledFeaturesTransformer.js` | Alias |
| Web widget, SPA route, or BFF route gate | AU report definition requires `reportsLeaveBalance`, but the BFF data route, report-pack path, Payroll Activity, and Payroll Register callers have no located equivalent feature gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-web/src/index.js`, `reporting-backends/bff/src/common/routerCallback.js`, `businessReportResources.js`, `payrollActivityService.js`, `payrollRegisterService.js` | Partially gated |
| Endpoint feature alignment | Platform feature is `entitlements-balance-summary-report`, but no Protected API map key was found | `sme-platform-api/src/feature/reports/payroll/entitlements-balance-summary/adapter-config.lua`; searched `PermissionMap.cs` for `entitlements`, `balance`, and `leave` | Missing |

**UI/BFF classification:** `Partially gated`

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | `entitlements-balance-summary-report`; currently bypasses access control | `sme-platform-api/src/feature/reports/payroll/entitlements-balance-summary/adapter-config.lua` | Matched |
| Protected API `PermissionMap.cs` entitlement | No exact feature key found; lookup returns an empty list for missing keys | `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/PermissionMap.cs` (`LookupEntitlement`) | Missing |
| BFF feature calculation | `ReportsPayroll` → `reportsLeaveBalance` | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js` | Alias |
| Web UI/route gate | AU Leave Balance route requires `reportsLeaveBalance`; BFF shared and fan-out callers have no located equivalent feature gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-backends/bff/src/businessReport/businessReportResources.js` | Partially gated |

## Findings
1. Removing `skip_access_control` activates `permission-validator`, which checks `CanPerformAction` with `read:entitlements-balance-summary-report` (`sme-platform-api/src/route/route.lua`, `security/permission-verifier.lua`). The absent map key produces no required entitlement and therefore denies every user (`CheckPermissionService.cs`).
2. The Reporting Web gate does not cover the report-pack/PDF path or Payroll Activity and Payroll Register fan-outs. Downstream Platform enforcement is required for those callers once the mapping exists.

## Verdict
`Insufficient evidence`

## Residual risk
Do not deploy the flag removal until a reviewed Protected API map key ties this feature to `ReportsPayroll`, and positive/negative business-scoped authorization tests pass.
