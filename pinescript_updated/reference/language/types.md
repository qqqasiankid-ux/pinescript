# Pine Script v6 Types

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: qqqasiankid-ux/ok repository (reference/types.md)

## Scope

This file documents all Pine Script v6 types including basic types, collection types, drawing types, and type qualifiers.

---

## Basic Types

### int
Integer type for whole numbers.

```pine
//@version=6
indicator("int example")
int i = 14
i := na
plot(i)
```

### float
Floating point type for decimal numbers.

```pine
//@version=6
indicator("float example")
float f = 3.14
f := na
plot(f)
```

### bool
Boolean type for true/false values.

```pine
//@version=6
indicator("bool example")
bool b = true
plot(b ? open : close)
```

### string
Text string type.

```pine
//@version=6
indicator("string example")
string s = "Hello World!"
plot(na, title=s)
```

### color
Color type with RRGGBB or RRGGBBAA hex format.

```pine
//@version=6
indicator("color example", overlay = true)
color textColor = color.green
color labelColor = #FF000080  // Red with 50% transparency
if barstate.islastconfirmedhistory
    label.new(bar_index, high, text = "Label", color = labelColor, textcolor = textColor)
```

---

## Collection Types

### array
Dynamic arrays. Always of "series" form.

```pine
//@version=6
indicator("array example", overlay=true)
array<float> a = na
a := array.new<float>(1, close)
plot(array.get(a, 0))
```

### map
Key-value pairs. Always of "series" form.

```pine
//@version=6
indicator("map example", overlay=true)
map<int, float> a = na
a := map.new<int, float>()
a.put(bar_index, close)
label.new(bar_index, a.get(bar_index), "Current close")
```

### matrix
Two-dimensional arrays. Always of "series" form.

```pine
//@version=6
indicator("matrix example")
matrix<int> m1 = matrix.new<int>(2, 3, 0)
if barstate.islastconfirmedhistory
    label.new(bar_index, high, str.tostring(m1))
```

---

## Drawing Types

All drawing types (line, label, box, table, linefill, polyline) are always of "series" form.

### line
```pine
//@version=6
indicator("line example")
var line line1 = na
line3 = line.new(bar_index - 1, high, bar_index, high, extend = extend.right)
```

### label
```pine
//@version=6
indicator("label example")
var label label1 = na
if barstate.islastconfirmedhistory
    label.new(bar_index, high, text = "label text")
```

### box
```pine
//@version=6
indicator("box example")
var box box1 = na
box3 = box.new(time, open, time + 60 * 60 * 24, close, xloc=xloc.bar_time)
```

### table
```pine
//@version=6
indicator("table example")
var table t = na
if barstate.islastconfirmedhistory
    t := table.new(position.top_right, 1, 1, color.yellow)
    table.cell(t, 0, 0, "text")
```

---

## Type Qualifiers

### const
Values established at compile time.

### simple
Values established at first bar, don't change across bars.

### series
Values that can change on every bar.

---

## Critical Rule: na Initialization

In Pine Script v6, na is typeless. You MUST explicitly declare the type.

### FORBIDDEN
```pine
x = na
var myLine = na
```

### REQUIRED
```pine
float x = na
int i = na
bool b = na
var line l = na
var label lb = na
```

---

## References

- [TradingView Pine Script v6 Type System](https://www.tradingview.com/pine-script-docs/language/type-system/)
