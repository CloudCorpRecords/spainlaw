# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the published Legal Fournier Spain-legal MCP remote. The ten built captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search`, `fetch`, `get_paid_consultation_options`, `get_paid_call_availability`, `get_visa_options`, `check_beckham_eligibility`, `get_residency_path`, `explain_nie_process`, `compare_tax_regimes`, `route_to_legal_fournier_help`. This is a labelled subset of the captured 13-tool surface: `find_legal_fournier_tool` and `run_legal_fournier_tool` (a meta-dispatch pair over unenumerated tools) and `create_legal_fournier_contact_request` (a PII lead-routing tool) are omitted, and their absence is asserted in the tests at the registry, MCP and CLI levels.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `spainlaw error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LEGALFOURNIER_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
