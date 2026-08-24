# Endpoint review registry

Batch-review index for SME Platform API endpoints under skip-access-control
investigation. See [access-model.md](access-model.md) for the established
request baseline and per-endpoint analysis scope.

| Endpoint | Product / feature | Callers | UI/BFF gate | Mapping status | Verdict | Residual risk | Evidence pack | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `GET /businesses/{businessId}/entitlements-balance-summary-report` | Leave Balance Summary (`entitlements-balance-summary-report` / `reportsLeaveBalance`) | Reporting Web → Reporting BFF; SME Web deep-link only | `reportsLeaveBalance` (`ReportsPayroll`) | Pending PermissionMap reconciliation | | | | |
| `GET /businesses/{businessId}/entitlements-balance-detail-report` | Leave Balance Detail (`entitlements-balance-detail-report` / `reportsLeaveBalanceDetail`) | Reporting Web → Reporting BFF; SME Web deep-link only | `reportsLeaveBalanceDetail` (`ReportsPayroll`) | Pending PermissionMap reconciliation | | | | |
| `GET /businesses/{businessId}/payroll-verification-report` | Payroll Verification (`payroll-verification-report` / `reportsPayrollVerification`) | Reporting Web → Reporting BFF; SME Web deep-link only | `reportsPayrollVerification` (`ReportsPayroll`) | Pending PermissionMap reconciliation | | | | |
| `GET /businesses/{businessId}/payroll-activity-report` | Payroll Activity (`payroll-activity-report` / `reportsPayrollActivity`) | Reporting Web → Reporting BFF; SME Web deep-link only | `reportsPayrollActivity` (`ReportsPayroll`) | Pending PermissionMap reconciliation | | | | |
| `GET /businesses/{businessId}/timesheets-report` | Timesheets (`timesheets-report`) | Reporting Web → Reporting BFF | Not traced in reference entitlements doc | Pending PermissionMap reconciliation | | | | |
