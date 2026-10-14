# @pipeworx/epoch-ai-models

AI model facts — organization, release date, parameter count, training-compute
(FLOP) estimate, hardware, accessibility — looked up by name, searched by
organization/domain/year/compute threshold, or aggregated into a yearly
training-compute trend.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1800+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `ai_model(name)` — one model's organization, release date, parameter count,
  training-compute estimate, hardware and accessibility. Resolves dated
  variants (a query for "GPT-4" matches the dataset's "GPT-4 (Mar 2023)" row).
  An unmatched name returns `found:false` with the closest names in the
  dataset, never an empty result.
- `ai_models_search(organization?, domain?, year?, min_compute?, query?, limit?, offset?)`
  — filter the dataset by organization, domain (e.g. "Language", "Vision"),
  publication year, and/or a minimum training-compute (FLOP) threshold.
  Newest-first, capped at `limit` (default 50, max 200) with
  `count_matching`/`truncated` so a large match set is never silently cut.
- `ai_compute_trend(domain?, from_year?)` — per-year count of models with a
  compute estimate and the largest training run (FLOP) that year, optionally
  filtered by domain and a starting year.

## Auth

Keyless.

## Data sources

- Epoch AI, "Data on AI models" — Epoch AI's dataset of notable machine
  learning models and their organization, release date, parameters, training
  compute, hardware and accessibility.

Licence: Epoch AI publishes this data under CC BY — "free to use, distribute,
and reproduce provided the source and authors are credited under the
Creative Commons Attribution license." Every tool response carries that
attribution plus `data_as_of` (when this isolate last pulled the dataset from
Epoch AI).

## Notes

- Epoch AI names dated variants with a trailing parenthetical (e.g.
  "GPT-4 (Mar 2023)", "GPT-4 Turbo (Nov 2023)"). `ai_model` strips that
  qualifier to resolve a bare "GPT-4"-style query to the canonical release;
  when more than one dated variant shares a base name, it picks the earliest
  one that carries a training-compute estimate.
- `Organization`, `Country (of organization)`, `Domain` and
  `Organization categorization` are comma-separated multi-value columns
  upstream (a model can list more than one organization or domain); every
  tool returns these as arrays and filters by substring/membership rather
  than exact string equality.
- Not every row carries a training-compute or parameter estimate — many
  entries are notable for other reasons (benchmark results, accessibility)
  and leave those fields blank upstream. `ai_models_search`'s `min_compute`
  filter and `ai_compute_trend` both treat a blank estimate as "excluded,"
  not zero.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "epoch-ai-models": {
      "url": "https://gateway.pipeworx.io/epoch-ai-models/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/epoch-ai-models/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1800+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ai_model \
  -H 'Content-Type: application/json' \
  -d '{"name":"GPT-4"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ai_model`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "epoch-ai-models": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-epoch-ai-models"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-epoch-ai-models
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Epoch Ai Models data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
