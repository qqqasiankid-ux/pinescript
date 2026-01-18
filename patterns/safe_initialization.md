# Safe Initialization
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents safe initialization patterns in Pine Script v6, with emphasis on proper `na` handling and type-safe variable declarations.

## Canonical Rules
- Always use explicit types when initializing with `na`
- Drawing objects must be explicitly typed when initialized to `na`
- Collections (arrays, maps, matrices) require type parameters
- Variables should be initialized before first use
- Use `var` for values that persist across bars
- Uninitialized variables can cause runtime errors

## The na Initialization Rule

### FORBIDDEN: Untyped na Assignment

These patterns will cause compile errors in Pine Script v6:

```pine
//@version=6
indicator("BROKEN - Do Not Use")

// FORBIDDEN - No type specified
x = na  // ERROR: Cannot determine type

// FORBIDDEN - Drawing object without type
var myLine = na  // ERROR: Cannot determine type

// FORBIDDEN - Collection without type
var myArray = na  // ERROR: Cannot determine type
```

### REQUIRED: Explicit Type with na

Always specify the type when initializing with `na`:

```pine
//@version=6
indicator("Correct Initialization", overlay=true)

// REQUIRED - Explicit basic types
float x = na
int count = na
string text = na
bool flag = na
color col = na

// REQUIRED - Explicit drawing object types
var line myLine = na
var label myLabel = na
var box myBox = na
var table myTable = na

// REQUIRED - Alternative: Initialize with actual object
var line activeLine = line.new(bar_index, close, bar_index, close)
```

## Safe Initialization Patterns

### Pattern 1: Basic Variable Initialization

**Example (v6):**
```pine
//@version=6
indicator("Basic Init")

// With explicit type and na
float price = na
int barCount = na
bool condition = na

// With initial value (type inferred but explicit is better)
float explicitPrice = 0.0
int explicitCount = 0
bool explicitFlag = false

// First bar initialization
if barstate.isfirst
    price := close
    barCount := 0
    condition := true

plot(na(price) ? 0 : price)
```

### Pattern 2: var Initialization for Persistence

**Example (v6):**
```pine
//@version=6
indicator("Var Init", overlay=true)

// Initialize once on first bar, persists across bars
var float startPrice = na
var int totalBars = 0
var line trendLine = na

if barstate.isfirst
    startPrice := close
    
totalBars += 1

// Drawing objects with var
if bar_index % 10 == 0
    if not na(trendLine)
        line.delete(trendLine)
    trendLine := line.new(bar_index-10, low[10], bar_index, low, color=color.blue)

plot(startPrice)
```

### Pattern 3: Collection Initialization

**Example (v6):**
```pine
//@version=6
indicator("Collection Init")

// Arrays - specify type parameter
var prices = array.new<float>()
var labels = array.new<string>()
var flags = array.new<bool>()

// Initialize with size and default value
var fixedArray = array.new<float>(10, 0.0)

// Maps - specify key and value types
var priceMap = map.new<string, float>()
var countMap = map.new<int, int>()

// Matrix - specify type and dimensions
var priceMatrix = matrix.new<float>(5, 5, 0.0)

// Add values safely
array.push(prices, close)
map.put(priceMap, syminfo.tickerid, close)
```

### Pattern 4: Drawing Object Lifecycle

**Example (v6):**
```pine
//@version=6
indicator("Drawing Init", overlay=true)

// Initialize as na with explicit type
var line supportLine = na
var label priceLabel = na
var box highlightBox = na

// Check and delete old objects before creating new ones
if barstate.islast
    // Clean up old line
    if not na(supportLine)
        line.delete(supportLine)
    
    // Create new line
    supportLine := line.new(
         bar_index-20, 
         low[20], 
         bar_index, 
         low, 
         color=color.green,
         width=2
         )
    
    // Labels
    if not na(priceLabel)
        label.delete(priceLabel)
    
    priceLabel := label.new(
         bar_index, 
         high, 
         "Price: " + str.tostring(close),
         style=label.style_label_down
         )
```

### Pattern 5: Conditional Initialization

