# Endpoint evidence pack template

Use this template to produce one concise, cited evidence pack per SME Platform
API endpoint. Copy the pack skeleton below into `docs/packs/<endpoint-slug>.md`
(for example, `docs/packs/entitlements-balance-summary-report.md`).

**Consumes:** one supplied SME Platform API endpoint (method and path).

**Produces:** one evidence pack suitable for human review and registry linkage.

See [access-model.md](../access-model.md) for the established request baseline
and per-endpoint analysis scope. Update the corresponding row in
[registry.md](../registry.md) when the pack is complete.

---

## Mapping status definitions

Each cell in the **Mapping reconciliation** table uses exactly one of these
status values:

| Status | Meaning |
| --- | --- |
| `Matched` | Observed value at this layer aligns with the reconciled entitlement or feature for this endpoint. **Requires a cited source** (file path, symbol, or doc link). |
| `Conflict` | Observed value disagrees with another layer or with expected access for this endpoint. Cite all conflicting sources. |
| `Missing` | No mapping or gate was found at this layer after reasonable investigation. State what was searched. |
| `Alias` | Observed value differs in name or shape but represents the same effective access (for example, legacy feature key). Cite both sides of the alias. |
| `Not applicable` | This layer does not apply to the endpoint (for example, no Web UI route). **Requires a cited source** explaining why the layer is out of scope. |

**Citation rule:** `Matched` and `Not applicable` must include a cited source in
the **Source evidence** column. Other statuses should cite evidence where it
exists; do not mark `Matched` or `Not applicable` without a citation.

---

## Pack skeleton

Copy from the heading below through **Residual risk**, then replace placeholders
and fill every section with cited evidence.

```markdown
# <METHOD> <SME Platform API path>

## Proposed authorization predicate
Verified user identity has the required entitlement for the effective feature
and access to the requested business/company-file.

## Caller inventory

## Tenant identifier propagation

## Mapping reconciliation
| Layer | Observed value | Source evidence | Status |
| --- | --- | --- | --- |
| SME Platform API Lua `feature()` | | | |
| Protected API `PermissionMap.cs` entitlement | | | |
| BFF feature calculation | | | |
| Web UI/route gate | | | |

## Findings

## Verdict
`No expected user impact` / `No UI impact, API behaviour change` /
`User-visible impact` / `Insufficient evidence`

## Residual risk
```

### Verdict values

Use exactly one of:

- `No expected user impact` — access behaviour unchanged for entitled users.
- `No UI impact, API behaviour change` — API enforcement changes without a
  corresponding UI gate change.
- `User-visible impact` — users may lose or gain access in the product surface.
- `Insufficient evidence` — reconciliation or caller tracing incomplete; do not
  guess.
