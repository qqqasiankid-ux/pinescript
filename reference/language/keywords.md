# Keywords
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents Pine Script v6 reserved keywords used for control flow, logic, variables, types, and modules.

## Canonical Rules
- Keywords are reserved and cannot be used as variable names
- Keywords are case-sensitive in Pine Script
- Proper keyword usage is mandatory for script compilation
- Some keywords were introduced in specific Pine Script versions
- Keywords define the structure and control flow of scripts

## Control Flow Keywords

### if
Conditional statement for branching logic.

**Syntax (v6):**
```pine
//@version=6
indicator("If Example")

if close > open
    // Executed when condition is true
    bgcolor(color.green)
else if close < open
    // Executed when first condition is false and this is true
    bgcolor(color.red)
else
    // Executed when all conditions are false
    bgcolor(color.gray)
```

**Rules:**
- Requires 4-space indentation for block
- Can have multiple `else if` branches
- `else` is optional

---

### for
Loop with counter for fixed iterations.

**Syntax (v6):**
```pine
//@version=6
indicator("For Loop")

var total = 0.0
for i = 0 to 9
    total += i

plot(total)
```

**Rules:**
- `to` creates inclusive range (0 to 9 = 10 iterations)
- Can use `by` for step: `for i = 0 to 10 by 2`
- Loop variable is local to the loop

---

### for...in
Loop over array, matrix, or map elements.

**Syntax (v6):**
```pine
//@version=6
indicator("For In Example")

var prices = array.from(10.0, 20.0, 30.0, 40.0)

var sum = 0.0
for price in prices
    sum += price

plot(sum)
```

**Matrix iteration:**
```pine
//@version=6
indicator("Matrix For In")

var m = matrix.new<float>(2, 2, 0)

// Iterate by row
for row in m
    for element in row
        // Process each element
        element
```

---

### while
Loop with condition (continues while condition is true).

**Syntax (v6):**
```pine
//@version=6
indicator("While Loop")

var count = 0
var result = 0

while count < 10
    result += count
    count += 1

plot(result)
```

**Rules:**
- Condition evaluated before each iteration
- Must ensure termination to avoid infinite loops
- Use `break` to exit early

---

### switch
Multi-way conditional (introduced in Pine Script v5).

**Syntax (v6):**
```pine
//@version=6
indicator("Switch Example")

timeframe = input.string("D", "Timeframe", options=["1", "5", "15", "60", "D"])

color = switch timeframe
    "1"  => color.red
    "5"  => color.orange
    "15" => color.yellow
    "60" => color.blue
    "D"  => color.green
    =>      color.gray  // Default case

bgcolor(color)
```

**Expression form:**
```pine
//@version=6
indicator("Switch Expression")

value = switch
    close > open => 1
    close < open => -1
    =>              0

plot(value)
```

---

## Logical Keywords

### and
Logical AND operator.

**Syntax (v6):**
```pine
//@version=6
indicator("And Example")

bool uptrend = close > open and high > high[1]
bool condition = rsi > 50 and volume > sma(volume, 20)

bgcolor(uptrend ? color.green : na)
```

---

### or
Logical OR operator.

**Syntax (v6):**
```pine
//@version=6
indicator("Or Example")

bool signal = close > high[1] or close < low[1]
bool alert = rsi > 70 or rsi < 30

bgcolor(signal ? color.yellow : na)
```

---

### not
Logical NOT operator (negation).

**Syntax (v6):**
```pine
//@version=6
indicator("Not Example")

bool downtrend = close < open
bool uptrend = not downtrend

bgcolor(uptrend ? color.green : color.red)
```

---

## Variable Declaration Keywords

### var
Declares variable that persists across bars (initialized once).

**Syntax (v6):**
```pine
//@version=6
indicator("Var Example")

var int barCount = 0  // Initialized only on first bar
var float startPrice = close  // Captures first bar close

barCount += 1  // Increments every bar

plot(barCount)
plot(startPrice)
```

**Rules:**
- Initialized only once (on first bar if `barstate.isfirst`)
- Value persists across bars
- Can be reassigned with `:=` operator

---

### varip
Variable that persists across real-time ticks (intrabar persistence).

**Syntax (v6):**
```pine
//@version=6
indicator("Varip Example")

varip int tickCount = 0  // Persists across ticks within same bar
varip float highestTick = 0.0

tickCount += 1  // Increments on every tick

if high > highestTick
    highestTick := high

plot(tickCount)
```

**Rules:**
- Only updates on real-time data (not historical bars)
- Resets when new bar opens
- Useful for tick-level analysis

---

## Type Keywords

### type
Declares user-defined type (UDT) - object with fields.

