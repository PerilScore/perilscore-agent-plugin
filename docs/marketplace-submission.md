# Marketplace submission and reviewer verification

The public plugin package and the hosted MCP service have separate releases.
Plugin version 1.0.2 updates submission guidance; the hosted MCP server
currently identifies itself as 1.0.0. `server.json` describes that remote
service, not an executable server shipped in this repository.

## Cursor Marketplace and Grok Bot

Submit this official public repository at
https://x.ai/bot/marketplace/submit-a-plugin. The form accepts a public plugin
repository or hosted MCP URL and offers Grok Bot, Cursor, and Grok marked
“coming soon.” Prefer the repository to include the skills and OAuth config;
include `https://app.perilscore.com/mcp` in Testing Instructions. Reference any
previous Cursor application and ask for the records to be linked. There is no
documented automatic migration of pending Cursor applications.

Use PerilScore's GitHub identity and `info@perilscore.com`. Recommended handle:
`perilscore`; website: `https://perilscore.com`. The existing
`assets/logo.png` is a 512×512 PNG with an opaque white background, matching
the form's square/background-plate requirement. Its public URL is
`https://raw.githubusercontent.com/PerilScore/perilscore-agent-plugin/main/assets/logo.png`.

Do not check the local/in-app testing attestation until both have been
performed on the actual release. Submit only after reviewing the current
Publisher Terms linked by the form. Connector installation is free; explicitly
disclose that fresh premium reports use existing PerilScore account credits.

The package's `mcp.json` uses the public static client
`perilscore-cursor-mcp`, no client secret, and S256 PKCE. The server registers
these exact callbacks:

- Hosted Cursor / Grok Bot: `https://www.cursor.com/agents/mcp/oauth/callback`
- Cursor desktop: `http://localhost:8787/callback`

The client ID is public metadata. Customer tokens, reviewer credentials, API
keys, and customer report data never belong in this repository or a public PR.

## Grok Build

The Grok package uses `.grok-plugin/plugin.json` and `.mcp.json`. Its public
static client is `perilscore-grok-mcp`. Grok Build chooses an available port
for `http://127.0.0.1:<port>/callback`; the server permits port-agnostic
matching for that loopback host and callback path. No client secret is needed.

Validate the package with `grok plugin validate .`.

For the official catalog, fork https://github.com/xai-org/plugin-marketplace
and add one remote entry to `.grok-plugin/marketplace.json` pointing at
`https://github.com/PerilScore/perilscore-agent-plugin.git`. Pin a full,
publicly reachable commit SHA from the official repository. Use PerilScore
brand keywords and owned domains, not generic discovery terms. Generate and
commit the index, then run the two official CI checks:

```bash
python3 scripts/generate-plugin-index.py
python3 scripts/validate-catalog.py
python3 scripts/generate-plugin-index.py --check
```

Open a PR with xAI's template. A later package update requires a new SHA pin
and regenerated index; bump the plugin manifest versions when releasing it.
No additional PerilScore backend or vendored copy is required.

## Reviewer walkthrough

Record the host, tested commit, timestamp, observed outcome, and redacted
evidence for each step. A local automated test is not a completed Grok Bot
host test.

1. Install the package. Confirm both skills and the `perilscore` server appear.
2. Run one free score with no account. Confirm `credit_cost: 0` and
   `credit_charged: false`; missing evidence stays explicit.
3. Connect an authorized test account using the host's Authenticate flow.
   Confirm sign-in returns to consent and the host callback; no secret is
   copied into chat. Reading `get_account_credits` verifies premium capacity
   without creating a report.
4. Only with an explicitly approved test budget, create one premium report
   using a unique idempotency key. Reuse the exact arguments and key for a
   retry; verify only one logical report charge.
5. Read the returned public ID with `get_score`, including requested evidence
   sections. Verify reads use no additional credit and that another
   organization's ID is inaccessible.
6. If an already-paid test report has missing property records, explicitly
   request `retry_report_enrichment`. Confirm it keeps the same ID and uses
   no additional score credit. Refresh only while work is active.
7. Verify cancellation, disconnected authentication, read-only scope denial,
   and insufficient-capacity errors without silently upgrading free requests.

Provide reviewer sign-in instructions through the platform's private review
channel. Muse's dedicated reviewer password account is specific to Muse;
never reuse its identity or credentials for another marketplace.

Grok uses a separately provisioned reviewer identity at
`https://app.perilscore.com/auth/reviewer/grok/` once enabled by the operator.
This password sign-in returns to ordinary OAuth consent; it is not an OAuth
client secret or an authorization bypass. The account starts empty with a
bounded report-credit budget and automatic refill disabled. Verify actual
deployment, provisioning, login, balance, and callback before claiming the
review account is ready. Supply its password only in the platform's private
reviewer channel, never in this repository or a public PR.

## Official MCP Registry

`server.json` is prepared metadata for a remote-only listing. It contains no
package or runtime dependency and does not publish itself. Validate it with
the official `mcp-publisher validate server.json` command, then publish only
after authenticating ownership of the `io.github.PerilScore/` namespace.

Registry discovery does not create a platform-specific OAuth registration.
Free scoring is public. Premium access requires a client that supports the
documented static client registration, one of the trusted Claude clients, or
an authorized organization Bearer API key. PerilScore does not advertise
dynamic client registration or accept arbitrary client metadata URLs. Do not
claim universal one-click premium OAuth compatibility.

Smithery supports hosted URLs and uses Client ID Metadata Documents for its
OAuth gateway. Evaluate its exact client identity and callbacks before
enabling premium account connections; scanning the public tools alone does
not prove authenticated compatibility.
