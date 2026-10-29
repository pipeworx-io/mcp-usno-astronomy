# @pipeworx/usno-astronomy

Solar eclipses, local eclipse circumstances, Moon phases and the seasons from
the US Naval Observatory's Astronomical Applications API. Answers "when is the
next solar eclipse", "what time does the eclipse start in Madrid", "when is the
next full moon" and "when is the summer solstice" from the official ephemeris.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1857+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `usno_solar_eclipses(year?)` — the solar eclipses in a year with date and
  type (total, annular, partial, hybrid). With no arguments it returns the
  next eclipse after today. Lunar eclipses are not published by this API.
- `usno_eclipse_circumstances(date, lat, lon, height?)` — what a solar eclipse
  looks like from one place: magnitude, obscuration, duration, and the UT
  time, Sun altitude and azimuth of each contact. `found:false` with
  `reason:"not_visible"` when the eclipse cannot be seen from there.
- `usno_moon_phases(year? | date?, count?)` — every New Moon, First Quarter,
  Full Moon and Last Quarter in a year, or the next `count` phases from a
  date, plus `next_full_moon` / `next_new_moon`.
- `usno_seasons(year?)` — equinoxes, solstices, perihelion and aphelion for a
  year, with northern-hemisphere season names attached.

## Auth

Keyless.

## Data sources

- <https://aa.usno.navy.mil/api/eclipses/solar/year?year=2026> — solar eclipses in a year.
- <https://aa.usno.navy.mil/api/eclipses/solar/date?date=2026-08-12&coords=40.42,-3.70&height=0> — local circumstances.
- <https://aa.usno.navy.mil/api/moon/phases/year?year=2026> and `/api/moon/phases/date?date=YYYY-MM-DD&nump=N` — Moon phases.
- <https://aa.usno.navy.mil/api/seasons?year=2026> — seasons.

Things the next person would otherwise rediscover:

- **All times are UT1**, never local. Every response carries `time_note`.
- Years are limited to **1800–2050**; USNO refuses others with an HTTP 200
  whose body is `{"error": ...}`. Every error, including "eclipse not visible
  from selected location", comes back as **200 + `error`**, so the pack reads
  the body rather than the status.
- `height` must be an **integer** number of metres; a decimal is rejected.
- The `loc=` place-name form of the circumstances endpoint answers "Please
  supply valid coordinates" — pass `coords=lat,lon` and resolve place names
  first.
- There is **no lunar eclipse endpoint** (`/api/eclipses/lunar/*` is 404).
- USNO's front end can take 3 s to accept a TCP connection and 6 s more for
  TLS (measured 2026-09-10, 6–22 s per call). The gateway caches this pack's
  answers for a week, which is honest for a fixed ephemeris.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "usno-astronomy": {
      "url": "https://gateway.pipeworx.io/usno-astronomy/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/usno-astronomy/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1857+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/usno_solar_eclipses \
  -H 'Content-Type: application/json' \
  -d '{}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/usno_solar_eclipses`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-usno-astronomy` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/usno-astronomy/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "usno-astronomy": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-usno-astronomy"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Usno Astronomy data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
