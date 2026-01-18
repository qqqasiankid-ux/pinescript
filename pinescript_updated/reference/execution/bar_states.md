# Bar States

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: qqqasiankid-ux/ok repository (reference/variables.md)

## Scope

This file documents all `barstate.*` built-in variables in Pine Script v6, which identify the state of the bar the script is executing on.

---

## Canonical Rules

- All barstate variables return `series bool` type
- These variables can cause indicator repainting when used
- Behavior differs between historical and realtime bars
- NOT recommended to use `barstate.isconfirmed` in `request.security` expressions

---

## barstate.isconfirmed

**Type:** series bool

Returns `true` if the script is calculating the last (closing) update of the current bar. The next script calculation will be on new bar data.

### Pitfall
Code using this variable could calculate differently on history and real-time data. It is NOT recommended to use in `request.security` expressions.

---

## barstate.isfirst

**Type:** series bool

Returns `true` if the current bar is the first bar in the barset, `false` otherwise.

### Pitfall
Can cause indicator repainting. Calculates differently on history vs real-time.

---

## barstate.ishistory

**Type:** series bool

Returns `true` if current bar is a historical bar, `false` otherwise.

### Pitfall
Can cause indicator repainting.

---

## barstate.islast

**Type:** series bool

Returns `true` if current bar is the last bar in barset, `false` otherwise. This condition is `true` for all real-time bars.

### Pitfall
Can cause indicator repainting.

---

## barstate.islastconfirmedhistory

**Type:** series bool

Returns `true` if script is executing on the dataset's last bar when market is closed, or on the bar immediately preceding the real-time bar if market is open.

### Pitfall
Can cause indicator repainting.

---

## barstate.isnew

**Type:** series bool

Returns `true` if script is currently calculating on a new bar, `false` otherwise. This variable is `true` when calculating on historical bars or on the first update of a newly generated real-time bar.

### Pitfall
Can cause indicator repainting.

---

## barstate.isrealtime

**Type:** series bool

Returns `true` if current bar is a real-time bar, `false` otherwise.

### Pitfall
Can cause indicator repainting.

---

## Example (v6)

```pine
//@version=6
indicator("Bar States Demo", overlay = true)

// Highlight realtime bars
bgcolor(barstate.isrealtime ? color.new(color.purple, 80) : na, title = "Realtime highlight")

// Show label on first bar
if barstate.isfirst
    label.new(bar_index, high, "First Bar", color = color.green)

// Execute logic only on confirmed bars
if barstate.isconfirmed
    // Safe to use values that won't change
    float confirmedClose = close
```

---

## Pitfalls / Failure Modes

1. **Repainting** - All barstate variables can cause repainting; strategies may behave differently in backtesting vs live
2. **request.security** - Using barstate.isconfirmed inside request.security returns unreliable values
3. **History vs Realtime** - Logic dependent on these variables behaves differently on historical vs realtime bars
4. **Strategy testing** - Strategies using barstate variables may show different results after reloading

---

## References

- [TradingView Pine Script v6 Reference - barstate](https://www.tradingview.com/pine-script-reference/v6/)
- [Execution Model Documentation](https://www.tradingview.com/pine-script-docs/language/execution-model/)
