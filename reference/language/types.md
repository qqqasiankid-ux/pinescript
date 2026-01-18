# Types
## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope
This file documents the Pine Script v6 type system, including basic types, collections, drawing objects, and type qualifiers.

## Canonical Rules
- Pine Script v6 uses explicit typing for most declarations
- Type inference works for simple assignments but explicit types are preferred
- `na` assignments require explicit type declarations
- Collections (array, map, matrix) require type parameters
- Type qualifiers (const, simple, series) affect when values can be determined
- Type mismatches cause compile-time errors

## Basic Types

### int
Integer numbers (whole numbers without decimals).

**Example (v6):**
```pine
//@version=6
indicator("Integer Example")

int count = 10
int barIndex = bar_index
int negative = -42

plot(count)
```

**Range:** Approximately -9.2 quintillion to 9.2 quintillion (-2^63 to 2^63-1)

---

### float
Floating-point numbers (decimals).

**Example (v6):**
```pine
//@version=6
indicator("Float Example")

float price = 123.45
float ratio = 1.618
float calculated = close * 1.05

plot(calculated)
```

**Precision:** IEEE 754 double-precision (approximately 15-17 significant decimal digits)

---

### bool
Boolean values (`true` or `false`).

**Example (v6):**
```pine
//@version=6
indicator("Boolean Example")

bool isUptrend = close > close[1]
bool condition = high > high[1] and low > low[1]

bgcolor(isUptrend ? color.green : color.red)
```

---

### string
Text strings (sequences of characters).

**Example (v6):**
```pine
//@version=6
indicator("String Example")

string symbol = syminfo.tickerid
string message = "Price: " + str.tostring(close)
string template = "{0} is trading at {1}"

var label myLabel = label.new(bar_index, high, message)
```

---

### color
Color values for visual styling.

**Example (v6):**
```pine
//@version=6
indicator("Color Example")

color bullColor = color.green
color bearColor = color.red
color transparent = color.new(color.blue, 50)  // 50% transparency

plot(close, color = close > open ? bullColor : bearColor)
```

**Built-in colors:** `color.red`, `color.green`, `color.blue`, `color.yellow`, `color.white`, `color.black`, `color.gray`, etc.

---

## Collection Types

### array<type>
Ordered collection of elements of the same type.

**Example (v6):**
```pine
//@version=6
indicator("Array Example")

var prices = array.new<float>()

array.push(prices, close)

if array.size(prices) > 100
    array.shift(prices)

float avgPrice = array.avg(prices)
plot(avgPrice)
```

**Type parameter required:** `array.new<float>()`, `array.new<int>()`, `array.new<string>()`, etc.

---

### map<keyType, valueType>
Key-value pairs collection (introduced in Pine Script v5+).

**Example (v6):**
```pine
//@version=6
indicator("Map Example")

var symbolPrices = map.new<string, float>()

if barstate.isnew
    map.put(symbolPrices, syminfo.tickerid, close)

float storedPrice = map.get(symbolPrices, syminfo.tickerid)
plot(storedPrice)
```

**Key types allowed:** `string` or `int`
**Value types:** Any Pine Script type

---

### matrix<type>
Two-dimensional array (rows and columns).

**Example (v6):**
```pine
//@version=6
indicator("Matrix Example")

var priceMatrix = matrix.new<float>(3, 3, 0.0)

matrix.set(priceMatrix, 0, 0, close)
matrix.set(priceMatrix, 0, 1, high)
matrix.set(priceMatrix, 0, 2, low)

float cell = matrix.get(priceMatrix, 0, 0)
plot(cell)
```

---

## Drawing Object Types

### line
Line object for drawing lines on chart.

**Example (v6):**
```pine
//@version=6
indicator("Line Example", overlay=true)

var line myLine = na

if barstate.islast
    myLine := line.new(bar_index-10, low[10], bar_index, high, color=color.blue, width=2)
```

---

### label
Label object for text annotations.

**Example (v6):**
```pine
//@version=6
indicator("Label Example", overlay=true)

var label myLabel = na

if barstate.islast
    myLabel := label.new(bar_index, high, "High", color=color.green, style=label.style_label_down)
```

---

### box
Box/rectangle object for highlighting areas.

**Example (v6):**
```pine
//@version=6
indicator("Box Example", overlay=true)

var box myBox = na

if barstate.islast
    myBox := box.new(bar_index-10, high[10], bar_index, low, bgcolor=color.new(color.blue, 80))
```

---

### table
Table object for displaying data in a grid.

**Example (v6):**
```pine
//@version=6
indicator("Table Example", overlay=true)

var myTable = table.new(position.top_right, 2, 2)

if barstate.islast
    table.cell(myTable, 0, 0, "Symbol", bgcolor=color.gray)
    table.cell(myTable, 1, 0, syminfo.tickerid, bgcolor=color.white)
```

