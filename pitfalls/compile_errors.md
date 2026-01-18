# Compile Errors
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents common Pine Script v6 compile-time errors, their causes, and solutions. These errors prevent script execution and must be resolved before the script can run.

## Canonical Rules
- All compile errors must be fixed before script execution
- Compile errors are deterministic and reproducible
- Error messages provide hints to the root cause
- Version-specific syntax differences can cause compile errors
- Proper indentation and syntax structure are mandatory

## Common Compile Errors

### 1. "The if statement is too long"

**Cause:** Local block inside an `if` statement exceeds Pine Script's compilation limits.

**Example (v6):**
```pine
//@version=6
indicator("Long If Block", overlay=true)

var float result = 0.0

if close > open
    // If this block becomes too large with many calculations
    float calc1 = close * 1.1
    float calc2 = open * 0.9
    float calc3 = high * 1.05
    // ... hundreds of similar lines ...
    result := calc1 + calc2 + calc3
```

**Solution:** Break logic into separate functions or blocks.

```pine
//@version=6
indicator("Refactored If Block", overlay=true)

calculateValues() =>
    calc1 = close * 1.1
    calc2 = open * 0.9
    calc3 = high * 1.05
    calc1 + calc2 + calc3

var float result = 0.0
if close > open
    result := calculateValues()
```

### 2. "Script requesting too many securities"

**Cause:** Script exceeds the maximum limit of 40 `request.*()` calls.

**Example (v6) - PROBLEMATIC:**
```pine
//@version=6
indicator("Too Many Securities", overlay=true)

// Requesting 50+ securities will fail
sym1 = request.security(syminfo.tickerid, "1", close)
sym2 = request.security(syminfo.tickerid, "5", close)
// ... 40+ more calls ...
```

**Solution:** Optimize by using tuples and consolidating requests.

```pine
//@version=6
indicator("Optimized Securities", overlay=true)

// Use tuples to request multiple values in one call
[close1, close5, close15] = request.security(syminfo.tickerid, "1", [close, close, close])

// Use request.security_lower_tf() for intrabar data
var array<float> intrabars = array.new<float>()
if barstate.islast
    intrabars := request.security_lower_tf(syminfo.tickerid, "1", close)

// Only 3 actual security calls instead of 50+
plot(close1)
plot(close5)
plot(close15)
```

### 3. "Script could not be translated from: null"

**Cause:** Attempting to run a Pine Script version 1 script in newer versions without updating syntax.

**Example (v6) - BROKEN:**
```pine
//@version=6
// Using v1 syntax will fail
study("Old Script")  // 'study' was deprecated
```

**Solution:** Update to current v6 syntax.

```pine
//@version=6
indicator("Updated Script", overlay=true)
// Use 'indicator' instead of deprecated 'study'
plot(close)
```

### 4. "no viable alternative at character '$'"

**Cause:** Invalid character in variable names or syntax errors. Dollar sign `$` is not valid in Pine Script identifiers.

**Example (v6) - BROKEN:**
```pine
//@version=6
indicator("Invalid Character")
float $price = close  // $ not allowed in variable names
```

**Solution:** Use only valid characters (letters, numbers, underscore).

```pine
//@version=6
indicator("Valid Names")
float price = close
float price_usd = close
```

### 5. "Mismatched input expecting"

**Cause:** Indentation errors or incorrect block structure. Pine Script requires consistent 4-space indentation.

**Example (v6) - BROKEN:**
```pine
//@version=6
indicator("Bad Indentation")

if close > open
plot(close)  // Missing indentation!
```

**Solution:** Use proper 4-space indentation.

```pine
//@version=6
indicator("Correct Indentation")

if close > open
    plot(close)  // Properly indented with 4 spaces
```

### 6. "Script has too many local variables"

**Cause:** Exceeding the limit of approximately 1000 local variables in a single scope.

**Example (v6) - PROBLEMATIC:**
```pine
//@version=6
indicator("Too Many Variables")

// Creating hundreds of individual variables
float var1 = close * 1
float var2 = close * 2
float var3 = close * 3
// ... 1000+ variables ...
```

**Solution:** Combine expressions or use arrays/collections to reduce variable count.

```pine
//@version=6
indicator("Optimized Variables")

// Use arrays to store related values
var multipliers = array.new<float>()
if barstate.isfirst
    for i = 1 to 100
        array.push(multipliers, i)

// Calculate on demand instead of storing
calculateValue(multiplier) =>
    close * multiplier

result = calculateValue(array.get(multipliers, 0))
plot(result)
```

## Pitfalls / Failure Modes

### Pitfall 1: Ignoring Version Directive
- **Problem:** Omitting `//@version=6` causes script to default to older version with different behavior
- **Impact:** Syntax errors, unexpected behavior, deprecated functions
- **Prevention:** Always explicitly declare version at the top of every script

### Pitfall 2: Nested Block Complexity
- **Problem:** Deep nesting of if/for/while blocks can hit compilation limits
- **Impact:** "too long" errors even with moderate code
- **Prevention:** Refactor nested logic into separate functions; keep blocks shallow

### Pitfall 3: Security Call Proliferation
- **Problem:** Using `request.security()` in loops or generating multiple calls dynamically
- **Impact:** Hitting 40-call limit unexpectedly
- **Prevention:** Count security calls explicitly; use tuples; cache results

### Pitfall 4: Copy-Pasting from Old Scripts
- **Problem:** Using deprecated syntax from v1-v4 scripts without updating
- **Impact:** Compilation failures with cryptic messages
- **Prevention:** Always update syntax to v6; use `indicator` not `study`, `input.*` functions correctly

### Pitfall 5: Invisible Character Errors
- **Problem:** Copy-pasting code introduces special Unicode characters that look like spaces
- **Impact:** "no viable alternative" errors at seemingly valid code
- **Prevention:** Use plain text editors; re-type suspicious lines; check character codes

### Pitfall 6: Variable Explosion in Loops
- **Problem:** Declaring new variables inside loops that execute many times
- **Impact:** Exceeds local variable limit
- **Prevention:** Declare variables outside loops; reuse variables; use collections

## References
- TradingView Pine Script v6 Documentation: https://www.tradingview.com/pine-script-docs/
- Pine Script Error Messages: https://www.tradingview.com/pine-script-docs/language/Errors
- request.security() Limits: https://www.tradingview.com/pine-script-docs/concepts/Other-timeframes-and-data
- Pine Script Migration Guide: https://www.tradingview.com/pine-script-docs/migration-guides/
