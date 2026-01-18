# Compile Errors

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: qqqasiankid-ux/ok repository (concepts/common_errors.md)

## Scope

This file documents common Pine Script v6 compile-time errors, their causes, and solutions.

---

## Canonical Rules

- Compile errors occur before script execution, during the compilation phase
- All compile errors must be resolved before a script can run
- Error messages provide hints about the location and nature of the problem

---

## Error: "The if statement is too long"

### Description
This error occurs when the indented code (local block) inside an `if` structure is too large for the compiler to process.

### Example (v6)
```pine
//@version=6
indicator("My script")

var int e = 0
if barstate.islast
    int a = 1
    int b = 2
    int c = 3
    int d = 4
    e := a + b + c + d

plot(e)
```

### Solution
Break large `if` blocks into smaller functions or reduce the number of statements within the block.

---

## Error: "Script requesting too many securities"

### Description
The maximum number of securities in a script is limited to **40**. Duplicate `request.security` calls with identical parameters are optimized out by the compiler.

### Example (v6)
```pine
//@version=6
indicator("Securities count")
a = request.security(syminfo.tickerid, '42', close)  // (1) first unique security call
b = request.security(syminfo.tickerid, '42', close)  // same call as above, optimized out

plot(a)
plot(a + 2)
plot(b)

sym(p) =>  // no security call on this line
    request.security(syminfo.tickerid, p, close)
plot(sym('D'))  // (2) one indirect call to security
plot(sym('W'))  // (3) another indirect call to security

c = request.security(syminfo.tickerid, timeframe.period, open)  // result unused, optimized out
```

### Canonical Rule
- Maximum 40 unique `request.security` calls per script
- Identical calls are deduplicated by the compiler
- Unused results may be optimized out

---

## Error: "Script could not be translated from: null"

### Description
Usually occurs in version 1 Pine scripts and indicates incorrect code. Upgrading to a newer version provides better error messages.

### Example (Problematic - v1)
```pine
study($)
```

### Solution
Upgrade to v6 and use proper syntax:
```pine
//@version=6
indicator("title")
```

---

## Error: "no viable alternative at character '$'"

### Description
This syntax error indicates an unexpected character. The error message hints at what character is problematic.

### Example (Problematic)
```pine
// @version=2
study($)
```

### Solution
```pine
//@version=6
indicator("title")
```

---

## Error: "Mismatched input expecting"

### Description
Similar to "no viable alternative" but the compiler knows what should be at that position. Often caused by incorrect indentation.

### Example (Problematic)
```pine
//@version=6
indicator("My Script")
    plot(1)
```

Error: `line 3: mismatched input 'plot' expecting 'end of line without line continuation'`

### Solution
Remove the indentation - `plot` should be at the global scope:
```pine
//@version=6
indicator("My Script")
plot(1)
```

---

## Error: "Script has too many local variables"

### Description
This error appears when the script is too large to compile. Each `var = expression` creates a local variable, and auxiliary variables are created during compilation.

### Example (Inefficient)
```pine
var1 = expr1
var2 = expr2
var3 = var1 + var2
```

### Solution
Combine expressions to reduce variable count:
```pine
var3 = expr1 + expr2
```

---

## Pitfalls / Failure Modes

1. **Indentation errors** - Pine Script uses indentation for scoping; incorrect indentation causes "mismatched input" errors
2. **Version mismatch** - Using syntax from one version in another causes compilation failures
3. **Security call limits** - Exceeding 40 unique security calls is a hard limit that cannot be worked around
4. **Large scripts** - Scripts with too many variables may need refactoring into libraries

---

## References

- [TradingView Pine Script v6 Reference](https://www.tradingview.com/pine-script-reference/v6/)
- [Pine Script Error Messages Documentation](https://www.tradingview.com/pine-script-docs/error-messages/)