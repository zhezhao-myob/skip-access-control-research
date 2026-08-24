# Skip Access Control Investigation

Human-reviewable impact analysis for introducing strict user role, entitlement,
and company-file access checks to SME Platform API endpoints.

## Review rule

Every endpoint must reconcile SME Platform API Lua `feature()`,
AccountRight Protected API `PermissionMap.cs`, BFF feature calculation, and
web UI gating. Any missing or conflicting mapping is a finding and blocks a
`No expected user impact` verdict.