**Syntax (v6):**
```pine
//@version=6
indicator("Type Example")

type Point
    int x
    float y
    string label

var point = Point.new(bar_index, close, "Start")

// Access fields
float yValue = point.y

// Modify fields
point.label := "Updated"
```

**Rules:**
- Introduced in Pine Script v5
- Creates custom objects with named fields
- Fields can be any Pine Script type
- Use `.new()` to create instances

---

### enum
Declares enumeration (set of named constants).

**Syntax (v6):**
```pine
//@version=6
indicator("Enum Example")

enum Direction
    UP
    DOWN
    SIDEWAYS

getDirection(c) =>
    if c > c[1]
        Direction.UP
    else if c < c[1]
        Direction.DOWN
    else
        Direction.SIDEWAYS

direction = getDirection(close)

color = switch direction
    Direction.UP => color.green
    Direction.DOWN => color.red
    Direction.SIDEWAYS => color.gray

bgcolor(color)
```

---

## Module Keywords

### import
Imports external library for use in script.

**Syntax (v6):**
```pine
//@version=6
indicator("Import Example")

import TradingView/ta/1 as ta_lib

// Use imported library functions
rsiValue = ta_lib.rsi(close, 14)

plot(rsiValue)
```

**Rules:**
- Imports published Pine Script libraries
- Can use alias with `as` keyword
- Format: `username/libraryname/version`

---

### export
Exports function or type from library script (library scripts only).

**Syntax (v6):**
```pine
//@version=6
library("MyLibrary")

export customRSI(float src, int length) =>
    // Custom RSI implementation
    ta.rsi(src, length) * 1.1
```

**Rules:**
- Only used in library scripts (not indicators/strategies)
- Makes functions/types available for import
- Requires `library()` declaration instead of `indicator()`

---

### method
Declares a method for a type (extends functionality of types).

**Syntax (v6):**
```pine
//@version=6
indicator("Method Example")

type Point
    float x
    float y

method distance(Point this, Point other) =>
    math.sqrt(math.pow(this.x - other.x, 2) + math.pow(this.y - other.y, 2))

point1 = Point.new(0.0, 0.0)
point2 = Point.new(3.0, 4.0)

dist = point1.distance(point2)  // Calls method
plot(dist)  // Plots 5.0
```

---

## Loop Control Keywords

### break
Exits a loop immediately.

**Syntax (v6):**
```pine
//@version=6
indicator("Break Example")

var result = 0
for i = 0 to 100
    if i > 10
        break  // Exit loop when i > 10
    result += i

plot(result)
```

---

### continue
Skips to next iteration of loop.

**Syntax (v6):**
```pine
//@version=6
indicator("Continue Example")

var sum = 0
for i = 0 to 20
    if i % 2 == 0
        continue  // Skip even numbers
    sum += i

plot(sum)  // Sum of odd numbers only
```

---

## Pitfalls / Failure Modes

### Pitfall 1: Using Keywords as Variable Names
- **Problem:** `var if = 10` or `float for = close`
- **Impact:** Compile error: "Syntax error"
- **Prevention:** Never use keywords as identifiers; use descriptive names

### Pitfall 2: Incorrect Indentation with if/for
- **Problem:** Missing or inconsistent indentation in blocks
- **Impact:** "Mismatched input" compile error
- **Prevention:** Always use 4-space indentation; never tabs

### Pitfall 3: Confusing var and varip
- **Problem:** Using `varip` for historical analysis (it only works on real-time)
- **Impact:** Historical bars show unexpected values
- **Prevention:** Use `var` for bar-to-bar persistence; `varip` only for tick counting

### Pitfall 4: Infinite while Loops
- **Problem:** `while` condition never becomes false
- **Impact:** Script timeout, execution error
- **Prevention:** Ensure loop termination; use counter limits; test thoroughly

### Pitfall 5: Missing Default in switch
- **Problem:** No default case (`=>`) in switch when value doesn't match any case
- **Impact:** Returns `na` unexpectedly
- **Prevention:** Always include default case for complete coverage

### Pitfall 6: Improper for...in Usage
- **Problem:** Trying to modify array elements directly in `for...in` loop
- **Impact:** Doesn't modify original array (loop variable is copy)
- **Prevention:** Use index-based `for` loop for modifications: `for i = 0 to array.size(arr) - 1`

## References
- TradingView Pine Script v6 Language Reference: https://www.tradingview.com/pine-script-docs/language/
- Control Structures: https://www.tradingview.com/pine-script-docs/language/Control-structures
- Variable Declaration: https://www.tradingview.com/pine-script-docs/language/Variable-declarations
- User-Defined Types: https://www.tradingview.com/pine-script-docs/language/Type-system#user-defined-types
- Libraries: https://www.tradingview.com/pine-script-docs/concepts/Libraries
