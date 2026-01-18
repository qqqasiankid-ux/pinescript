# Technical Analysis Functions (Selected)

## Status

- **STATUS: OBSERVED** – Content derived from the user‑provided `ok` repository.  Not all statements have been formally verified against TradingView's official Pine Script v6 documentation.  Use with caution.
- **CONFIDENCE:** MEDIUM
- **LAST_UPDATED:** 2026‑01‑18

## Scope

This document provides concise descriptions, return values, remarks, examples, and common pitfalls for a curated set of frequently used technical analysis functions in Pine Script v6.  The goal is to populate the existing stub with verified, concrete information without overwhelming the reader with the entire universe of `ta.*` functions.  For a complete list, refer to the full TradingView documentation.

---

## Canonical Rules

While functions in the `ta` namespace are user‑friendly wrappers around standard mathematical procedures, they still obey Pine Script rules.  The following general principles apply:

1. **Versioning:** All examples must include the `//@version=6` directive, and code samples should compile under Pine Script v6.
2. **Series vs. Simple Inputs:** Functions accept series arguments (values that change each bar) unless otherwise noted.  Always pass series variables explicitly.
3. **NA Handling:** Most functions either ignore or propagate `na` values.  Check the remarks for each function.  Propagating `na` means the result will be `na` if any input element is `na`, while ignoring `na` means the function uses only non‑`na` values for its computation【815768674167105†L90-L99】.
4. **Repainting:** Some functions return values that can cause indicator repainting when used on real‑time bars (e.g., moving averages recalculated each tick).  Use caution and refer to the execution model.
5. **Resource Limits:** Complex functions such as `ta.valuewhen` require the script to execute on every bar, which can affect performance and may contribute to the “too many securities” or “script too long” compile errors.

---

## Selected Functions

The following sections describe selected `ta` functions.  Each entry includes a synopsis, return value, remarks, an example in v6 syntax, and typical pitfalls.

### `ta.sma()` – Simple Moving Average

**Synopsis:** Returns the simple moving average of `source` over the last `length` bars【815768674167105†L915-L934】.

**Returns:** The arithmetic mean of the last `length` values of `source`【815768674167105†L915-L934】.

**Remarks:**

- `na` values in the `source` series are ignored【815768674167105†L915-L934】.
- Because the SMA uses only completed bars, it can repaint on real‑time bars if used without proper handling.

**Example (v6):**

```pine
//@version=6
indicator("Simple Moving Average")
plot(ta.sma(close, 15), title="SMA")

// Equivalent implementation (inefficient)
pine_sma(x, y) =>
    float sum = 0.0
    for i = 0 to y - 1
        sum := sum + x[i] / y
    sum
plot(pine_sma(close, 15), color=color.gray)
```

**Pitfalls:**

- The SMA uses a fixed window.  Sudden spikes or gaps can distort the average, and `na` values reduce the effective sample size.
- When used on real‑time bars, recalculations may cause repainting.  To avoid this, consider using historical bars (`barstate.islastconfirmedhistory`) or the `max_bars_back()` hint.

### `ta.ema()` – Exponential Moving Average

**Synopsis:** Returns the exponentially weighted moving average (EWMA) of `source` with smoothing factor `alpha = 2/(length + 1)`【815768674167105†L337-L357】.

**Returns:** Exponential moving average of `source` for the specified length【815768674167105†L337-L357】.

**Remarks:**

- `na` values in the `source` are ignored【815768674167105†L337-L357】.
- Because the EMA gives more weight to recent values, it reacts faster to changes than the SMA.
- Calling `ta.ema()` can cause indicator repainting on real‑time bars; use `barstate.islastconfirmedhistory` when plotting to lock in values.

**Example (v6):**

```pine
//@version=6
indicator("Exponential Moving Average")
plot(ta.ema(close, 15), title="EMA")

// Equivalent implementation (less efficient)
pine_ema(src, length) =>
    float alpha = 2 / (length + 1)
    float sum = 0.0
    sum := na(sum[1]) ? src : alpha * src + (1 - alpha) * nz(sum[1])
    sum
plot(pine_ema(close, 15), color=color.gray)
```

**Pitfalls:**

- The EMA depends on all previous bars and thus requires `max_bars_back()` if used within functions or loops.
- Because the first value is undefined until enough bars have passed, early results may be `na`.

### `ta.atr()` – Average True Range

**Synopsis:** Returns the smoothed moving average of the true range, which measures volatility【815768674167105†L35-L57】.

**Returns:** Average true range for the specified `length`【815768674167105†L35-L57】.

**Remarks:**

- The true range is defined as `max(high - low, abs(high - close[1]), abs(low - close[1]))`【815768674167105†L35-L57】.
- `na` values in the input series are ignored【815768674167105†L35-L57】.
- ATR is often used to position stop‑loss orders; larger ATR means higher volatility.

**Example (v6):**