**Example (v6):**
```pine
//@version=6
indicator("Conditional Init")

// Initialize based on condition
float pivotLevel = na

// Set value when condition is met
if ta.pivothigh(5, 5)
    pivotLevel := high[5]

// Use with na check
plot(na(pivotLevel) ? close : pivotLevel, "Pivot")

// Alternative: Use ternary for initialization
float safeValue = na(pivotLevel) ? 0.0 : pivotLevel
plot(safeValue)
```

### Pattern 6: Function Return Value Initialization

**Example (v6):**
```pine
//@version=6
indicator("Function Init")

// Function that may return na
calculateAverage(float src, int len) =>
    if bar_index >= len
        ta.sma(src, len)
    else
        na  // Not enough bars yet

// Initialize with explicit type to handle na return
float avgValue = na
avgValue := calculateAverage(close, 20)

// Safe usage with na check
plot(na(avgValue) ? close : avgValue)
```

## Pitfalls / Failure Modes

### Pitfall 1: Forgetting Type with na
- **Problem:** `x = na` without type declaration
- **Impact:** Compile error: "Cannot determine type"
- **Solution:** Always use explicit type: `float x = na`

### Pitfall 2: Uninitialized Variable Access
- **Problem:** Using variable before assigning any value
- **Impact:** Unexpected `na` values, runtime errors
- **Prevention:** Initialize all variables at declaration or before first use

### Pitfall 3: Drawing Object Memory Leaks
- **Problem:** Creating drawing objects without deleting old ones
- **Impact:** Memory limits exceeded, performance degradation
- **Prevention:** Always delete old objects before creating new ones: `line.delete(oldLine)`

### Pitfall 4: Missing na Checks
- **Problem:** Using potentially `na` values in calculations without checking
- **Impact:** Propagates `na` through calculations, unexpected plot results
- **Prevention:** Use `na()` function to check: `if not na(value)`

### Pitfall 5: Incorrect var Usage
- **Problem:** Using `var` when you want value to change every bar
- **Impact:** Value stuck at first initialization
- **Prevention:** Only use `var` for values that should persist; omit for bar-by-bar updates

### Pitfall 6: Collection Type Mismatches
- **Problem:** Creating `array.new<float>()` but trying to push strings
- **Impact:** Compile error: Type mismatch
- **Prevention:** Ensure collection type matches usage; use correct type parameter

## Advanced Patterns

### Pattern 7: Lazy Initialization

**Example (v6):**
```pine
//@version=6
indicator("Lazy Init")

var float cachedValue = na
var bool initialized = false

// Initialize only when needed
getValue() =>
    if not initialized
        cachedValue := close * 1.1
        initialized := true
    cachedValue

result = getValue()
plot(result)
```

### Pattern 8: Multi-State Initialization

**Example (v6):**
```pine
//@version=6
indicator("Multi-State Init")

// User-defined type for state management
type TradingState
    float entryPrice
    int entryBar
    bool isActive

var state = TradingState.new(na, na, false)

// Initialize state on condition
if not state.isActive and close > ta.sma(close, 20)
    state.entryPrice := close
    state.entryBar := bar_index
    state.isActive := true

// Reset state
if state.isActive and close < ta.sma(close, 20)
    state.isActive := false

plot(state.entryPrice)
```

## Best Practices Summary

1. **Always explicit types with na:** `float x = na`, never `x = na`
2. **Initialize before use:** Set initial value or handle `na` explicitly
3. **Use var for persistence:** When value should persist across bars
4. **Delete old objects:** Before creating new drawing objects
5. **Check for na:** Use `na()` function before calculations
6. **Type consistency:** Match collection types with their usage
7. **Document initialization:** Comment why variables start as `na` or specific values

## References
- TradingView Pine Script v6 Type System: https://www.tradingview.com/pine-script-docs/language/Type-system
- Variable Declaration: https://www.tradingview.com/pine-script-docs/language/Variable-declarations
- na Value: https://www.tradingview.com/pine-script-docs/language/Type-system#na-not-available-value
- Drawing Objects: https://www.tradingview.com/pine-script-docs/concepts/Lines-and-boxes
- Arrays and Collections: https://www.tradingview.com/pine-script-docs/language/Arrays