---

### linefill
Fill area between two lines.

**Example (v6):**
```pine
//@version=6
indicator("Linefill Example", overlay=true)

line1 = line.new(bar_index-10, high[10], bar_index, high, color=color.blue)
line2 = line.new(bar_index-10, low[10], bar_index, low, color=color.red)

linefill.new(line1, line2, color=color.new(color.purple, 80))
```

---

### polyline
Multi-segment line (connected points).

**Example (v6):**
```pine
//@version=6
indicator("Polyline Example", overlay=true)

var points = array.new<chart.point>()

if barstate.islast and array.size(points) == 0
    array.push(points, chart.point.new(bar_index-10, high[10]))
    array.push(points, chart.point.new(bar_index-5, low[5]))
    array.push(points, chart.point.new(bar_index, close))
    
    polyline.new(points, closed=false, line_color=color.blue, line_width=2)
```

---

## Special Types

### chart.point
Represents a point on the chart (x, y coordinates with optional time).

**Example (v6):**
```pine
//@version=6
indicator("Chart Point Example", overlay=true)

point1 = chart.point.new(bar_index, high)
point2 = chart.point.new(bar_index-10, low[10], time[10])

// Used with polyline objects
```

---

### na
Special value representing "not available" or "no value".

**Example (v6):**
```pine
//@version=6
indicator("NA Example")

// MUST use explicit type with na
float value = na
var line myLine = na

// Check for na
bool isNA = na(value)

plot(na(close[100]) ? 0 : close[100])
```

**Critical:** Always use explicit type when initializing with `na`: `float x = na`, not `x = na`

---

## Type Qualifiers

### const
Value is determined at compile time and never changes.

**Example (v6):**
```pine
//@version=6
indicator("Const Example")

const int MAX_BARS = 100
const float PI = 3.14159
const string TITLE = "My Indicator"

// Cannot be reassigned
// MAX_BARS := 200  // ERROR
```

---

### simple
Value is determined at bar zero and remains constant across all bars.

**Example (v6):**
```pine
//@version=6
indicator("Simple Example")

simple int lookback = input.int(14, "Lookback")
simple string symbol = input.symbol("AAPL", "Symbol")

// Known at start of script execution
```

---

### series
Value can change on every bar (default for most variables).

**Example (v6):**
```pine
//@version=6
indicator("Series Example")

series float price = close  // Changes every bar
series bool condition = close > open

// Most variables are series by default
float dynamicValue = high - low  // Implicitly series
```

---

## Type Conversion

Pine Script performs automatic type conversion in some cases, but explicit conversion is often needed.

**Example (v6):**
```pine
//@version=6
indicator("Type Conversion")

// int to float (automatic)
float result = 10  // 10 becomes 10.0

// Explicit conversions
int fromFloat = int(123.45)  // 123
float fromInt = float(42)    // 42.0
string fromFloat = str.tostring(123.45)  // "123.45"
string fromInt = str.tostring(42)  // "42"

plot(result)
```

---

## Pitfalls / Failure Modes

### Pitfall 1: Untyped na Assignment
- **Problem:** `x = na` without type declaration
- **Impact:** Compile error: "Cannot determine type"
- **Prevention:** Always use explicit type: `float x = na`

### Pitfall 2: Type Mismatch in Collections
- **Problem:** Mixing types in array operations
- **Impact:** Compile error when trying to push wrong type
- **Prevention:** Declare array type explicitly and enforce it

### Pitfall 3: Forgetting Type Parameters
- **Problem:** `array.new()` without `<type>`
- **Impact:** Compile error in v6
- **Prevention:** Always specify: `array.new<float>()`

### Pitfall 4: Drawing Object Type Confusion
- **Problem:** Treating drawing objects as values (e.g., trying to add lines)
- **Impact:** Type error
- **Prevention:** Drawing objects are references; use proper methods

### Pitfall 5: Qualifier Misuse
- **Problem:** Using `series` value where `simple` or `const` is required
- **Impact:** Compile error in function parameters or contexts requiring known values
- **Prevention:** Understand when values must be known (e.g., array sizes need `simple int`)

### Pitfall 6: Implicit Type Inference Failures
- **Problem:** Relying on type inference with complex expressions
- **Impact:** Unclear errors or unexpected behavior
- **Prevention:** Use explicit types for clarity and reliability

## References
- TradingView Pine Script v6 Type System: https://www.tradingview.com/pine-script-docs/language/Type-system
- Pine Script Built-in Types: https://www.tradingview.com/pine-script-docs/language/Types
- Type Qualifiers: https://www.tradingview.com/pine-script-docs/language/Qualifiers
- Collections (Arrays, Maps, Matrix): https://www.tradingview.com/pine-script-docs/language/Arrays
- Drawing Objects: https://www.tradingview.com/pine-script-docs/concepts/Lines-and-boxes
