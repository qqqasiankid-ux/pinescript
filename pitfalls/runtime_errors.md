# Runtime Errors
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents common Pine Script v6 runtime errors that occur during script execution. These errors prevent scripts from completing execution on specific bars or conditions.

## Canonical Rules
- Runtime errors occur after successful compilation
- Runtime errors are often data-dependent or resource-dependent
- Some runtime errors can be prevented with defensive coding
- Memory and performance limits are enforced at runtime
- Historical data access has hard limits that must be respected

## Common Runtime Errors

### 1. "Loop is too long (> 500 ms)"

**Cause:** A loop takes more than 500 milliseconds to execute on a single bar, exceeding Pine Script's execution time limit.

**Example (v6) - PROBLEMATIC:**
```pine
//@version=6
indicator("Slow Loop", overlay=true)

var result = 0
if barstate.islast
    // Linear search through large dataset - O(n)
    for i = 1 to 5000
        for j = 1 to 5000
            result := result + 1  // 25 million iterations!

plot(result)
```

**Solution:** Optimize with better algorithms (e.g., binary search) or reduce iterations.

```pine
//@version=6
indicator("Fast Binary Search", overlay=true)

// Binary search - O(log n) instead of O(n)
binarySearch(arr, target) =>
    var int result = -1
    int left = 0
    int right = array.size(arr) - 1
    
    while left <= right
        int mid = math.floor((left + right) / 2)
        float midVal = array.get(arr, mid)
        
        if midVal == target
            result := mid
            break
        else if midVal < target
            left := mid + 1
        else
            right := mid - 1
    result

// Example usage
var sortedPrices = array.new<float>()
if barstate.isfirst
    for i = 0 to 100
        array.push(sortedPrices, i * 10.0)

index = binarySearch(sortedPrices, close)
plot(index)
```

### 2. "The requested historical offset (X) is beyond the historical buffer's limit (Y)"

**Cause:** Accessing historical data beyond the available bar history. Pine Script maintains a limited historical buffer for each variable.

**Example (v6) - PROBLEMATIC:**
```pine
//@version=6
indicator("Buffer Overflow", overlay=true)

// Trying to access 5000 bars back without declaring buffer size
var float deepHistory = na
if bar_index > 5000
    deepHistory := close[5000]  // May exceed buffer!

plot(deepHistory)
```

**Solution:** Use `max_bars_back()` to explicitly declare required history.

```pine
//@version=6
indicator("Fixed Buffer Size", overlay=true)

var float deepHistory = na

// Explicitly declare we need 5000 bars of history
max_bars_back(close, 5000)

if bar_index >= 5000
    deepHistory := close[5000]
else
    deepHistory := na

plot(deepHistory)
```

**Alternative Solution:** Use arrays for flexible historical storage.

```pine
//@version=6
indicator("Array-Based History", overlay=true)

var priceHistory = array.new<float>()

// Store history in array (no buffer limits)
array.push(priceHistory, close)

// Keep only last 5000 bars
if array.size(priceHistory) > 5000
    array.shift(priceHistory)

// Access deep history safely
float deepValue = na
if array.size(priceHistory) >= 5000
    deepValue := array.get(priceHistory, 0)

plot(deepValue)
```

### 3. "Memory limits exceeded"

**Cause:** Script consumes too much memory, often from excessive `request.*()` calls, large arrays, or too many drawing objects.

**Example (v6) - PROBLEMATIC:**
```pine
//@version=6
indicator("Memory Heavy", overlay=true)

// Requesting many securities with all historical data
spy_data = request.security("SPY", "D", close, barmerge.gaps_off, barmerge.lookahead_off)
qqq_data = request.security("QQQ", "D", close, barmerge.gaps_off, barmerge.lookahead_off)
iwm_data = request.security("IWM", "D", close, barmerge.gaps_off, barmerge.lookahead_off)
// ... 37+ more securities with full history
```

**Solution:** Optimize `request.*()` usage with selective data fetching.

```pine
//@version=6
indicator("Memory Optimized", overlay=true)

// Request only necessary data using tuples
[spy, qqq, iwm] = request.security(
     syminfo.tickerid, 
     "D", 
     [close, close, close]
     )

// Use request.security_lower_tf only when needed
var array<float> intrabars = array.new<float>()
if barstate.islast
    // Limit scope to current bar
    intrabars := request.security_lower_tf(syminfo.tickerid, "1", close)
    // Process immediately and discard
    float avg = array.avg(intrabars)
    array.clear(intrabars)

plot(spy)
```

**Additional Strategies:**
```pine
//@version=6
indicator("Drawing Object Limits", overlay=true)

// Limit drawing objects to prevent memory issues
var lines = array.new<line>()
const int MAX_LINES = 50

if close > open
    // Create new line
    newLine = line.new(bar_index, low, bar_index, high, color=color.green)
    array.push(lines, newLine)
    
    // Remove oldest if exceeding limit
    if array.size(lines) > MAX_LINES
        oldLine = array.shift(lines)
        line.delete(oldLine)
```

## Pitfalls / Failure Modes

### Pitfall 1: Nested Loops Without Bounds
- **Problem:** Nested loops with large iteration counts can exponentially increase execution time
- **Impact:** "Loop is too long" errors, especially on historical bars with many iterations
- **Prevention:** Calculate time complexity; prefer O(log n) or O(n) algorithms; avoid O(n²) or worse

### Pitfall 2: Assuming Unlimited History
- **Problem:** Accessing historical bars without considering buffer limits
- **Impact:** Runtime errors when accessing deep history; script works on recent bars but fails on older ones
- **Prevention:** Always use `max_bars_back()` for known deep access; use arrays for flexible storage

### Pitfall 3: Unbounded Array Growth
- **Problem:** Arrays that grow indefinitely across all bars
- **Impact:** Memory exhaustion, especially on long timeframes or datasets
- **Prevention:** Implement size limits; use circular buffers; clear arrays when no longer needed

### Pitfall 4: Excessive Drawing Objects
- **Problem:** Creating drawing objects (lines, labels, boxes) on every bar without cleanup
- **Impact:** Memory limits exceeded; visual clutter; performance degradation
- **Prevention:** Limit active objects (e.g., MAX_DRAWINGS constant); delete old objects; use conditional creation

### Pitfall 5: Security Call Data Accumulation
- **Problem:** Using `request.security()` with unnecessary historical data retention
- **Impact:** Memory limits exceeded with multiple securities
- **Prevention:** Request only current/recent data; use `barmerge.lookahead_on` carefully; consolidate requests

### Pitfall 6: Ignoring barstate Conditions
- **Problem:** Running expensive operations on every historical bar
- **Impact:** Cumulative performance issues; timeout on large datasets
- **Prevention:** Use `barstate.islast` or `barstate.isrealtime` to limit expensive calculations to necessary bars

## References
- TradingView Pine Script v6 Documentation: https://www.tradingview.com/pine-script-docs/
- Pine Script Execution Model: https://www.tradingview.com/pine-script-docs/language/Execution-model
- max_bars_back(): https://www.tradingview.com/pine-script-docs/language/Max-bars-back
- Limitations in Pine Script: https://www.tradingview.com/pine-script-docs/writing/Limitations
- request.security() Best Practices: https://www.tradingview.com/pine-script-docs/concepts/Other-timeframes-and-data
