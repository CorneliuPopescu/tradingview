# tradingview
TradingView Scripting

## TradingView MCP

- Server: `tradingview-mcp/` (cloned from https://github.com/tradesdontlie/tradingview-mcp, gitignored; run `git clone` + `npm install` there on a fresh checkout).
- `.mcp.json` runs it with Windows `node.exe` so it can reach TradingView Desktop on `127.0.0.1:9222`.
- Start TradingView with the debug port first (from WSL):

```bash
cmd.exe /c start "" "C:\Program Files\WindowsApps\TradingView.Desktop_3.4.1.8194_x64__n534cwy3pjxzj\TradingView.exe" --remote-debugging-port=9222
```

## Scripts

- `scripts/ema_clouds_monthly_levels.pine` — Ripster EMA clouds + monthly high/low rays + anchored VWAPs (D/M/3M) + VWAP tendency table. Explained in [`scripts/ema_clouds_monthly_levels.md`](scripts/ema_clouds_monthly_levels.md).
