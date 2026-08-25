# GET /businesses/{business-id}/entitlements-balance-detail-report

## Proposed authorization predicate
Verified user identity has `ReportsPayroll` for the requested business/company-file and may read the leave-balance detail report.

## Caller inventory
- Reporting Web’s AU Leave Balance Detail route requires `reportsLeaveBalanceDetail`: `reporting-web/src/components/Reports/reportDetailsEnums.js` (`LEAVE_BALANCE_DETAIL`).
- The BFF exposes `GET /bff/leaveBalanceDetail/:businessId`: `reporting-backends/bff/src/leaveBalanceDetail/leaveBalanceDetailRouter.js`.
- It calls `/businesses/${businessId}/entitlements-balance-detail-report`: `reporting-backends/bff/src/leaveBalanceDetail/leaveBalanceDetailService.js`.
- `reporting-backends/bff/src/businessReport/businessReportResources.js` also wires `LEAVE_BALANCE_DETAIL` to this service for report-pack/PDF processing.
- SME Web is deep-link only; SME Web BFF has no matching caller: `sme-web/src/modules/dashboard/types/ReportsDetail.js`, `DashboardPayrollReportsSelectors.js`; full `sme-web-bff/src` search found no matching terms.

## Tenant identifier propagation
The BFF callback copies route `businessId` into its context (`reporting-backends/bff/src/common/routerCallback.js`); the service embeds it in the Platform API path. The Platform adapter uses regex group 1 as the Protected API company-file id and calls `/{businessId}/Report/Payroll/EntitlementsBalanceDetail` (`sme-platform-api/src/feature/reports/payroll/entitlements-balance-detail/adapter-config.lua`). Reporting Web supplies the URL `businessId` separately from `verifiedBusinessId`, which it supplies as the BFF header; whether those values can diverge must be verified (`reporting-web/src/integration/sagas/leaveBalanceDetailIntegration.js`, `integration/callBff.js`).

## UI/BFF eligibility-gate check
| Check | Observed value | Source evidence | Classification |
| --- | --- | --- | --- |
| Business context used for `current-user` and roles | `businessId` is used for both calls | `reporting-backends/bff/src/enabledFeatures/enabledFeaturesService.js` | Matched |
| User role IDs mapped to business roles and entitlements | Role IDs join against business roles and collect `SystemEntitlement` | `reporting-backends/bff/src/business/transformers/buildUserEntitlements.js` | Matched |
| Entitlement-to-`enabledFeatures` calculation and filters | `ReportsPayroll` produces `reportsLeaveBalanceDetail`; region/subscription filters follow | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`, `enabledFeaturesTransformer.js` | Alias |
| Web widget, SPA route, or BFF route gate | AU report definition requires `reportsLeaveBalanceDetail`, but the BFF data route and shared report-pack path have no located equivalent feature gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-web/src/index.js`, `reporting-backends/bff/src/common/routerCallback.js`, `businessReportResources.js` | Partially gated |
| Endpoint feature alignment | Platform feature has no located Protected API PermissionMap key | `sme-platform-api/src/feature/reports/payroll/entitlements-balance-detail/adapter-config.lua`; searched `PermissionMap.cs` for `entitlements`, `balance`, and `leave` | Missing |

**UI/BFF classification:** `Partially gated`

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | `entitlements-balance-detail-report`; currently bypasses access control | `sme-platform-api/src/feature/reports/payroll/entitlements-balance-detail/adapter-config.lua` | Matched |
| Protected API `PermissionMap.cs` entitlement | No exact feature key found; missing keys resolve to an empty entitlement list | `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/PermissionMap.cs` (`LookupEntitlement`) | Missing |
| BFF feature calculation | `ReportsPayroll` → `reportsLeaveBalanceDetail` | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js` | Alias |
| Web UI/route gate | AU Leave Balance Detail route requires `reportsLeaveBalanceDetail`; the BFF shared report-pack path has no located equivalent gate | `reporting-web/src/components/Reports/reportDetailsEnums.js`, `reporting-backends/bff/src/businessReport/businessReportResources.js` | Partially gated |

## Findings
1. Flag removal results in `CanPerformAction(read:entitlements-balance-detail-report)`. Without an exact PermissionMap mapping the downstream permission check denies all callers (`sme-platform-api/src/security/permission-verifier.lua`, `CheckPermissionService.cs`).
2. The Reporting Web gate does not cover the shared report-pack/PDF path. Downstream Platform enforcement is required for this caller once the mapping exists.
3. The detail saga’s URL business ID and `verifiedBusinessId` header originate separately. Their equality is unverified; this is a tenant-identifier propagation finding.

## Verdict
`Insufficient evidence`

## Residual risk
Add and review a Protected API mapping to `ReportsPayroll`; verify the two detail-saga business identifiers cannot diverge; then test both entitled and unentitled users against the selected business before removal.
