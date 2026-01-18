# Drawing Functions

## Status
- STATUS: CANONICAL
- CONFIDENCE: MEDIUM
- LAST_UPDATED: 2026-01-18

## Scope

This document describes Pine Script v6 drawing functions for plotting series data on charts.  In particular, it focuses on the `plot()` function and related parameters that control how data is rendered.  While other drawing helpers like `bgcolor()` and `plotchar()` exist, this page concentrates on the most common case: plotting indicator values as lines, histograms or shapes.

## Canonical Rules

The `plot()` function is the primary way to draw time‑series data in Pine scripts.  Its signature is:

```
plot(series, title, color, linewidth, style, trackprice,
     histbase, offset, join, editable, show_last, display, force_overlay) → plot
```

Key rules derived from TradingView’s official documentation【349325309863596†L264-L300】:

1. **Series argument** – The first argument is mandatory and must be of type “series int/float”【349325309863596†L273-L280】.  Booleans must be cast to int/float before plotting.  
2. **Title** – Must be a compile‑time constant string【349325309863596†L283-L295】.  It appears in the Data Window and script settings.  
3. **Color** – Accepts a series of type color【349325309863596†L296-L300】.  Use `na` or a color with 100‑level transparency to hide plots conditionally.  
4. **Line width** – Controls the thickness in pixels for line and histogram styles【349325309863596†L302-L307】.  Some styles ignore this parameter.  
5. **Style enumeration** – Selects how values are rendered (e.g., `plot.style_line`, `plot.style_histogram`, `plot.style_circles`, etc.)【349325309863596†L308-L351】.  Note that some styles (circles and crosses) are not joined across bars unless `join = true`【349325309863596†L348-L351】.  
6. **Track price** – When `true`, draws a horizontal dotted line at the last plotted value【349325309863596†L353-L357】.  
7. **Histbase** – Baseline used for area and column styles【349325309863596†L360-L364】.  
8. **Offset** – Shifts the plot forwards or backwards along the time axis.  Offset values must be constants and cannot change across bars【349325309863596†L369-L370】.  
9. **Join** – Applicable only to `plot.style_circles` and `plot.style_cross`.  When set to `true`, adjacent shapes are connected with a one‑pixel line【349325309863596†L372-L376】.  
10. **Editable** – Controls whether users can edit the plot’s style in the indicator settings.  Default is `true`【349325309863596†L377-L380】.  
11. **Show_last** – Limits plotting to the last `N` bars.  Requires an `input int` and cannot be dynamic【349325309863596†L382-L386】.  
12. **Display** – Governs whether a plot is drawn.  Setting `display = display.none` hides the plot completely【349325309863596†L387-L393】.  Hidden plots can still provide values for alerts or downstream scripts.

In addition to `plot()`, the following helpers are often used:

- **`bgcolor(color)`** – Sets the background color of the script’s visual pane for the current bar.  Commonly used to highlight conditions.  
- **`plotchar(series, title, text, location, color, size)`** – Draws characters or symbols on the chart.  

## Examples (v6)

### Plotting moving averages with custom parameters

The following script plots two exponential moving averages (EMA) and colors the chart background when the shorter EMA crosses over or under the longer EMA.  It demonstrates how to set titles, colors and styles, and how to use the `join`, `editable`, `show_last` and `display` parameters.

```pine
//@version=6
indicator("EMA crossover demo", overlay = true)
float emaFast = ta.ema(close, 50)
float emaSlow = ta.ema(close, 100)

// Detect crossovers
bool emaCrossOver  = ta.crossover(emaFast, emaSlow)
bool emaCrossUnder = ta.crossunder(emaFast, emaSlow)

// Plot EMAs with titles and colors
plot(emaFast,  "EMA 50",  color = color.green, linewidth = 2)
plot(emaSlow,  "EMA 100", color = color.red,   linewidth = 2)

// Shade the background on crosses
bgcolor(emaCrossOver  ? color.new(color.green, 90) :
        emaCrossUnder ? color.new(color.red,   90) : na)

// Plot a debug series that is hidden by default
float diff = emaFast - emaSlow
plot(diff, title="Difference", color = color.blue,
     display = display.none, show_last = 100)
```

This example uses:

- `color.new(c, transparency)` to control plot transparency.  
- `bgcolor()` to highlight cross events with translucent shading.  
- `display = display.none` to hide a debug plot while still computing its values.  
- `show_last = 100` to only keep the last 100 values of the hidden plot.  

## Pitfalls / Failure Modes

1. **Type mismatches** – Booleans cannot be plotted directly; convert them to numbers (e.g., `condition ? 1 : 0`)【349325309863596†L273-L280】.  
2. **Unused optional parameters** – Many scripts leave optional parameters at their defaults; unnecessary customization can clutter code.  
3. **Incorrect use of `join`** – The `join` flag only affects `plot.style_circles` and `plot.style_cross`【349325309863596†L372-L376】.  It has no effect on other styles.  
4. **`show_last` limitations** – The argument must be an integer known at compile time【349325309863596†L382-L386】.  Dynamically sizing this parameter results in a compile error.  
5. **Hidden plots and scale** – Using `display = display.none` hides a plot and removes it from the chart’s scale【349325309863596†L387-L393】.  Hidden plots are ideal for supplying data to alerts but cannot be inspected visually without toggling their display in the settings.  

## References

- TradingView Pine Script manual – Plotting functions and parameters【349325309863596†L264-L300】【349325309863596†L372-L376】.  
- TradingView Pine Script manual – Plot styles and limitations【349325309863596†L308-L351】.  
