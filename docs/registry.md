# Endpoint review registry

Batch-review index for SME Platform API endpoints under skip-access-control
investigation. The established request baseline and pack template live in
`.agents/skills/access-control-impact/`.

Add a row when a pack is written. Leave the table empty until then.

| Endpoint | Product / feature | Callers | UI/BFF gate | Mapping status | Verdict | Residual risk | Evidence pack | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| GET `/businesses/{id}/entitlements-balance-summary-report` | Leave Balance / `entitlements-balance-summary-report` | Reporting Web, report-pack, Payroll Activity/Register fan-outs | Partially gated: `reportsLeaveBalance` | Missing Protected API map | Insufficient evidence | Flag removal would deny all callers until an exact map key is added | [pack](packs/entitlements-balance-summary-report.md) | Pending |
| GET `/businesses/{id}/entitlements-balance-detail-report` | Leave Balance Detail / `entitlements-balance-detail-report` | Reporting Web, BFF report-pack path | Partially gated: `reportsLeaveBalanceDetail` | Missing Protected API map | Insufficient evidence | Flag removal would deny all callers; URL/header business-ID relationship is unverified | [pack](packs/entitlements-balance-detail-report.md) | Pending |
| GET `/businesses/{id}/payroll-verification-report` | Payroll Verification / `payroll-verification-report` | Reporting Web, Reporting BFF shared export path | Partially gated: `reportsPayrollVerification` | Matched to `ReportsPayroll` | Insufficient evidence | Shared export path and full SME Web caller inventory remain unverified | [pack](packs/payroll-verification-report.md) | Pending |
| GET `/businesses/{id}/payroll-activity-report` | Payroll Activity / `payroll-activity-report` | Reporting Web, Reporting BFF shared business-report path | Partially gated: `reportsPayrollActivity` | Primary map matched; dependent summary map missing | User-visible impact | The summary fan-out will fail for entitled users until its exact map key is added | [pack](packs/payroll-activity-report.md) | Pending |
| POST `/businesses/{id}/exception-report-data-auditor/tax-code-cash` | Data Auditor Tax Code Cash / `data-auditor` | Reporting Web and dedicated BFF route; shared business-report path unverified | Partially gated: list only; deep-link and BFF paths bypass | Missing `write-data-auditor` map; read map and UI entitlements also conflict | User-visible impact | Flag removal denies confirmed callers until write map and entitlement policy are aligned | [pack](packs/tax-code-cash-data-auditor-report.md) | Pending |
| POST `/businesses/{id}/exception-report-data-auditor/prepaid-transactions` | Data Auditor Prepaid Transactions / `data-auditor` | Reporting Web and dedicated BFF route; shared business-report path unverified | Partially gated: list only; deep-link and BFF paths bypass | Missing `write-data-auditor` map; read map and UI entitlements also conflict | User-visible impact | Flag removal denies confirmed callers until write map and entitlement policy are aligned | [pack](packs/prepaid-transactions-data-auditor-report.md) | Pending |
