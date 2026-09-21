# PerilScore product contract

Use this reference for product invariants. Obtain current field and tool schemas from the live discovery endpoints listed in `SKILL.md`.

## Free and premium matrix

| Surface | Free path | Premium path | Important distinction |
|---|---|---|---|
| REST `GET /api/v1/score` | — | Send `Authorization: Bearer $PERILSCORE_API_KEY`; a fresh eligible full report may consume one credit | REST is an authenticated production full-report surface. It does not accept a caller-supplied idempotency key and is not the explicit two-tool model used by canonical MCP. |
| Canonical MCP `/mcp` | Call public `score_address` | Call protected `build_underwriting_report` with `address` and `idempotency_key` | Free lookup and premium creation are separate operations |

The free response contains per-peril location scores, nearest-fire-station context, Fire Protection Score, and a locked overall score. It does not include premium property, RCV, permit, imagery, or AI underwriting data.

Premium output can include an Overall Property Score, normalized property and construction data, COPE, an indicative replacement-cost range and factors, permit signals where available, imagery metadata, AI-analysis lifecycle status, and billing metadata. Availability varies; never infer a field that is absent.

An authenticated account without capacity receives an explicit, non-billable capacity error from protected report creation. Canonical MCP `score_address` remains separately available for a free preview. Preserve the distinction between a preview and a completed full report.

Authenticated premium REST can reuse a recent matching score, but it does not expose the same caller-controlled idempotency and structured charge/reuse contract as canonical MCP. Treat an ambiguous premium REST timeout as billing-sensitive: do not automatically retry it. Prefer canonical MCP for agentic, concurrent, or retry-prone paid workflows.

## Canonical MCP operations

The canonical server currently advertises these operations; use `tools/list` as the source of truth:

- `score_address`: public, free, read-only, and always non-billable.
- `build_underwriting_report`: protected premium creation; requires a stable idempotency key and may consume one credit.
- `get_score`, `list_scores`: read owned score data and history.
- `get_share_score_url`: return the owned web sharing workflow; it does not send email.
- `get_report_pdf_url`: return an authenticated PDF URL for an owned score.
- `get_account_credits`: return capacity, plan, and billing status.
- `export_score_factors`: return explainable overall and per-peril factors.
- `get_acord_url`: return an authenticated ACORD 140 or eligible ACORD 80 URL.

Use the minimum OAuth scope advertised by the live manifest/tool metadata. Treat pagination cursors as opaque. Keep organization ownership boundaries intact for every public ID.

## Billing and idempotency

- A fresh authenticated premium REST score can consume one credit. REST has no caller-supplied idempotency key or equivalent explicit replay contract.
- A fresh premium MCP report can charge one credit after entitlement and capacity checks.
- A recent matching report or same-operation idempotent replay can be returned without another charge.
- Create one key per logical premium attempt. Persist it before sending the call.
- Reuse that key for timeouts or transient retries only when every non-key argument is identical.
- Never use the same key for a different address or property candidate.
- When multiple property units match, surface the candidates. Retry the selected candidate with its `property_candidate_key` and a new idempotency key.
- Inspect structured billing fields instead of deducing a charge from the presence of premium-looking data.

Treat `processing`, `partial`, maintenance, conflict, and not-entitled responses as distinct states. A permit failure may make a report partial; it does not make the report free or erase its billing result. Apply automatic retries only where the live contract makes replay safe; never blindly retry premium REST.

## Score semantics

- Natural perils: hurricane, wildfire, hail, flood, earthquake, and tornado.
- Crime: separate 0–10 underwriting context; exclude it from the headline catastrophe-based overall property score.
- Peril and app-facing overall scores: 0–10. Higher generally means more risk.
- Fire Protection Score: 1–10. Lower is stronger. Expected bands are Strong (1–3), Adequate (4–6), Limited (7–8), and Remote (9–10).
- Missing values: preserve as missing. A missing score is not zero risk.
- RCV: an indicative structure rebuild range with assumptions and missing inputs; not market value or a certified valuation.

## Failure behavior

Handle invalid input/address resolution, authentication failure, ownership denial, account capacity, throttling, timeouts, provider failures, address selection, partial results, and asynchronous processing explicitly. Follow HTTP `Retry-After` or structured retry guidance when returned. Bound retries and avoid retry storms.

For UI or agent output:

- Say which tier completed.
- Identify unavailable and unverified data.
- Show charge/reuse status for premium creation.
- Do not turn a score into an automatic coverage, pricing, cancellation, or eligibility decision.
