# Copyable prompt recipes

Replace bracketed placeholders. Never paste an actual API key into a prompt.

## Build an integration

```text
Use $build-with-perilscore to integrate PerilScore into this codebase for [USE CASE].

Inspect the existing stack and conventions first. Choose REST for deterministic
server-side requests or canonical MCP when an AI agent should discover and call
tools. Make the free versus premium behavior explicit before implementation.

Requirements:
- Read the current PerilScore llms.txt and the applicable OpenAPI or MCP contract.
- Keep PERILSCORE_API_KEY server-side; never expose it to browser code or logs.
- Preserve null, unavailable, locked, partial, processing, and not-entitled states.
- Treat peril scores as 0–10 and Fire Protection Score as 1–10 where lower is better.
- Keep crime separate from the catastrophe-based overall property score.
- Describe replacement cost as indicative, not an appraisal or certified valuation.
- Require explicit intent and a stable idempotency key for a potentially billable
  MCP report; never make a paid request as an automated test.
- Add timeouts, bounded retry/rate-limit handling for safe operations, and mocked tests.
- Treat authenticated premium REST as billing-sensitive: a fresh score may consume
  one credit and there is no caller-supplied idempotency key. Do not automatically
  retry it after an ambiguous timeout; prefer premium MCP for replay-safe creation.
- Treat results as decision-support evidence requiring professional verification.

Implement the smallest clean adapter consistent with this repository. Finish by
listing files changed, environment variables, tests run, live calls made, and
assumptions.
```

## Add a guaranteed-free lookup

```text
Use $build-with-perilscore to add a guaranteed non-billable property-risk lookup
for [PRODUCT/WORKFLOW]. Use canonical MCP score_address. Do not use REST for a
guaranteed-free lookup: it creates authenticated, credit-backed full reports.
Show per-peril scores, fire-station context, Fire Protection Score with lower-is-
better labeling, missing-data states, and crime separately. Keep the overall
premium score locked. Add mocked success, invalid-address, throttling, and timeout
tests without calling production.
```

## Add explicit premium MCP reporting

```text
Use $build-with-perilscore to add an authenticated canonical MCP workflow that
creates a premium underwriting report only after [USER CONFIRMATION MECHANISM].
Use build_underwriting_report with a persisted idempotency key per logical
operation. Reuse that key only for identical transient retries; support property
candidate selection with a new key. Clearly render charged, reused, processing,
partial, and not-entitled outcomes. Keep the API key server-side and mock all
billable calls in tests.
```

## Ask an MCP agent for a free score

```text
Use PerilScore's free score_address tool for [FULL US ADDRESS]. Summarize the
highest natural-peril scores, Fire Protection Score (lower is better), important
factors, crime context, and unavailable data. Do not create a premium report and
do not treat the result as a coverage decision.
```

## Ask an MCP agent for a premium report

```text
Build a premium PerilScore underwriting report for [FULL US ADDRESS] using the
stable idempotency key [8-128 CHARACTER KEY]. I understand that a fresh report
may consume one credit. Report whether a credit was charged or a recent result
was reused, then summarize COPE, RCV assumptions, permit availability, imagery
or AI-analysis status, significant peril factors, missing data, and fields that
require professional verification.
```
