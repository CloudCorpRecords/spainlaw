# spainlaw

Spain legal guidance from your own machine's tool list. spainlaw is a local forwarding client for the published Legal Fournier Spain-legal MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the remote service, returning the answers exactly as the service sends them back.

The built surface covers published-content search and fetch, the published paid-consultation options and live paid-call availability, and rule-based guidance for people relocating to or living in Spain: visa-route ranking, Beckham-regime eligibility checks, residency-path milestones, the NIE process, tax-regime comparison, and routing a case to the firm's help channels. Nothing is computed or stored locally — the package is a pipe, not a law firm.

This is a labelled subset rebuild: the upstream capture enumerates thirteen tools, and ten are built here. Three are omitted — `find_legal_fournier_tool` and `run_legal_fournier_tool`, a meta-dispatch pair that finds and runs unenumerated calculators and screeners by id, and `create_legal_fournier_contact_request`, which creates a lead at the firm from a person's name, email and case summary. The omitted tools are not registered, not served in `tools/list`, and not available on the CLI.

## The tools

- **`search`** — Search published articles, videos and canonical legal resources, with language, type and topic filters. No parameters are required.
- **`fetch`** — Fetch one published item in full by id or URI, with offset and length controls. No parameters are required.
- **`get_paid_consultation_options`** — The published paid consultation options, prices and quote/payment endpoints, returned as data. Read-only; it creates no order and submits no payment. Takes no input.
- **`get_paid_call_availability`** — Live paid-call availability for a date range. `from`, `to` and `timezone` are required.
- **`get_visa_options`** — Ranked visa routes for a set of circumstances. `nationality`, `income_source` and `intent` are required.
- **`check_beckham_eligibility`** — A gatecheck for the Beckham special tax regime. `employment_type` and `move_reason` are required.
- **`get_residency_path`** — Residency-path milestones from a current status and time in Spain. `current_status` and `years_in_spain` are required.
- **`explain_nie_process`** — How the NIE process works. Takes no input.
- **`compare_tax_regimes`** — A comparison of tax regimes for a set of circumstances. No parameters are required.
- **`route_to_legal_fournier_help`** — Compose a ready-to-send handoff message for the firm's help channels from non-contact inputs only; it submits nothing. `area` is required.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/spainlaw mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "spainlaw": {
      "command": "npx",
      "args": ["-y", "@reneturcios/spainlaw", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/spainlaw
docker run --rm ghcr.io/cloudcorprecords/spainlaw get_visa_options '{"nationality":"US","income_source":"remote","intent":"live"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
spainlaw search '{"query":"digital nomad visa","language":"en"}'
spainlaw fetch '{"id":"example"}'
spainlaw get_paid_consultation_options '{}'
spainlaw get_visa_options '{"nationality":"US","income_source":"remote","intent":"live"}'
spainlaw check_beckham_eligibility '{"employment_type":"employee","move_reason":"job"}'
spainlaw route_to_legal_fournier_help '{"area":"visa"}'
spainlaw --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `LEGALFOURNIER_MCP_URL` — Overrides the upstream endpoint. The default is the pinned published endpoint, `https://legalfournier.com/mcp/spain-legal`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `spainlaw error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface, asserts the three omitted tools are absent at the registry, MCP and CLI levels, exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`.
