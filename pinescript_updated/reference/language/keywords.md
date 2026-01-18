# Pine Script v6 Keywords

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: qqqasiankid-ux/ok repository (reference/keywords.md)

## Scope

This file documents all Pine Script v6 keywords including control flow, logical operators, variable modifiers, and module system keywords.

---

## Control Flow Keywords

### if
Defines conditional execution blocks.

```pine
//@version=6
indicator("if example")
float x = na
if close > open
    x := close
else
    x := open
plot(x)
```

### for
Creates count-controlled loops.

```pine
//@version=6
indicator("for example")
int result = 0
for i = 1 to 14
    if close[i] > close
        result += 1
plot(result)
```

### while
Creates condition-controlled loops.

```pine
//@version=6
indicator("while example")
int counter = 10
int factorial = 1
while counter > 0
    factorial := factorial * counter
    counter := counter - 1
plot(factorial)
```

### switch
Transfers control based on conditions.

```pine
//@version=6
indicator("switch example")
string i_maType = input.string("EMA", "MA type", options = ["EMA", "SMA", "RMA", "WMA"])
float ma = switch i_maType
    "EMA" => ta.ema(close, 10)
    "SMA" => ta.sma(close, 10)
    => ta.wma(close, 10)
plot(ma)
```

---

## Logical Operators

### and
Logical AND. Short-circuit evaluation.

### or
Logical OR. Short-circuit evaluation.

### not
Logical negation.

---

## Variable Modifiers

### var
One-time initialization. Variable persists across bars.

```pine
//@version=6
indicator("var example")
var int count = 0
if close > open
    count := count + 1
plot(count)
```

### varip
Var intrabar persist. Persists across ticks within a bar.

```pine
//@version=6
indicator("varip example")
varip int tickCount = 0
tickCount := tickCount + 1
plot(tickCount)
```

---

## Type Declaration Keywords

### type
Declares user-defined types (UDT).

### enum
Creates enumerations with predefined constants.

---

## Module System Keywords

### import
Loads external libraries.

### export
Marks functions/types for library export.

### method
Declares methods that can use dot notation.

---

## Pitfalls / Failure Modes

1. **var vs varip** - var persists across bars; varip persists across ticks within the same bar
2. **Loop limits** - Loops have 500ms execution time limit
3. **Short-circuit evaluation** - and/or may not evaluate second operand

---

## References

- [TradingView Pine Script v6 Reference](https://www.tradingview.com/pine-script-reference/v6/)
