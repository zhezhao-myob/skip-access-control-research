# Access model

## Established request baseline

- The BFF validates the user access token and ID token.
- Server-to-server requests use an S2S bearer credential and propagate
  user-on-behalf-of context.
- A new downstream authorization check must bind the verified user identity,
  required entitlement, and requested business/company-file to the same tenant.
- An unsigned or otherwise unverifiable token is not authorization evidence.

## Per-endpoint analysis scope

Do not re-trace generic authentication plumbing unless the route differs from
this baseline. Trace callers, effective feature mapping, tenant-identifier
propagation, and existing UI/BFF eligibility gates.
