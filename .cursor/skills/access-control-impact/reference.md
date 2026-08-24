# Access-control impact reference

## Token and authentication baseline

- The BFF validates both the user access token and ID token.
- Server-to-server calls use an S2S bearer credential and propagate user-on-behalf-of context.
- Authorization evidence must bind a verified user identity, the effective entitlement, and the requested business/company-file to the same tenant.
- An unsigned, malformed, expired, unverified, or otherwise unverifiable token is neither caller identity nor authorization evidence.

Do not re-prove generic BFF or S2S authentication plumbing in every pack. Trace it only if the endpoint route, credential flow, user-context propagation, or tenant binding differs from this baseline.

Per-endpoint analysis traces callers, effective feature mapping, tenant-identifier
propagation, and existing UI/BFF eligibility gates. Write human-review artefacts
to `<investigation-root>/docs/packs/` and `<investigation-root>/docs/registry.md`.

## Mapping layers

Reconcile these four layers for every endpoint:

1. SME Platform API Lua route and `feature()`.
2. Protected API route and `PermissionMap.cs` entitlement.
3. BFF feature calculation.
4. Web UI or route gate.

Use the endpoint-pack template statuses exactly:

- `Matched`: cited evidence shows the layer aligns with the effective access.
- `Conflict`: cited evidence shows disagreement between layers or with expected access.
- `Missing`: no mapping or gate was found; state the repositories, paths, and terms searched.
- `Alias`: different names or forms have cited evidence of the same effective access.
- `Not applicable`: cited evidence establishes that the layer does not apply.

`Matched` and `Not applicable` always require citations. Cite file paths and relevant symbols, route definitions, or documentation so a reviewer can reproduce the conclusion.

## Discrepancy policy

Create a finding for every:

- conflict between layers;
- missing value or gate;
- extra mapping not represented by the other layers;
- undocumented alias;
- difference in method behaviour for the same route family; or
- client route whose business, company-file, or tenant identifier cannot be matched to the entitlement context.

An unresolved finding blocks `No expected user impact`. Use `Insufficient evidence` whenever a mapping layer, caller, eligibility gate, or tenant-identifier relationship cannot be verified. Do not guess a benign result.

## Batch review

Work in batches of five to eight endpoint packs:

1. Complete the cited packs and their registry rows.
2. Recheck that all four layers and tenant propagation have evidence or an explicit status.
3. List unresolved findings and verdicts for human review.
4. Wait for review before beginning the next batch.

The batch boundary is a review control, not permission to carry unresolved findings into a `No expected user impact` verdict.
