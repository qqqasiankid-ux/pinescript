# Safe Initialization Patterns

## Status
- STATUS: CANONICAL
- CONFIDENCE: HIGH
- LAST_UPDATED: 2026-01-18
- SOURCE: VERSION_POLICY, qqqasiankid-ux/ok repository

## Scope

This file documents the critical na initialization rules in Pine Script v6 and safe patterns for variable declaration.

---

## Canonical Rules

1. In Pine Script v6, na is typeless
2. ALL variables initialized with na MUST have explicit type declarations (regardless of whether they use var, varip, or no modifier)

---

## FORBIDDEN Patterns

```pine
x = na
var myLine = na
```

## REQUIRED Patterns

```pine
//@version=6
indicator("Safe initialization")

float x = na
int i = na
bool b = na
var line l = na
var label lb = na
var box bx = na
```

---

## References

- [Pine Script v6 Type System](https://www.tradingview.com/pine-script-docs/language/type-system/)
