---
name: access-control-impact
description: Produces human-reviewable access-control impact packs for SME Platform API endpoints. Use when analysing strict role, entitlement, and business/company-file checks, tracing endpoint callers, or reconciling Lua, PermissionMap.cs, BFF, and UI feature mappings.
---

# Access-control impact

Produce cited endpoint evidence packs and update the review registry. Treat every endpoint as a human-reviewable access-control impact analysis; do not infer missing mappings.

## Fixed input contract

```text
Endpoints:
- METHOD /businesses/{business-id}/...

Repositories:
- sme-platform-api
- sme-web
- sme-web-bff
- reporting-web
- reporting-backends
- accountright-protected-api

Investigation root:
- absolute path to skip-access-control
```

Ask for the endpoint list, repository locations, or investigation root if any required input is absent.

## Authorization baseline

The BFF validates user access and ID tokens. Server-to-server calls use an S2S bearer credential with user-on-behalf-of context. Authorization must bind the verified user, effective entitlement, and requested business/company-file to one tenant. An unsigned or unverifiable token is not identity or authorization evidence.

Do not re-trace generic BFF or S2S token plumbing unless a route differs from this baseline. Read [reference.md](reference.md) when evaluating identity propagation or exceptions.

## Workflow

1. Read `<investigation-root>/docs/access-model.md` and `docs/templates/endpoint-pack.md`.
2. For each endpoint, locate the SME Platform Lua route and its `feature()`.
3. Locate the Protected API route and `PermissionMap.cs` entitlement mapping.
4. Trace every caller and UI/BFF eligibility gate in the supplied repositories.
5. Reconcile the Lua feature, Protected API entitlement, BFF feature calculation, and Web UI/route gate.
6. Create a cited pack at `docs/packs/<endpoint-slug>.md`, following the template exactly, and update its row in `docs/registry.md`.
7. Set the verdict to `Insufficient evidence` when any layer is unverified.

## Review rules

- Use only the template mapping statuses and cite every `Matched` or `Not applicable` value.
- Record a finding for a conflict, missing value, extra mapping, undocumented alias, different method behaviour, or a client route whose tenant identifier cannot be matched to the entitlement context.
- Do not set `No expected user impact` while any finding is unresolved.
- Review batches of 5–8 endpoint packs, then stop for human review before beginning another batch.

## Output

Return the updated evidence packs, registry rows, unresolved findings, and each endpoint verdict. Keep generic authentication plumbing out of packs unless the endpoint route deviates from the established baseline.

## Reference

Read [reference.md](reference.md) for the full baseline, reconciliation guidance, discrepancy policy, and batch-review procedure.
