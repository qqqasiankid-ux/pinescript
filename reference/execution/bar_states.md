# Bar States
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents the `barstate.*` built-in variables in Pine Script v6. These variables provide information about the current bar's position and state in the script's execution cycle.

## Canonical Rules
- All `barstate.*` variables have type `series bool`
- `barstate.*` variables are read-only and cannot be modified
- Bar state changes can affect script behavior and cause repainting
- Understanding bar states is critical for avoiding repainting indicators
- Bar states are evaluated on every script execution for each bar

## barstate Variables

### barstate.isconfirmed (series bool)

Returns `true` if the current bar is closed and confirmed, `false` otherwise.

**Example (v6):**
```pine
//@version=6
indicator("Bar Confirmed", overlay=true)

// Only plot on confirmed bars
if barstate.isconfirmed
    label.new(bar_index, high, "Confirmed", color=color.green, style=label.style_label_down)
```

**Repainting Warning:** Using `barstate.isconfirmed` in conditions will cause the script to behave differently on real-time vs. historical bars, potentially causing repainting.

---

### barstate.isfirst (series bool)

Returns `true` on the first bar of the dataset (bar_index == 0), `false` otherwise.

**Example (v6):**
```pine
//@version=6
indicator("First Bar Init", overlay=true)

var float startPrice = na

// Initialize on first bar only
if barstate.isfirst
    startPrice := close

plot(startPrice, "Start Price", color=color.blue)
```

**Repainting Warning:** Generally safe, as the first bar is always historical. However, if the dataset changes (e.g., adding more history), initialization may change.

---

### barstate.ishistory (series bool)

Returns `true` if the current bar is a historical bar (not real-time), `false` otherwise.

**Example (v6):**
```pine
//@version=6
indicator("Historical vs Realtime", overlay=true)

bgcolor(barstate.ishistory ? color.new(color.blue, 90) : color.new(color.red, 90))
```

**Repainting Warning:** Behavior differs between historical and real-time bars. Scripts that use this for logic decisions will behave differently in real-time vs. history.

---

### barstate.islast (series bool)

Returns `true` on the last bar of the dataset (the current real-time bar or the last historical bar), `false` otherwise.

**Example (v6):**
```pine
//@version=6
indicator("Last Bar Label", overlay=true)

if barstate.islast
    label.new(bar_index, high, "Last Bar", color=color.red, style=label.style_label_down)
```

**Repainting Warning:** `barstate.islast` changes as new bars form. Code executed only on `barstate.islast` will produce different results on historical replay vs. real-time execution.

---

### barstate.islastconfirmedhistory (series bool)

Returns `true` on the last confirmed historical bar before real-time bars begin, `false` otherwise.

**Example (v6):**
```pine
//@version=6
indicator("Last Historical Bar", overlay=true)

var float lastHistoricalClose = na

if barstate.islastconfirmedhistory
    lastHistoricalClose := close
    label.new(bar_index, high, "Last History", color=color.orange, style=label.style_label_down)

plot(lastHistoricalClose, "Last Historical Close")
```

**Repainting Warning:** Generally stable, but the boundary between historical and real-time can shift if the script is reloaded or the chart is refreshed.

---

### barstate.isnew (series bool)

Returns `true` on the first execution of the script on a bar (when a new bar opens), `false` on subsequent executions on the same bar.

**Example (v6):**
```pine
//@version=6
indicator("New Bar Detection", overlay=true)

var int barCount = 0

if barstate.isnew
    barCount += 1

plot(barCount, "Bar Count")
```

**Repainting Warning:** `barstate.isnew` is `true` only once per bar. On historical bars, it's always `true` because the script runs once per bar. On real-time bars, it's `true` only when the bar first opens, causing potential repainting if used in alert conditions or decisions.

---

### barstate.isrealtime (series bool)

Returns `true` if the current bar is updating in real-time, `false` if it's a historical bar.

**Example (v6):**
```pine
//@version=6
indicator("Realtime Indicator", overlay=true)

// Different behavior for real-time bars
color barColor = barstate.isrealtime ? color.red : color.gray
plotcandle(open, high, low, close, color=barColor)
```

**Repainting Warning:** Scripts that behave differently based on `barstate.isrealtime` will repaint when real-time bars become historical. This is a common source of repainting in strategies and alerts.

---

## Usage Patterns

### Pattern 1: One-Time Initialization
```pine
//@version=6
indicator("Init Pattern", overlay=true)

var initialized = false
var float baseline = na

if barstate.isfirst and not initialized
    baseline := close
    initialized := true

plot(baseline)
```

### Pattern 2: Last Bar Calculations
```pine
//@version=6
indicator("Last Bar Only", overlay=true)

var table resultsTable = table.new(position.top_right, 2, 1)

if barstate.islast
    // Update table only on last bar for efficiency
    table.cell(resultsTable, 0, 0, "Close", bgcolor=color.gray)
    table.cell(resultsTable, 1, 0, str.tostring(close, "#.##"), bgcolor=color.white)
```

### Pattern 3: Non-Repainting Logic
```pine
//@version=6
indicator("Non-Repainting", overlay=true)

// Use confirmed closes only
confirmed_close = barstate.isconfirmed ? close : close[1]

// This will use the previous bar's close on real-time bars until confirmed
plot(confirmed_close, "Confirmed Close")
```

## Pitfalls / Failure Modes

### Pitfall 1: Using barstate.isnew for Alerts
- **Problem:** Alerts triggered with `barstate.isnew` conditions only fire when bar opens
- **Impact:** Misses intrabar price movements; different behavior in real-time vs. history
- **Prevention:** Understand that historical bars always have `barstate.isnew == true`; use confirmation logic

### Pitfall 2: Mixing Historical and Realtime Logic
- **Problem:** Script behaves differently based on `barstate.ishistory` or `barstate.isrealtime`
- **Impact:** Repainting; strategies show different results in backtesting vs. live
- **Prevention:** Keep logic consistent across bar states; use confirmed data only

### Pitfall 3: Overusing barstate.islast
- **Problem:** Complex calculations only on `barstate.islast` skip historical validation
- **Impact:** Undetected bugs in historical logic; performance issues if too complex
- **Prevention:** Test logic on historical bars; use `barstate.islast` sparingly for display/tables only

### Pitfall 4: Assuming barstate.isfirst is Stable
- **Problem:** Dataset changes (more history added) can change which bar is "first"
- **Impact:** Initialization values change; breaking changes in script behavior
- **Prevention:** Use `var` for persistence; document assumptions about data availability

### Pitfall 5: Repainting with barstate.isconfirmed
- **Problem:** Logic that changes behavior on confirmed vs. unconfirmed bars
- **Impact:** Indicators show different values in real-time vs. after bar closes
- **Prevention:** Use confirmed data consistently; document repainting behavior

### Pitfall 6: Performance Issues with Real-time Checks
- **Problem:** Expensive operations inside `if barstate.isrealtime` run on every tick
- **Impact:** Script slowdown; may hit execution time limits
- **Prevention:** Minimize real-time calculations; use `barstate.isnew` to limit updates

## References
- TradingView Pine Script v6 Bar States: https://www.tradingview.com/pine-script-docs/language/Bar-states
- Pine Script Execution Model: https://www.tradingview.com/pine-script-docs/language/Execution-model
- Avoiding Repainting: https://www.tradingview.com/pine-script-docs/writing/Repainting
- bar_index Built-in: https://www.tradingview.com/pine-script-docs/concepts/Bar-index