```pine
//@version=6
indicator("Average True Range")
plot(ta.atr(14), title="ATR")

// Equivalent implementation
pine_atr(length) =>
    float trueRange = na(high[1]) ? high - low : math.max(math.max(high - low, math.abs(high - close[1])), math.abs(low - close[1]))
    ta.rma(trueRange, length)
plot(pine_atr(14), color=color.gray)
```

**Pitfalls:**

- ATR does not convey direction; it measures only volatility.  Use in combination with trend indicators for trading decisions.
- In scripts that change timeframes using `request.security()`, ensure you set `gaps=false` or adjust for higher‑resolution data to avoid inflated ranges.

### `ta.rsi()` – Relative Strength Index

**Synopsis:** Returns the RSI of `source`, which compares the magnitude of recent gains to recent losses over a period【815768674167105†L809-L833】.

**Returns:** RSI value ranging from 0 to 100【815768674167105†L809-L833】.

**Remarks:**

- `na` values in the `source` are ignored【815768674167105†L809-L833】.
- RSI uses the smoothed moving average `ta.rma()` internally【815768674167105†L809-L833】.

**Example (v6):**

```pine
//@version=6
indicator("Relative Strength Index")
plot(ta.rsi(close, 14), title="RSI")

// Manual implementation
pine_rsi(x, length) =>
    float up = math.max(x - x[1], 0)
    float dn = math.max(x[1] - x, 0)
    float rs = ta.rma(up, length) / ta.rma(dn, length)
    100 - 100 / (1 + rs)
plot(pine_rsi(close, 14), color=color.gray)
```

**Pitfalls:**

- Traditional overbought/oversold thresholds (70/30) may not apply in all markets.  Interpret RSI in context.
- Using RSI on small `length` values can produce erratic signals.

### `ta.macd()` – Moving Average Convergence/Divergence

**Synopsis:** Calculates the MACD line (difference between two EMAs), signal line (EMA of MACD), and histogram【815768674167105†L523-L546】.

**Returns:** A tuple `[macdLine, signalLine, histLine]`【815768674167105†L523-L546】.

**Remarks:**

- By default uses `lengthFast = 12`, `lengthSlow = 26`, and `signalSmoothing = 9` bars.
- `na` values in `source` are ignored【815768674167105†L523-L546】.

**Example (v6):**

```pine
//@version=6
indicator("MACD")
[macdLine, signalLine, histLine] = ta.macd(close, 12, 26, 9)
plot(macdLine, color=color.blue)
plot(signalLine, color=color.orange)
plot(histLine, color=color.red, style=plot.style_histogram)
```

**Pitfalls:**

- MACD signals can lag in choppy markets.  Crossovers alone are not sufficient; confirm with price action.
- Because it relies on EMAs, MACD inherits the repainting behaviour discussed in the `ta.ema()` section.

### `ta.bb()` and `ta.bbw()` – Bollinger Bands and Band Width

**Synopsis:** `ta.bb()` returns the middle (moving average), upper, and lower Bollinger Bands based on a simple moving average and standard deviation【815768674167105†L82-L114】.  `ta.bbw()` returns the width of the Bollinger Bands relative to the middle band【815768674167105†L117-L141】.

**Returns:**

- `ta.bb(source, length, mult)` returns `[middle, upper, lower]`【815768674167105†L82-L114】.
- `ta.bbw(source, length, mult)` returns the band width in percentage terms【815768674167105†L117-L141】.

**Remarks:**

- `na` values in the `source` are ignored【815768674167105†L82-L114】【815768674167105†L117-L141】.
- Bollinger Bands adapt to volatility; narrower bands indicate low volatility and vice versa.

**Example (v6):**

```pine
//@version=6
indicator("Bollinger Bands", overlay=true)
[middle, upper, lower] = ta.bb(close, 20, 2)
plot(middle, color=color.yellow)
plot(upper, color=color.green)
plot(lower, color=color.red)

plot(ta.bbw(close, 20, 2), title="BB Width", color=color.blue, overlay=false)
```

**Pitfalls:**

- Interpreting price crossing the bands as trading signals can be unreliable; combine with other indicators.
- When market volatility spikes, band width expands; using fixed threshold values for band width may lead to false signals.

### `ta.vwap()` – Volume‑Weighted Average Price

**Synopsis:** Calculates the volume‑weighted average price for a series, optionally with an anchor and standard deviation multiplier to create bands【815768674167105†L1120-L1150】.

**Returns:** A single VWAP series or a tuple `[vwap, upper_band, lower_band]` if a standard deviation multiplier is provided【815768674167105†L1120-L1150】.

**Remarks:**

- Calculations begin only when the anchor condition is true; before that, the function returns `na`【815768674167105†L1120-L1150】.
- VWAP resets at the specified anchor (e.g., daily, weekly, monthly) and is often used by institutional traders to benchmark execution quality.

**Example (v6):**

