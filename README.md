# PerilScore agent plugin

Official PerilScore plugin for Cursor, Grok Build, and Grok Bot. It connects
agents to PerilScore's hosted MCP server for address-level catastrophe risk,
fire-protection context, property enrichment, COPE, indicative replacement
cost, permits, reports, and account workflows for US properties.

## What is included

- Hosted MCP endpoint: `https://app.perilscore.com/mcp`
- `perilscore-property-risk` skill for underwriting and property-risk work
- `build-with-perilscore` skill for application and agent integrations
- Native Cursor and Grok plugin manifests
- No hooks, executable scripts, downloaded binaries, or post-install code

The MCP server currently advertises nine tools:

- `score_address` is public, read-only, and always free.
- `build_underwriting_report` is protected and can consume one credit for a
  fresh premium report. It requires explicit paid intent and an idempotency key.
- `get_score`, `list_scores`, `get_share_score_url`, `get_report_pdf_url`,
  `get_account_credits`, `export_score_factors`, and `get_acord_url` are
  protected account/report tools.

Use the live tool list and schemas as the source of truth.

## Install

### Cursor and Grok Bot

Install **PerilScore** from Cursor Marketplace. Cursor and Grok Bot use the
plugin's `mcp.json` and the public OAuth client `perilscore-cursor-mcp`.
Authentication uses Authorization Code with S256 PKCE and no client secret.

Until the listing is approved, clone this repository into Cursor's local plugin
directory and reload Cursor:

```bash
git clone https://github.com/PerilScore/perilscore-agent-plugin.git \
  ~/.cursor/plugins/local/perilscore
```

### Grok Build

Install from the official xAI plugin marketplace when the listing is approved,
or install the repository directly:

```bash
grok plugin install https://github.com/PerilScore/perilscore-agent-plugin --trust
```

Grok Build uses `.mcp.json` and the public OAuth client
`perilscore-grok-mcp`. It binds a random `127.0.0.1` loopback port and completes
S256 PKCE without a client secret.

### Grok on the web

Open [Grok connectors](https://grok.com/connectors), choose **New Connector →
Custom**, and enter `https://app.perilscore.com/mcp`.

## Example requests

Free, non-billable lookup:

> Use PerilScore's free score_address tool for 100 Main St, Miami, FL 33131.
> Summarize the highest natural-peril scores, Fire Protection Score (lower is
> better), important factors, crime context, and unavailable data. Do not build
> a premium report.

Premium report:

> Build a premium PerilScore underwriting report for 100 Main St, Miami, FL
> 33131 using idempotency key miami-main-review-001. I understand that a fresh
> report may consume one credit. Report whether a credit was charged or a recent
> result was reused, then summarize COPE, RCV assumptions, permits, significant
> factors, missing data, and fields requiring professional verification.

Integration work:

> Use the build-with-perilscore skill to add a guaranteed-free property-risk
> lookup to this application. Preserve missing data, keep crime separate, label
> Fire Protection Score as lower-is-better, and mock production calls in tests.

## Authentication and data access

Public discovery, MCP App resources, and `score_address` require no account.
Protected tools use OAuth 2.1 with PKCE or a PerilScore organization API key.
OAuth requests only the documented `score:read`, `score:write`, and
`account:read` scopes. The plugin contains no credentials.

This plugin sends tool arguments to `https://app.perilscore.com` and receives
the corresponding PerilScore results. It does not include telemetry, hooks, or
background executables. Review PerilScore's [privacy policy](https://app.perilscore.com/privacy/)
and [terms](https://app.perilscore.com/terms/).

Never place an API key in a repository, prompt, URL, screenshot, or chat
transcript. For organization-scoped service access, use the host's secret or
environment-variable facility and send `Authorization: Bearer ps_api_...` only
to `https://app.perilscore.com/mcp`.

## Important interpretation rules

- Peril and app-facing overall scores are generally 0–10; higher means more risk.
- Fire Protection Score is 1–10; lower is better.
- Crime is separate from the catastrophe-based overall score.
- Missing data is not zero risk.
- Replacement cost is an indicative structure rebuild range, not an appraisal
  or certified valuation.
- PerilScore provides decision-support evidence and does not make coverage,
  pricing, eligibility, binding, or cancellation decisions. Qualified insurance
  professionals must verify material data and review customer rules.

## Development and verification

Validate the Grok package locally:

```bash
grok plugin validate .
```

For Cursor, copy the repository to `~/.cursor/plugins/local/perilscore`, reload
the app, and confirm both skills and the `perilscore` MCP server appear. Test
public discovery and `score_address` without authentication. Do not make a live
premium call solely as a connectivity test.

Live contracts:

- [MCP manifest](https://app.perilscore.com/.well-known/mcp.json)
- [Developer documentation](https://app.perilscore.com/developers/)
- [LLM discovery](https://app.perilscore.com/llms.txt)

## Support and security

- Support: [info@perilscore.com](mailto:info@perilscore.com)
- Security reporting: see [SECURITY.md](SECURITY.md)

## License

The plugin package is available under the [MIT License](LICENSE). PerilScore's
hosted service remains governed by its own terms.
