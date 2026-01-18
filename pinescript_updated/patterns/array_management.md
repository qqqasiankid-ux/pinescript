# Array Management

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18

## Scope

This document describes how to work with one‑dimensional arrays in Pine Script v6.  Arrays allow scripts to store multiple values of the same type in a single container, facilitating dynamic data structures, buffers and custom calculations.  Because arrays are advanced features, readers should be comfortable with basic Pine syntax before using them.

## Canonical Rules

Pine Script arrays are governed by the following rules【22696357522030†L173-L193】【22696357522030†L203-L233】:

1. **One‑dimensional collections** – Arrays hold multiple value references of a single type【22696357522030†L173-L183】.  They cannot nest arrays within arrays.  
2. **Homogeneous types** – All elements must be of the same built‑in, user‑defined or enum type【22696357522030†L180-L181】.  Attempting to mix types will cause a compile error.  
3. **Array IDs** – Arrays are referenced by an ID similar to line and label IDs【22696357522030†L183-L187】.  Pine does not use the bracket `[]` operator to index arrays; instead, functions such as `array.get()` and `array.set()` read and write values【22696357522030†L183-L187】.  
4. **Indexing** – Indices start at 0 and go up to `size - 1`【22696357522030†L189-L193】.  Passing an out‑of‑range index to `array.get()` or `array.set()` triggers a runtime error.  
5. **Dynamic size** – Arrays can grow or shrink during script execution, and scripts may contain multiple arrays【22696357522030†L189-L193】.  The maximum size is 100 000 elements.  
6. **Declaration syntax** – Use `array<type>` or `type[]` in a variable declaration to indicate an array.  The right‑hand expression must return an array or `na`【22696357522030†L203-L233】.  
7. **Creation functions** – Arrays are created using functions in the `array` namespace:  
   - `array.new<type>(size, initial_value)` – Creates an array with a fixed size and fills elements with `initial_value`【22696357522030†L231-L244】.  
   - `array.from(v0, v1, …)` – Creates an array from a list of values【22696357522030†L256-L266】.  
   - `array.copy(id)` – Copies an existing array (not shown).  
   Use type‑specific variations like `array.new_int()`, `array.new_float()`, etc.  
8. **Persistence** – Declaring an array with the `var` keyword ensures it is created once on the first bar and persists across bars【22696357522030†L269-L279】.  Arrays declared with `varip` behave similarly but update on each tick for realtime bars【22696357522030†L293-L299】.  
9. **Reading and writing** – Use `array.get(id, index)` to read and `array.set(id, index, value)` to write elements【22696357522030†L301-L307】.  Use `array.size(id)` to determine the current length.  
10. **Appending and inserting** – `array.push(id, value)` appends to the end, increasing the size by one【22696357522030†L335-L346】.  `array.unshift(id, value)` inserts at the beginning【22696357522030†L351-L356】.  

## Examples (v6)

### Creating and using a float array

```pine
//@version=6
indicator("Array example", overlay = false)

// Declare a persistent float array with initial elements
var float[] priceArray = array.from(0.1, 2.5, 5.0)

// Display array information on the last bar
if barstate.islast
    // Show the array size
    string txt = "Array size: " + str.tostring(array.size(priceArray)) + "\n"
    // Get the first and last values using array.get()
    txt := txt + "First: " + str.tostring(array.get(priceArray, 0)) + "\n"
    txt := txt + "Last:  " + str.tostring(array.get(priceArray, array.size(priceArray) - 1))
    label.new(bar_index, high, txt)

// Modify an element on the last bar
if barstate.islast
    array.set(priceArray, 2, 33.0)  // set the third element to 33.0
```

This script declares an array with three float values using `array.from()`.  On the last bar it displays the array’s size and element values using `array.size()` and `array.get()`.  It then modifies the third element with `array.set()`.

### Dynamically growing an array with `var`

```pine
//@version=6
indicator("Growing array", overlay = false)
// Create a persistent array that grows by one element each bar
var float[] values = array.new_float(0)
// Append the current close price
array.push(values, close)
// On the last bar, show the array size and the most recent value
if barstate.islast
    label.new(bar_index, high,
        "Size: " + str.tostring(array.size(values)) +
        "\nLatest: " + str.tostring(array.get(values, array.size(values) - 1)))
```

In this example `var` ensures that the `values` array persists across bars.  A new element (the current close price) is appended on each bar with `array.push()`.

## Pitfalls / Failure Modes

1. **Out‑of‑range errors** – Calling `array.get()` or `array.set()` with an index ≥ `array.size()` results in a runtime error【22696357522030†L189-L193】.  Always check the size before indexing.
2. **Memory consumption** – Arrays can grow to 100 000 elements【22696357522030†L189-L193】.  Large arrays may increase script execution time or memory usage.
3. **Incorrect types** – Mixing element types or omitting the type in declarations may cause compile‑time errors【22696357522030†L180-L183】.
4. **Redundant re‑creation** – Declaring an array without `var` will recreate it on each bar, resetting its contents.  Use `var` or `varip` when persistent data is required【22696357522030†L269-L279】.

## References

- TradingView Pine Script manual – Arrays introduction and constraints【22696357522030†L173-L193】.
- TradingView Pine Script manual – Declaring and creating arrays【22696357522030†L203-L233】.
- TradingView Pine Script manual – Reading and writing array elements【22696357522030†L301-L307】.
- TradingView Pine Script manual – Adding elements to arrays【22696357522030†L335-L346】.
