# BUILD-REPORT — spainlaw

## Surface built (labelled subset)

Clean-room forwarding client for the published Legal Fournier Spain-legal remote (`com.legalfournier/spain-legal` v0.4.0). **Labelled subset: 10 of 13 captured tools**, exactly the fixture's 10 tools, in fixture order:

`search`, `fetch`, `get_paid_consultation_options`, `get_paid_call_availability`, `get_visa_options`, `check_beckham_eligibility`, `get_residency_path`, `explain_nie_process`, `compare_tax_regimes`, `route_to_legal_fournier_help`.

Omissions (absent at registry, MCP-layer and CLI level, asserted in tests):

- `find_legal_fournier_tool` — meta-dispatch pair member (finds unenumerated public tools)
- `run_legal_fournier_tool` — meta-dispatch pair member (executes unenumerated calculators/screeners by tool_id)
- `create_legal_fournier_contact_request` — PII lead routing (creates a lead from a person's name, email and case summary)

Full 19-file factory set per TEMPLATE.md: package.json, server.json, Dockerfile, tsconfig.json, .gitignore, package-lock.json, README.md (written fresh), LICENSE (MIT, Copyright (c) 2026 Rene Turcios), CHANGELOG.md (single 0.1.0 entry), src/{index,cli,mcp,types,forward}.ts, test/forward.test.ts, .github/workflows/{ci,publish,publish-container,registry-publish}.yml (publish-container IMAGE ghcr.io/cloudcorprecords/spainlaw, smoke asserts server name spainlaw).

Tools are registered via the MCP SDK low-level Server API (ListToolsRequestSchema / CallToolRequestSchema handlers) with explicit schema literals in src/forward.ts **generated programmatically** from the fixture (generator script in /tmp/laneb4-spainlaw-scratch/gen.py parses and re-serialises the fixture JSON; no schema JSON was retyped by hand).

## Fixture conflicts

None. The spec's required-field list matches the fixture exactly for all 10 tools (verified tool by tool: no-required for search, fetch, get_paid_consultation_options, explain_nie_process, compare_tax_regimes; from/to/timezone for get_paid_call_availability; nationality/income_source/intent for get_visa_options; employment_type/move_reason for check_beckham_eligibility; current_status/years_in_spain for get_residency_path; area for route_to_legal_fournier_help). Note, not a conflict: most fixture inputSchemas carry a `$schema` key (draft 2020-12) — the fixture governs and it is served verbatim; parity deep-equality confirms this.

## Labelled reconstructions

- Env override `LEGALFOURNIER_MCP_URL` for the pinned upstream `https://legalfournier.com/mcp/spain-legal` (labelled reconstruction per spec).
- Failure rendering as `spainlaw error:` text results (labelled reconstruction per spec).
- The registry record declares no headers: no `Authorization` header is ever sent, under any environment (asserted in tests, including with key-like env vars set).

Wire: streamable-HTTP JSON-RPC forwarding (initialize → notifications/initialized → tools/call), honouring `Mcp-Session-Id` when issued, parsing plain-JSON and SSE replies; upstream results pass through verbatim, including the upstream isError flag. Forwarding core takes an injectable fetch; all tests use a fake fetch only — no live network calls were made or claimed.

## Lockfile

- Route: CLEAN Route 3 from birth, generated in the build dir with no node_modules present: `npm install --package-lock-only --ignore-scripts --offline`, then `npm ci --ignore-scripts --offline` to populate node_modules.
- Verified: parses, name == `@reneturcios/spainlaw`, exactly **128 package entries**.
- sha256: `5df6b9a4512f80a2006ad1a96380908888b5d347408d9ab183e151c55d1de30b`

## server.json

- name: `io.github.CloudCorpRecords/spainlaw` (matches package.json mcpName and the Dockerfile label `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/spainlaw"`)
- description: "Spain legal guidance: published articles, visa routes, Beckham checks, residency and tax." — **89 characters**
- npm entry `@reneturcios/spainlaw` carries version 0.1.0; OCI entry `ghcr.io/cloudcorprecords/spainlaw:0.1.0` carries NO version field.

## Verification results (own run)

- `npx tsc --noEmit` — exit 0
- `npm run build` — exit 0
- `npm test` — **47 tests, 47 pass, 0 fail** (includes registration-integrity deep-equal against the fixture's 10 tools in order, omission absence at registry/MCP/CLI levels, no-Authorization assertions under several environments, env override, verbatim argument forwarding, result pass-through incl. upstream isError, SSE parsing, error rendering, CLI `--version` prints 0.1.0)
- Parity: `python3 harness/parity.py --original harness/fixtures/legalfournier-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/spainlaw/dist/cli.js mcp" --out builds/spainlaw/parity-out` → **verdict PASS, parity 100.0** (10/10 identical, no missing, no extra, no schema diffs)
