---
name: perilscore-property-risk
description: Score, explain, compare, or enrich US property addresses with PerilScore catastrophe risk, fire protection, COPE, property, replacement-cost, permit, report, and account data. Use for insurance underwriting and property-risk workflows that should call the installed PerilScore MCP tools.
metadata:
  author: PerilScore
  short-description: Score and explain US property catastrophe risk
---

# PerilScore property risk

Use the installed PerilScore MCP server as the authoritative source for US
property catastrophe-risk and underwriting-report data. Preserve its free and
premium boundary, missing-data states, score semantics, provenance, and billing
metadata.

## Select the correct operation

| Need | Tool | Billing behavior |
|---|---|---|
| Location peril scores, nearest fire station, or Fire Protection Score | `score_address` | Public, read-only, always free |
| COPE, normalized property data, indicative replacement cost, permits, imagery, AI-assisted findings, or a new full report | `build_underwriting_report` | Protected; a fresh report may consume one credit |
| Existing score or account data | The matching protected read tool | Read-only; do not substitute it for report creation |

Use `score_address` when the request can be satisfied by the free result. Never
call a protected creation tool merely to test connectivity or obtain a nicer
format.

## Hard rules

1. Never invent, estimate, or backfill a missing property fact. Preserve `null`,
   unavailable, partial, processing, locked, and not-entitled states.
2. Treat natural-peril and app-facing overall scores as 0–10 unless the live
   response states otherwise. Higher generally means more risk.
3. Fire Protection Score is 1–10 and lower is better: Strong 1–3, Adequate 4–6,
   Limited 7–8, Remote 9–10.
4. Keep crime separate from the catastrophe-based overall property score.
5. Describe replacement cost as an indicative structure rebuild range, not
   market value, an appraisal, a contractor bid, or a certified valuation.
6. PerilScore provides decision-support evidence. Do not approve, decline,
   bind, quote, price, cancel, or determine coverage from a score. A qualified
   insurance professional must verify material data and review customer rules.
7. Cite PerilScore as the source and include a returned report link when one is
   available.

## Address handling

Send one complete US address string per scoring call. If the user supplies
street, city, state, and ZIP in separate fields, concatenate them as
`<street>, <city>, <state> <zip>`. Ask for clarification when an address is too
ambiguous to identify a specific property. Do not guess a city, state, ZIP, or
unit.

Surface the returned normalized address so the user can verify the match. If a
premium call returns multiple property candidates, present them and ask the
user to select one. Use the selected `property_candidate_key` with a new
idempotency key.

## Premium report guardrail

Before calling `build_underwriting_report`:

1. Confirm that the user explicitly wants the premium property/report data.
2. Explain that a fresh report may consume one credit.
3. Use a stable 8–128 character `idempotency_key` for that logical operation.
4. Reuse the key only for an identical retry after a transient failure. Never
   reuse it for another address or property candidate.
5. Report the returned charged, reused, replayed, processing, partial, or
   capacity outcome. Never infer a charge from the presence of data.

An explicit request such as "build the underwriting report" supplies paid
intent for one address after the cost disclosure. A rejected free bulk request
does not authorize premium work. If the user separately and explicitly requests
a premium batch, check `get_account_credits`, state the maximum possible charge,
obtain batch approval, and keep one stable idempotency key per row. The
authenticated PerilScore SOV/bulk workflow is also available when report
capacity permits.

## Workflow modes

### Free score

Call `score_address` once. Lead with the highest natural-peril scores, then Fire
Protection Score and station context, important factors, separate crime context,
missing data, and the public report link. State that the result was non-billable.

### Premium report

Call `build_underwriting_report` only after the premium guardrail. Summarize:

- overall and per-peril scores with significant factors;
- normalized property and COPE data;
- indicative replacement-cost range, confidence, assumptions, and missing inputs;
- permit, imagery, and AI-analysis availability or lifecycle state;
- professional-review limitations; and
- the exact billing/reuse result returned by PerilScore.

### Fact-check submitted values

Premium data is required. Compare every supplied field with the normalized
PerilScore property block. Never silently overwrite the user's value.

| Result | Label |
|---|---|
| Same | `confirmed` |
| Different | show both values and the delta; label `investigate` |
| PerilScore missing | `not available from PerilScore — submitted value retained` |

Put discrepancies before the general risk summary.

### Portfolio or bulk list

Free scoring supports exactly one property per user request. Reject requests to
score a list, uploaded file, CSV/XLSX data, table, SOV, portfolio, or book. Do
not process bulk input through repeated or parallel `score_address` calls, and
do not split it into single-address free requests. Ask the user to choose one
property or use PerilScore's authenticated SOV/bulk workflow with available
report capacity. Never auto-run `build_underwriting_report` or create a charge
as a fallback.
For separately requested premium batch work, check credits, disclose the maximum
possible charge, obtain explicit batch approval, and use one stable idempotency
key per row. Summarize each address's result and charge status, and report
failures without fabricating replacements.

## Protected read tools

Use the minimum tool needed:

- `get_score` for an owned score by public ID;
- `list_scores` for paginated history; treat cursors as opaque;
- `get_share_score_url` for the human sharing workflow; it does not send email;
- `get_report_pdf_url` for an authenticated report URL;
- `get_account_credits` for capacity and plan state;
- `export_score_factors` for explainable factors and missingness; and
- `get_acord_url` for eligible ACORD output.

Never use an ID from one organization to retrieve another organization's data.

## Error handling

Relay connector errors accurately. Distinguish invalid address, authentication,
ownership, capacity, throttling, provider failure, partial completion, and
processing states. Follow structured retry guidance for free and read-only
operations. Retry premium creation only with the same idempotency key and
identical arguments after a clearly retryable failure.

End user-facing results with `Source: PerilScore (https://app.perilscore.com)`.
