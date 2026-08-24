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

1. Read [reference.md](reference.md) and [endpoint-pack.md](endpoint-pack.md).
2. For each endpoint, locate the SME Platform Lua route and its `feature()`.
3. Locate the Protected API route and `PermissionMap.cs` entitlement mapping.
4. Trace every caller and UI/BFF eligibility gate in the supplied repositories.
5. Apply the UI/BFF eligibility-gate procedure in [reference.md](reference.md):
   business context → role evaluation → entitlements → `enabledFeatures` →
   product gate → endpoint-feature alignment.
6. Reconcile the Lua feature, Protected API entitlement, BFF feature calculation, and Web UI/route gate.
7. Create a cited pack at `<investigation-root>/docs/packs/<endpoint-slug>.md` from [endpoint-pack.md](endpoint-pack.md), and update its row in `<investigation-root>/docs/registry.md`.
8. Set the verdict to `Insufficient evidence` when any layer is unverified.

## Delegation

Delegate only when the batch has two or more independent endpoint families, or more than two endpoints. Do not launch one research agent per layer per endpoint.

1. Collect shared mapping once (Platform Lua `feature()`, Protected API route, `PermissionMap.cs` entitlement) and put it in each research-agent prompt. Do not create a separate catalogue file.
2. Dispatch one **read-only** research agent per endpoint family (typically 3–5 endpoints that share a product or BFF pattern). Give absolute repository paths and exact endpoint scope.
3. Keep one **coordinator** as the only writer of packs and `docs/registry.md`. Research agents return evidence handoffs; they must not set `No expected user impact`.
4. Dispatch a **reviewer** only for packs with `Conflict`, `Missing`, `Alias`, `Partially gated`, or `Ungated`. Fully cited packs without those statuses go to the human batch review.

Read [reference.md](reference.md) for the evidence-handoff format and interference rules.

## Review rules

- Use only the template mapping statuses and cite every `Matched` or `Not applicable` value.
- Record a finding for a conflict, missing value, extra mapping, undocumented alias, different method behaviour, or a client route whose tenant identifier cannot be matched to the entitlement context.
- Do not set `No expected user impact` while any finding is unresolved.
- Review batches of 5–8 endpoint packs, then stop for human review before beginning another batch.

## Output

Return the updated evidence packs, registry rows, unresolved findings, and each endpoint verdict. Keep generic authentication plumbing out of packs unless the endpoint route deviates from the established baseline.

## Reference

Read [reference.md](reference.md) for the full baseline, UI/BFF procedure, discrepancy policy, batch-review procedure, and subagent delegation. Use [endpoint-pack.md](endpoint-pack.md) as the pack skeleton.
