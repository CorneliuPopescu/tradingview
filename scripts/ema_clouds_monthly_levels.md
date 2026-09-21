# Ripster EMA Clouds + Monthly Levels (`EMA+MLV`)

One overlay indicator, Pine Script v6, in `ema_clouds_monthly_levels.pine`.
It bundles four things that used to be separate studies or manual drawings:

1. Ripster EMA clouds
2. Monthly high / low rays with month labels
3. Three anchored VWAPs (daily, monthly, quarterly)
4. A VWAP tendency table

Works on any symbol and any timeframe. Futures (CME session times) are handled
correctly; see the notes at the end.

## 1. Ripster EMA clouds

Port of ripster47's EMA Clouds (v4 -> v6), unchanged in behaviour.

- Five pairs of EMAs (defaults 8/9, 5/12, 34/50, 72/89, 180/200) on `hl2`.
- Each pair is drawn as a filled cloud. Green tint when the short EMA is above
  the long one, red tint when below.
- Clouds 1 to 3 are on by default, 4 and 5 off.
- "Display EMA Line" draws the EMA lines themselves (off by default).
- The EMA lines never add tags on the price axis.

## 2. Monthly levels

Horizontal rays for the month high / low structure, extended to the right.

- **Current month high and low.** Always drawn.
- **Higher highs / lower lows.** Walking back in time from the current month,
  the next month whose high is higher than everything since gets a ray; same
  for lows. "Extra higher highs / lower lows" sets how many of each (default 2,
  so up to 6 rays in total).
- **All-time high / low** get their own colour (default green). All other
  levels use the level colour (default red).
- Each ray starts on the first bar of its month and carries a label such as
  `Sep-26`. High labels sit above the ray, low labels below, a few bars to the
  right of the last bar ("Label offset").
- Each level also prints a price tag on the axis, in the ray's colour.

The monthly data comes from `request.security(..., "M", ...)`, so the same
levels appear on every timeframe.

## 3. Anchored VWAPs

Three VWAP curves on `hlc3`, thinnest line width, no axis tag.

| Curve     | Restarts on                      | Default colour            | Shown on          |
|-----------|----------------------------------|---------------------------|-------------------|
| Daily     | first bar of each trading day    | blue `#2962ff`            | intraday charts   |
| Monthly   | first bar of each month          | yellow `#ffeb3b`, 50% transparent | all charts |
| Quarterly | first bar of each quarter        | magenta `#ff00ff`         | all charts        |

The anchors are `timeframe.change("D")`, `("M")` and `("3M")`, the same rule
the built-in TradingView VWAP uses.

In the first month of a quarter the monthly and quarterly curves are the same
line (same start bar, same sums). The monthly one is drawn on top and half
transparent, so the magenta shows through it during that month. From the
second month on they split.

On a monthly chart the monthly VWAP is one bar long and the quarterly one three
bars; they only make sense on daily and intraday charts.

## 4. VWAP tendency table

A small table in a chart corner (default top right) with one row per VWAP:
`3M`, `M`, and on intraday charts `D`.

### Formula

For each bar in the period, with `vwap` the running anchored VWAP on that bar:

```
num += volume * (hlc3 - vwap)
den += volume
tendency = num / den / vwap * 100      (percent)
```

Both sums reset on the same bar the VWAP restarts. Green text when positive,
red when negative, two decimals.

### How to read it

The VWAP is the average price paid so far in the period. The tendency says how
far, on a volume-weighted basis, trading happened above or below that average
while it was forming.

- **Positive**: price kept running ahead of its own average. Buyers kept
  paying more than the average so far. Steady uptrend.
- **Negative**: the mirror image. Steady downtrend.
- **Near zero** (within about 0.2%): chop around the average.
- **0.2% to 1%**: a clear trend, sign gives the direction.
- **Above 1%**: a strong, one-sided period.

Rough size guide: for a straight, steady move with even volume the number is
about a quarter of the net move, so `+0.85%` points to a trend of roughly +3%
so far in the period. A move that fades back to the average pulls the number
toward zero, so it rewards moves that hold, not spikes.

Two properties to keep in mind:

- Heavy volume far from the average moves the number a lot; thin volume barely
  moves it.
- Early bars weigh more. The average is young then and price gets far from it
  fast. Late in the period the average has caught up and new bars add little.

The value depends on the chart bars. `M` on a 15-minute chart and `M` on a
daily chart differ a little for the same month.

Scale the thresholds above to the asset: a quiet stock reaches 0.5% less often
than NQ does.

## Settings summary

| Group           | Inputs                                                             |
|-----------------|--------------------------------------------------------------------|
| (EMA clouds)    | MA type, 10 lengths, source, show line, cloud 1-5 on/off, leading  |
| Anchored VWAP   | Daily / Monthly / Quarterly on/off, three colours                  |
| VWAP tendency   | Show table, table corner                                            |
| Monthly levels  | Show levels, extra highs/lows (0-5), level colour, ATH/ATL colour, label colour, label offset |

## Futures notes

- A CME monthly bar opens at 17:00 Chicago on the last calendar day of the
  previous month (e.g. the September bar opens Aug 31). `month(time)` on that
  bar returns August. The month label is therefore taken from `time_close`,
  which is always inside the right month. The ray itself still starts at the
  bar open time, which lands on the first session of the month on any chart.
- `time_tradingday` is **not** usable here: on a month bar it returns the
  latest trading day inside the bar, not the first, so rays would start at the
  month end and the current-month ray would point left.
- The daily VWAP restarts at the session open (17:00 Chicago for CME), same as
  the built-in VWAP.

## Workflow

Edit `ema_clouds_monthly_levels.pine`, inject it into the TradingView Pine
Editor through the MCP (`pine_set_source` + `pine_smart_compile`), then check a
fresh `capture_screenshot`. The script is saved in the TradingView library under
the same name.
