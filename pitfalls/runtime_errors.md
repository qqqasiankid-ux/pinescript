# Runtime Errors

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: qqqasiankid-ux/ok repository (concepts/common_errors.md)

## Scope

This file documents common Pine Script v6 runtime errors, their causes, and solutions.

---

## Canonical Rules

- Runtime errors occur during script execution, not compilation
- Runtime errors can appear on specific bars or conditions
- Many runtime errors can be prevented with defensive coding patterns

---

## Error: "Loop is too long (> 500 ms)"

### Description
Pine Script limits the computation time of loops on every historical bar and realtime tick to protect servers from infinite or very long loops.

### Example (Problematic)
```pine
//@version=6
indicator("Loop is too long", max_bars_back = 101)
int s = 0
for i = 1 to 1e3
    for j = 0 to 100
        if timestamp(2017, 02, 23, 00, 00) <= time[j] and time[j] < timestamp(2017, 02, 23, 23, 59)
            s := s + 1
plot(s)
```

### Solution
Optimize algorithms using binary search or reduce nested loop depth.

### Canonical Rule
- Loop computation is limited to 500ms per bar

---

## Error: "The requested historical offset (X) is beyond the historical buffer's limit (Y)"

### Description
Pine scripts maintain a historical buffer of values. Buffer size is determined automatically but can be insufficient with dynamic offsets on realtime bars.

### Solution: Use max_bars_back() function
```pine
//@version=6
indicator("Fixed with max_bars_back")
float myVar = close[barstate.ishistory ? 500 : 1000]
max_bars_back(close, 1000)
plot(myVar)
```

### Canonical Rules
- Maximum buffer size for most variables is 5000 bars
- Buffer size is NOT adjusted on realtime bars
- For drawings, always set max_bars_back(time, N)

---

## Error: "Memory limits exceeded"

### Description
Most commonly occurs when retrieving objects and collections from request.*() functions.

### Solution: Return calculated results only
```pine
//@version=6
indicator("Return results only")

dataFunction() => 
    var array<float> dataArray = array.new<float>(0)
    float bop = (close - open) / (high - low) * 100
    dataArray.push(bop)
    float avgBOP = dataArray.avg()
    [bop, avgBOP]

[reqValue, reqAverage] = request.security(syminfo.tickerid, "1D", dataFunction())
plot(reqValue)
```

### Canonical Rules
- Return calculated results from request.*(), not full collections
- Use var for persistent objects
- Avoid recreating tables/drawings on every bar

---

## References

- [TradingView Pine Script v6 Reference](https://www.tradingview.com/pine-script-reference/v6/)
- [Pine Script Error Messages](https://www.tradingview.com/pine-script-docs/error-messages/)