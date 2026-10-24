# OpenBB ODP lane — vibe-investing — 2026-10-10

**Status:** `SCHEMA_PROBE` / free tier **BLOCKED** for surprise data
**Why here:** This repo needs a broad fundamentals / macro / news surface across the
MyETF concentration work, the mNAV notes, and AIInvestor. ODP is the most complete
candidate. This records what actually ships on the free tier, measured rather than read
off the docs.

**Not** a replacement for the Zacks MCP lane in `research/zacks_mcp/`. That lane is
blocked for the *same* reason (see [Why it cannot replace Zacks](#why-it-cannot-replace-zacks)).

## Premise

Open Data Platform (ODP) is the surviving OpenBB codebase, Apache-2.0, `pip install openbb`.
It is positioned as a "connect once, consume everywhere" layer over many providers —
Python, REST (`openbb-api` on `127.0.0.1:6900`), MCP, and the Workspace UI.

That breadth is exactly what the concentration and mNAV lanes keep hitting missing in
vendor blogs and ad-hoc scrapes.

## Project status first (matters more than usual here)

| Fact | Detail |
|---|---|
| Company | OpenBB **wound down 2026-08-25** — no PMF, no buyer |
| `OpenBB-finance/*` | **all archived / read-only** since 2026-10-01 |
| `OpenBB-finance/workspace` | archived, **1 commit**, squashed dump, CI does not run |
| Steward (Workspace) | FINOS / Linux Foundation — no public repo yet as of 2026-10-10 |
| Steward (ODP) | `openbq-org/OpenBB` — non-profit, **active**, last push 2026-10-10 |

Do not clone `OpenBB-finance/workspace`. It is a dead snapshot. `openbq-org/OpenBB` is the
live repo.

## Measured behaviour

Python 3.13 venv, `pip install openbb`, then `pip install openbb-economy`. **No API keys.**

### The base package ships far less than it appears to

A base install exposes only alternative-data routers:

```
bls cboe cftc coverage deribit ecb eia famafrench federal_reserve finra fred
imf jodi nasdaq news oecd reference sec system tmx uscongress usda ustreasury
```

Everything conventional is a **separate extension**:

| Router | Base install | Extension |
|--------|--------------|-----------|
| `economy` | missing | `openbb-economy` |
| `equity` | missing | `openbb-*` provider |
| `crypto` | missing | `openbb-*` provider |
| `fx` | missing | `openbb-*` provider |
| `index` | missing | `openbb-*` provider |
| `technical` | missing | `openbb-*` provider |
| `quant` | missing | `openbb-*` provider |

Budget ~200 packages for the base install alone. Keep it in a throwaway venv.

### What the free tier actually returns

Both keyless calendar endpoints work, and both are **release schedules, not surprise feeds**:

| Endpoint | Rows | Columns |
|----------|------|---------|
| `obb.federal_reserve.release_calendar()` | 161 | `date, time, release, title, event_type` |
| `obb.economy.calendar(...)` | 43 | `country, category, event, source, reference_period` |

```
federal_reserve.release_calendar()
   date       time        release          title                          event_type
0  2026-10-13  4:15 p.m.   H.10   H.10 - Foreign Exchange Rates               Stat
1  2026-10-13  1:00 p.m.      CP           CP - Commercial Paper             Stat
2  2026-10-13  4:15 p.m.   H.15  H.15 - Selected Interest Rates              Stat
```

The Fed calendar is `event_type == "Stat"` across the board, and is dominated by routine
series (Commercial Paper, H.10, H.15) rather than market-moving releases. Useful as a
timing reference, not as an event tape.

## Missing vs a KEEP path

Consensus and actual values are **not on the free tier**. They sit behind a paid provider.
That means there is no `estimate`, `actual`, `previous`, or `surprise` field on either
calendar endpoint.

Missing vs a KEEP path:

- consensus / actual / surprise — **absent on free tier**
- `estimate_observed_ts` (when consensus was captured) — absent
- announcement timezone normalisation — partial only (`time` is a loose `"4:15 p.m."` string)
- point-in-time / backdated history — absent

## Why it cannot replace Zacks

`research/zacks_mcp/LANE_CONTRACT_2026-08-22.md` states:

> Do not use this MCP as an earnings-surprise tape.

ODP has the **same** limitation on the free tier, from the opposite direction: Zacks blocks
because it serves *current* holdings without dated history; ODP blocks because it serves a
schedule without any consensus at all. Neither yields `estimate_observed_ts`.

So the surprise gap is **not solved** by ODP. Any surprise work still needs the paid tier or
a separate consensus source.

## What ODP *is* good for, keyless

- `obb.fred.*` — FRED macro series (commodity / economy / fixedincome)
- `obb.federal_reserve.*` — Fed datasets, including `release_calendar`
- `obb.eia`, `obb.bls`, `obb.ecb`, `obb.oecd`, `obb.imf` — public statistical agencies
- `obb.news` — news surface
- `obb.sec` — filings surface

The public-agency routers are the genuinely free, genuinely useful part. That is a real gap
this repo does not currently have a single tool for.

## Verdict

Do **not** wire ODP into any lane on the free tier.

- For **surprise** work: blocked. Same wall as the Zacks lane, no workaround.
- For **public-agency macro series** (FRED/Fed/BLS): viable, and the cheapest single
  dependency for it. Prefer direct `fredapi` / agency endpoints for a single series —
  the ~200-package tree is not worth it until several providers are actually in use.
- For **Workspace UI**: not usable. The repo is archived and the hosted product is gone.

## Revisit condition

Re-open if any of:

1. A paid provider budget is approved **and** `estimate_observed_ts` / point-in-time
   consensus provenance is part of the offer.
2. More than ~3 public-agency sources are genuinely needed — at that point ODP's
   normalisation earns its dependency weight.
3. The FINOS-stewarded Workspace repo appears publicly with a runnable build.

## Working notes (reproduce)

```bash
uv venv /tmp/openbb-env
VIRTUAL_ENV=/tmp/openbb-env uv pip install openbb
VIRTUAL_ENV=/tmp/openbb-env uv pip install openbb-economy   # for obb.economy
```

`federal_reserve.release_calendar()` returns a plain `list`, **not** an OBB result object —
no `.to_dataframe()`. Wrap with `pd.DataFrame(...)` before inspecting. This is an easy
trap if you assume the usual OBB dataframe interface.

## References

- ODP: <https://github.com/openbq-org/OpenBB> (active)
- Archived workspace: <https://github.com/OpenBB-finance/workspace> (read-only, do not clone)
- Steward proposal: <https://github.com/finos/community/issues/445>
- Zacks lane: `research/zacks_mcp/LANE_CONTRACT_2026-08-22.md`