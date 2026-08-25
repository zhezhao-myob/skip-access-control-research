# POST /businesses/{business-id}/exception-report-data-auditor/prepaid-transactions

> The requested `/businesses/exception-report/prepaid-transactions/{id}` form
> is the Lua metrics label and Data Auditor upstream path shape. The
> client-facing Platform ingress is the path in this heading.

## Proposed authorization predicate
Verified user identity has the entitlement mapped to `write-data-auditor` for the requested business/company-file and may invoke the Data Auditor prepaid-transactions report.

## Caller inventory
- SME Platform’s nginx ingress matches `POST /businesses/{businessId}/exception-report-data-auditor/prepaid-transactions`, invokes this Data Auditor Lua adapter, and forwards the first capture to the configured Data Auditor API host: `sme-platform-api/sites-enabled/prepaid-transactions.conf`; `sme-platform-api/src/feature/reports/data-auditor/prepaid-transactions-report/route.lua`, `adapter-config.lua`.
- Reporting Web loads the prepaid-transactions report through `GET /bff/prepaidTransactions/:businessId`: `reporting-web/src/integration/sagas/prepaidTransactionsIntegration.js`; `reporting-backends/bff/src/prepaidTransactions/prepaidTransactionsRouter.js`.
- The BFF service posts to that ingress path: `reporting-backends/bff/src/prepaidTransactions/prepaidTransactionsService.js`.
- The shared BFF business-report resource registers the same service, but does not supply its required `reportName`; its route to this Platform endpoint is therefore unverified and may be malformed: `reporting-backends/bff/src/businessReport/businessReportResources.js`; `prepaidTransactions/prepaidTransactionsService.js`; `businessReport/businessReportService.js`.
- SME Web contains a dashboard deep link for `prepaidTransactions`: `sme-web/src/modules/dashboard/types/ReportsDetail.js`. No SME Web BFF source caller was found.
- The Protected API report controller is authenticated, but exposes a different downstream `GET Report/Exception/PrepaidTransactions` route: `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Reporting/Exception/PrepaidTransaction/PrepaidTransactionsController.cs`.

## Tenant identifier propagation
Reporting Web derives `verifiedBusinessId` from the route and sends it in both the BFF URL and `x-myreports-businessid` header: `reporting-web/src/integration/sagas/prepaidTransactionsIntegration.js`; `reporting-web/src/integration/callBff.js`. The BFF callback copies the URL parameter into service context, then the service embeds it in the Platform URL: `reporting-backends/bff/src/common/routerCallback.js`; `prepaidTransactions/prepaidTransactionsService.js`. Nginx capture group 1 is that Platform URL business ID; Platform uses it for the permission check and appends it to the Data Auditor upstream URL: `sme-platform-api/sites-enabled/prepaid-transactions.conf`; `src/route/route.lua`; `src/feature/reports/data-auditor/prepaid-transactions-report/adapter-config.lua`.

## UI/BFF eligibility-gate check
| Check | Observed value | Source evidence | Classification |
| --- | --- | --- | --- |
| Business context used for `current-user` and roles | `businessId` drives current-user, role, subscription, and business lookups | `reporting-backends/bff/src/enabledFeatures/enabledFeaturesService.js` | Matched |
| User role IDs mapped to business roles and entitlements | User role IDs are joined to business roles and their `SystemEntitlement` values | `reporting-backends/bff/src/business/transformers/buildUserEntitlements.js` | Matched |
| Entitlement-to-`enabledFeatures` calculation and filters | `reportsPrepaidTransactions` is enabled by `ReportsBanking`, `ReportsPurchases`, or `ReportsSales`, then subject to subscription filtering | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`; `enabledFeatures/types/SubscriptionFeatureToFeatureMap.js` | Conflict |
| Web widget, SPA route, or BFF route gate | Exceptions-list visibility tests `reportsPrepaidTransactions`, but exception-only routes have no `features` field and `Switch` admits them before testing `exceptionEnabledFeatures`; BFF and shared business-report paths have no located equivalent feature check | `reporting-web/src/components/Reports/tabs/ExceptionsTab.js`; `reporting-web/src/components/Reports/reportDetailsEnums.js`; `reporting-web/src/Switch.js`; `reporting-backends/bff/src/common/routerCallback.js`; `businessReportResources.js` | Partially gated |
| Endpoint feature alignment | Platform uses POST permission `create:data-auditor`, normalized to `write-data-auditor`; no exact PermissionMap key was found. The read-style `data-auditor` key maps to `ReportsAccounts`, which also disagrees with the UI feature's source entitlements | `sme-platform-api/src/security/permission-verifier.lua`; `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/RoleQuery.cs`; `PermissionMap.cs` | Conflict |

**UI/BFF classification:** `Partially gated`

UI/BFF gating is user-impact evidence only. It does not replace downstream authorization for the verified user, effective entitlement, and requested business/company-file.

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | `data-auditor`; currently bypasses access control | `sme-platform-api/src/feature/reports/data-auditor/prepaid-transactions-report/adapter-config.lua` | Matched |
| Protected API `PermissionMap.cs` entitlement | POST creates `write-data-auditor`, but `PermissionMap.cs` contains only read-style `data-auditor` → `ReportsAccounts` | `sme-platform-api/src/security/permission-verifier.lua`; `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/RoleQuery.cs`; `PermissionMap.cs` | Missing |
| BFF feature calculation | Prepaid-transactions feature is produced by `ReportsBanking`, `ReportsPurchases`, and `ReportsSales`, subject to subscription filtering | `reporting-backends/bff/src/enabledFeatures/transformers/buildEntitlementFeatureMap.js`; `enabledFeatures/types/SubscriptionFeatureToFeatureMap.js` | Conflict |
| Web UI/route gate | Exceptions-list visibility is gated, but exception-only deep links bypass `exceptionEnabledFeatures`; shared business-report invocation is unverified | `reporting-web/src/components/Reports/tabs/ExceptionsTab.js`; `reporting-web/src/Switch.js`; `reporting-backends/bff/src/businessReport/businessReportResources.js` | Conflict |

## Findings
1. Removing the flag activates `CanPerformAction(create:data-auditor)` for this POST route. Protected API normalizes `create` to `write-data-auditor` (`sme-platform-api/src/security/permission-verifier.lua`; `accountright-protected-api/src/AccountRight.Protected.Api.Web/Features/Role/Permission/RoleQuery.cs`).
2. `PermissionMap.cs` has no `write-data-auditor` entry. The read-style `data-auditor` → `ReportsAccounts` entry does not authorize this POST action; removal will therefore deny confirmed callers until an exact write map is added.
3. If a write map is later added using `ReportsAccounts`, it still conflicts with the UI report feature, which is derived from `ReportsBanking`, `ReportsPurchases`, and `ReportsSales`, subject to subscription filters.
4. The BFF ingress is wired to this adapter by `sites-enabled/prepaid-transactions.conf`. Exceptions-list visibility is feature-gated, but deep-link routing bypasses the exception feature check; BFF callbacks do not re-check enabled features.

## Verdict
`User-visible impact`

## Residual risk
Add an exact `write-data-auditor` PermissionMap entry and align it with the UI report entitlement policy before removing the flag. Integration-test authorized, unauthorized, and cross-business requests, including a UI-feature-enabled user without the selected write entitlement, in both permission transport modes. Confirm BFF validation binds its `x-myreports-businessid` header to `:businessId`; verify the shared business-report path supplies `reportName` or remove it from the caller inventory.
