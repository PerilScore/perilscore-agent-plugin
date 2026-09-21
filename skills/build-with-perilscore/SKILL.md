---
name: build-with-perilscore
description: Build, review, or troubleshoot integrations with the PerilScore REST API or canonical MCP server, including anonymous free property scoring, authenticated premium underwriting reports, saved-score workflows, and copyable implementation prompts. Use when an application or AI agent needs US property peril scores, fire-protection context, COPE, indicative replacement cost, permits, score factors, reports, ACORD links, or PerilScore account data.
---

# Build with PerilScore

Build against PerilScore's live contracts while preserving its free/premium boundary, billing safety, missing-data semantics, and decision-support role.

## Choose the integration path

| Need | Use | Billing behavior |
|---|---|---|
| Guaranteed non-billable lookup in an agent | MCP `score_address` without auth | Always free; never consumes a credit |
| Explicit premium report in an agent | MCP `build_underwriting_report` with OAuth or a Bearer API key | A fresh report may consume one credit |
| Deterministic server-to-server full reports | REST `GET /api/v1/score` with an API key | A fresh report may consume one credit |
| Read existing reports or account state | Protected MCP read tools | Read-only; do not substitute them for report creation |

Prefer the canonical MCP endpoint at `https://app.perilscore.com/mcp`. Do not start new work on legacy MCP endpoints. Prefer REST for conventional request/response application code and MCP when an agent should discover and invoke tools. Prefer premium MCP over premium REST when replay-safe billing or reliable charge metadata matters: premium REST has no caller-supplied idempotency key.

If a workflow must never incur a charge, use canonical MCP `score_address`. REST is a production full-report surface: make potential credit consumption visible before implementation and invocation.

Read [references/product-contract.md](references/product-contract.md) before implementing or reviewing an integration. Read [references/prompt-recipes.md](references/prompt-recipes.md) when producing prompts, examples, onboarding copy, or agent instructions.

## Discover the live contract

Retrieve the current production contract before writing request or response code:

- General behavior: `https://app.perilscore.com/llms.txt`
- REST: `https://app.perilscore.com/openapi.json`
- MCP: `https://app.perilscore.com/.well-known/mcp.json`, then MCP `tools/list`

Treat those endpoints as authoritative for fields, tool names, schemas, scopes, protocol versions, and limits. Treat the bundled references as durable product guardrails. Do not hardcode undocumented response fields or copy a stale schema into the integration.

If live discovery is unavailable, say so, implement only against verified fields, and isolate parsing so the contract can be refreshed later.

## Implement the adapter

1. Inspect the host project's framework, HTTP/MCP client, configuration, error conventions, and test style.
2. Define one narrow PerilScore adapter rather than spreading vendor-specific logic across views or UI components.
3. Keep `PERILSCORE_API_KEY` server-side. Never place it in browser bundles, source, prompts, URLs, logs, fixtures, screenshots, or chat transcripts. Use a backend boundary for browser applications.
4. Add explicit timeouts, rate-limit handling, and structured error mapping. Bound retries for free and read-only operations. Do not automatically retry premium REST after an ambiguous timeout. Retry premium MCP creation with the same idempotency key only when the arguments represent the same operation.
5. Preserve `null`, unavailable, partial, locked, processing, and not-entitled states. Never fabricate enrichment, convert missing data to zero, or label a free fallback as premium.
6. Expose returned provenance, assumptions, confidence, factors, and billing metadata when relevant. Keep policy or eligibility logic outside the PerilScore adapter.

## Guard premium operations

Before calling `build_underwriting_report`:

1. Require clear user intent to create a potentially billable report.
2. Authenticate with the minimum required scope.
3. Generate a stable 8–128 character idempotency key for the logical operation and persist it across transient retries.
4. Reuse a key only with identical arguments. If address selection is required, present the candidates and use the returned candidate key with a new idempotency key.
5. Report whether a credit was charged, a recent report was reused, processing continues, output is partial, or the account lacks capacity.

Do not invoke a premium operation merely to test connectivity, configuration, or parsing. Use discovery, mocks, fixtures, or the free tool instead.

For premium REST, disclose that a fresh score may consume one credit and that the endpoint does not accept a caller-supplied idempotency key. Avoid automatic retries after timeouts or connection loss. Prefer `build_underwriting_report` when the workflow requires replay-safe creation and explicit billing metadata.

## Interpret results correctly

- Treat peril scores as 0–10 unless the live contract states otherwise.
- Treat `fire_protection_score` as 1–10 where **lower is better**; do not average it as though higher were safer.
- Show crime separately from the catastrophe-based overall property score.
- Describe replacement-cost output as an indicative structure rebuild range, not market value, an appraisal, a contractor bid, or a carrier cost-manual certification.
- Present PerilScore as decision-support evidence. Require a qualified insurance professional to verify material data and customer-authored rules before a decision affecting an applicant, insured, or policyholder is finalized or communicated.

## Verify and hand off

- Mock external calls in automated tests. Cover free, premium, out-of-capacity, missing-data, invalid-address, rate-limit, timeout, and retry/replay behavior relevant to the chosen path.
- Assert that free workflows cannot invoke premium creation and that browser-delivered code contains no credential.
- Validate score direction and scale, crime separation, null preservation, and RCV labeling.
- Run the host project's focused checks, then broader checks when practical.
- Make no live billable request without explicit authorization. If authorized, use a real business address and reconcile returned billing metadata.
- Summarize the selected path, files changed, environment variables, tests run, live calls made, and any contract assumptions.