```pine
//@version=6
indicator("Simple VWAP")
myVwap = ta.vwap(open)
plot(myVwap)

//@version=6
indicator("Anchored VWAP")
vwapAnchorInput = input.string("Daily", "Anchor", options=["Daily", "Weekly", "Monthly"])
stdevMultiplierInput = input.float(1.0, "StdDev Mult")
anchorTimeframe = switch vwapAnchorInput
    "Daily"   => "1D"
    "Weekly"  => "1W"
    "Monthly" => "1M"
anchor = timeframe.change(anchorTimeframe)
[vwapSeries, upper, lower] = ta.vwap(open, anchor, stdevMultiplierInput)
plot(vwapSeries)
plot(upper, color=color.green)
plot(lower, color=color.green)
```

**Pitfalls:**

- Anchored VWAP resets at the beginning of each anchor period; failure to set `anchor` properly will produce unexpected results.
- Using large `stdevMultiplierInput` values can produce extremely wide bands that are difficult to interpret.

### `ta.supertrend()` – Supertrend Indicator

**Synopsis:** A trend‑following indicator that outputs both a supertrend line and direction of trend【815768674167105†L995-L1037】.

**Returns:** `[supertrend, direction]` where `direction` is 1 for down trends and –1 for up trends【815768674167105†L995-L1037】.

**Remarks:**

- Combines ATR and a factor to compute upper and lower bands.  When the price crosses these bands, the direction flips【815768674167105†L995-L1037】.
- The result can cause repainting on real‑time bars; confirm signals using `barstate.islastconfirmedhistory`.

**Example (v6):**

```pine
//@version=6
indicator("Supertrend", overlay=true)
[super, dir] = ta.supertrend(3, 10)
plot(dir < 0 ? super : na, "Up", color=color.green, style=plot.style_linebr)
plot(dir > 0 ? super : na, "Down", color=color.red, style=plot.style_linebr)
```

**Pitfalls:**

- Whipsaw markets can cause rapid alternation between up and down directions.  Adjust the `factor` and `atrPeriod` parameters to reduce false signals.
- Because the indicator uses ATR, the same considerations about `na` and volatility apply.

### `ta.cross()`, `ta.crossover()`, `ta.crossunder()` – Cross Detection

**Synopsis:** Detect when two series intersect.  `ta.cross()` returns true on the bar where the series cross; `ta.crossover()` returns true when the first series crosses above the second; `ta.crossunder()` returns true when the first crosses below the second【815768674167105†L252-L274】.

**Returns:** Boolean series indicating whether a cross occurred on the current bar【815768674167105†L252-L274】.

**Remarks:**

- The functions look at the current and previous bar values to detect crossings【815768674167105†L259-L274】.
- `na` values propagate; ensure both series have valid values.

**Example (v6):**

```pine
//@version=6
indicator("Cross Detection")
fastMA = ta.ema(close, 10)
slowMA = ta.ema(close, 30)
plot(fastMA, color=color.blue)
plot(slowMA, color=color.orange)

// Plot arrows on crossovers and crossunders
plotshape(ta.crossover(fastMA, slowMA), title="Bullish Cross", style=shape.triangleup,  location=location.belowbar, color=color.green)
plotshape(ta.crossunder(fastMA, slowMA), title="Bearish Cross", style=shape.triangledown, location=location.abovebar, color=color.red)
```

**Pitfalls:**

- Cross functions do not detect multiple crossings within a single bar.  On intraday charts with long bars, important intra‑bar crosses can be missed.
- Because these functions rely on the previous bar, they can introduce one‑bar lag in detection.

### `ta.valuewhen()` – Value on Condition

**Synopsis:** Returns the value of a series when a condition was true on the N‑th most recent occurrence【815768674167105†L1089-L1104】.

**Returns:** The value of `source` at the bar where the condition was true, or `na` if the condition has never been met【815768674167105†L1089-L1104】.

**Remarks:**

- This function requires the script to execute on every bar because it must track all occurrences of the condition【815768674167105†L1089-L1104】.
- Using `ta.valuewhen()` inside loops or nested scopes can lead to performance issues and unexpected behaviour.
- `ta.valuewhen()` can cause indicator repainting, as the value returned for the current bar may change when subsequent occurrences are processed【815768674167105†L1089-L1104】.

**Example (v6):**

```pine
//@version=6
indicator("Value When Example")
slowMA = ta.sma(close, 14)
fastMA = ta.sma(close, 7)
// Plot close price on the second most recent crossover between fast and slow MAs
plot(ta.valuewhen(ta.cross(slowMA, fastMA), close, 1), title="Close at previous cross", style=plot.style_stepline)
```

**Pitfalls:**

- Because `ta.valuewhen()` depends on past occurrences, misusing it inside loops or conditional scopes can cause compile errors or unexpected results.
- Frequent use can slow down script execution, especially on long histories.

---

## References

- Documentation and examples adapted from the user‑provided `ok` repository’s `reference/functions/ta.md` file【815768674167105†L82-L114】【815768674167105†L337-L357】【815768674167105†L809-L833】.
- Pine Script v6 official documentation: [TradingView Pine Script Reference](https://www.tradingview.com/pine-script-docs/en/v6/).