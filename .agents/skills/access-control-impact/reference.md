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

## UI/BFF eligibility-gate procedure

Reporting Web sends the user's access token and ID token as credentials, together
with the selected business context. It does not send roles or entitlements. The
BFF uses that context to obtain the business's `current-user` record and role
list through the platform gateway, derives the user's entitlement list, then
returns the resulting `enabledFeatures` list to Reporting Web.

Treat this as the standard investigation path for both Reporting Backends and
SME Web BFF unless the endpoint's implementation shows otherwise. Do not assume
that similarly named transformers or feature keys are equivalent; cite the
actual route and mapping evidence for the endpoint.

For each endpoint, determine and record:

1. **Business context** — the selected business/company-file identifier supplied
   to the BFF and the identifier used for `current-user` and role lookup.
2. **Role evaluation** — the BFF code that matches the current user's role IDs
   to the selected business's roles and derives entitlements.
3. **Feature evaluation** — the entitlement-to-`enabledFeatures` mapping,
   including applicable subscription or region filters.
4. **Product gate** — the web widget, navigation item, SPA route, or BFF route
   that tests the required enabled feature before requesting endpoint data.
5. **Endpoint alignment** — whether the feature used by that gate represents
   the same effective access as the Platform Lua feature and the Protected API
   `PermissionMap.cs` entitlement.

Classify the UI/BFF layer as:

- `Gated` — cited server-side feature calculation and a cited UI/BFF gate both
  require the aligned feature.
- `Partially gated` — a feature is calculated or tested, but a caller, route,
  or method variant can bypass the gate.
- `Ungated` — a user-facing caller reaches the endpoint without a matching
  feature gate.
- `Not applicable` — cited evidence establishes that no web/BFF caller exists.
- `Insufficient evidence` — the business context, feature mapping, or gate
  cannot be verified.

The UI/BFF classification is evidence about expected user impact, not request
authorization. A `Gated` result does not replace the downstream check for the
verified user, the effective entitlement, and access to the requested
business/company-file.

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

## Subagent delegation

Use subagents for cost and quality only when work is independent. Do not split Platform, BFF, and UI into three agents for the same endpoint: each agent re-orients on the same repositories and the conclusions drift.

### When to delegate

- At least two independent endpoint families in the batch, or more than two endpoints.
- Endpoint families share a product, BFF module, or Lua adapter pattern and do not require each other's pack verdicts.

Do not delegate when there is a single endpoint, when families share unresolved mapping questions, or when agents would write the same files.

### Roles

| Role | Scope | Writes |
| --- | --- | --- |
| Coordinator (this session) | Shared mapping, family grouping, pack synthesis, verdicts | `docs/packs/`, `docs/registry.md` |
| Research agent | One endpoint family: callers, tenant propagation, UI/BFF gate | None (read-only on application repos and investigation docs except its report file) |
| Reviewer | Packs whose status includes `Conflict`, `Missing`, `Alias`, `Partially gated`, or `Ungated` | None |

Research agents must not conclude `No expected user impact`. The coordinator sets the verdict only after reconciling all layers, including Lua and `PermissionMap.cs`. A UI/BFF classification of `Gated` is impact evidence only.

### Shared mapping

Before launching research agents, the coordinator collects Lua `feature()`, Protected API route, and `PermissionMap.cs` entitlement for the family (from existing packs, `docs/registry.md`, or a first pass in this session). Put that mapping in the research prompt so agents do not re-discover it unless a row is `Missing` or `Conflict` and the coordinator asked them to resolve it.

Do not maintain a separate mapping-catalogue file. Persist mapping in the pack and registry after synthesis.

### Evidence handoff

Each research agent returns one block per endpoint, no pack files:

```text
Endpoint:
Layer:
Observed value:
Exact source path + symbol:
Search scope:
Status:
Reasoning:
Open question:
```

Cover at least: callers, tenant identifier propagation, and the UI/BFF eligibility-gate checks. Cite file paths and symbols. If a search found nothing, state repositories, paths, and terms searched.

### Interference rules

- Research agents work in parallel only across families; they do not edit `docs/registry.md` or `docs/packs/`.
- The coordinator writes packs and registry rows after all family reports for the batch are in.
- Reviewer agents read packs and reports; they do not rewrite application code.
- Absolute paths to all six repositories and the investigation root appear in every research prompt.
