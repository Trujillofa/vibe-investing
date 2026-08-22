# Zacks MCP lane — vibe-investing — 2026-08-22

**Status:** `SCHEMA_PASS` / alpha **BLOCKED**  
**Why here:** MyETF concentration work and mNAV notes need a dated, attributed
fundamentals + ETF-holdings source. This is that source. It is not a new
perp bot and not TipRanks/Finviz scrapes.

## Premise

Official Zacks MCP (`zacks` in `~/.cursor/mcp.json` → `https://mcp.zacksdata.com`)
returns point-in-time North American statements and **current** ETF holdings.
That is a new data edge for:

- MyETF overlap / concentration vs issued ETFs (SOXX, SMH, QQQ, SPY)
- mNAV / listed-crypto-equity context (IBIT, GBTC, MSTR, COIN) as holdings
  **as of today**, not as a backtest

## Why this is new

Prior notes in this repo mix vendor blogs and manual mNAV observations. They
do not have a locked MCP schema, a KEEP bar, or a ban on treating a same-day
holdings print as history.

## 2026-08-22 schema probe

Source: Zacks Investment Research. No statement values or weights are stored
in this file.

| Tool | History |
|---|---|
| `get_company_snapshot` / income / balance / cash | Live probe: 5 annual years (AAPL `periods=40` still returned 2021–2025) |
| `get_etf_holdings` | `symbol` + `top_n` only; SPY/GDX `as_of` = 2026-08-22 |

Missing vs a KEEP path: >=10y statements, dated holdings history, prices,
costs, and a written redistribution pin.

Missing vs PEAD: `estimate_observed_ts`, announcement timezone. Do not use
this MCP as an earnings-surprise tape.

## Config (do not commit secrets)

```json
{
  "mcpServers": {
    "zacks": {
      "type": "http",
      "url": "https://mcp.zacksdata.com"
    }
  }
}
```

User-level `~/.cursor/mcp.json` is enough. Do not add MT5 LAN tokens to this
repo.

## Pass / stop

- **Pass (`DATA_PASS`):** licensed extract with dated holdings **or** >=10y
  PIT statements, plus a price/cost source, plus license to pin locally.
- **Stop:** treat today's holdings as a historical rebalance signal; publish
  live Zacks numbers without attribution; write a strategy before `DATA_PASS`.

## First allowed step

Desk-probe a **current** ETF (QQQ / SOXX / IBIT) through the MCP and write
overlap notes with as-of date + "Source: Zacks Investment Research". No
backtest until history exists.
