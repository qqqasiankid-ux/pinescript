# VERSION_POLICY.md  
## Pine Script Version Enforcement & Compatibility Rules

### Purpose

This document defines **strict versioning rules** for all Pine Script code, explanations, patterns, and references in this repository.

Its goals are to:
- Eliminate syntax drift
- Prevent mixed‑version hallucinations
- Enforce deterministic behavior
- Ensure long‑term stability for AI‑generated code

All AI agents and contributors **must obey this policy**.

---

## Canonical Target Version

- **Primary and default target:** Pine Script **v6**
- All generated or edited code **must assume v6 semantics**
- Any deviation must be explicitly labeled and justified

---

## Mandatory Version Declaration

Every Pine Script file or snippet **must begin with**:

```pine
//@version=6
```

### Forbidden
- Omitting the version directive
- Inferring the version implicitly
- Using version‑agnostic examples

If the version directive is missing, the code is invalid.

---

## Mixed‑Version Rules (Strict)

### Forbidden
- Mixing v4/v5/v6 syntax in the same example
- Referencing older behavior without labeling it
- Using deprecated keywords without documentation

### Allowed (With Label)
- Referencing older behavior **only** if clearly marked:

```md
LEGACY (v5 and earlier):
Description of behavior that no longer applies in v6.
```

---

## Type System Enforcement

### `na` Initialization Rules (Critical)

In Pine Script v6, `na` is **typeless**.

#### Forbidden
```pine
x = na
var myLine = na
```

#### Required
```pine
float x = na
int i = na
bool b = na
string s = na

var line   l = na
var label  lb = na
var box    bx = na
var table  t = na
```

All examples in this repository **must follow this rule**.

---

## Function Namespace Policy

- Prefer official namespaces (`ta.*`, `math.*`, `request.*`, `strategy.*`)
- Avoid manual implementations when a built‑in exists
- Never alias functions in a way that hides their namespace

Example (preferred):
```pine
rsi = ta.rsi(close, 14)
```

Example (discouraged unless justified):
```pine
rsi = 100 - (100 / (1 + rs))
```

---

## Strategy vs Indicator Separation

### Indicators
- Use `indicator()`
- Must not contain order placement logic
- Visualization only

### Strategies
- Use `strategy()`
- Order placement must follow **v6 strategy syntax**
- Backtesting behavior must be explicit

Never mix indicator and strategy semantics in one script.

---

## Execution Model Awareness

All content must respect Pine Script v6 execution rules:

- Single‑pass per bar execution
- Historical bars vs realtime bar differences
- `barstate.*` semantics
- Object persistence rules (`var` vs non‑`var`)

Any example relying on execution quirks must document them.

---

## request.security() Rules

- Must explicitly document repainting behavior
- Must specify `barmerge` behavior when relevant
- Must clarify whether lookahead is allowed or forbidden

Silent repainting is forbidden.

---

## Object Lifecycle Rules

For `line`, `label`, `box`, and `table`:

- Creation must be explicit
- Updates must reference existing objects
- Deletion must be intentional
- Object count limits must be respected

Leaking objects is considered a defect.

---

## Array & Matrix Rules

- Arrays must be bounds‑safe
- Indexing must be guarded
- Empty array access is forbidden
- Dynamic growth must be intentional

Any array example must handle the zero‑length case.

---

## Deprecation Handling

When Pine Script changes behavior:

- Old behavior is marked **LEGACY**
- Replacement behavior is documented
- Rationale is explained
- Old content is not silently removed

---

## Validation Requirement

Before code or documentation is accepted as correct:

- It must compile under Pine Script v6
- It must not rely on undocumented behavior
- It must not require TradingView experimental flags
- It must not exploit undefined execution order

If validation cannot be confirmed → content must be flagged.

---

## Enforcement

Violations of this policy require:

- Immediate correction, or
- Explicit refusal to generate code, or
- Clear documentation of uncertainty

Ignoring this policy invalidates the output.

---

## Delegated Version Authority (AI‑Maintained)

### Intent

This repository is designed to evolve alongside Pine Script.  
Accordingly, **authorized AI agents may update version‑related policies** when Pine Script behavior, syntax, or semantics change.

This section defines **how** that authority is exercised safely.

---

## Authorized AI Updates (Version Policy)

AI agents **are permitted** to update `VERSION_POLICY.md` **only if all conditions below are met**:

1. The change is based on **verified Pine Script documentation or compiler behavior**
2. The change reflects:
   - A new Pine Script version
   - A syntax change
   - A deprecation
   - A behavioral clarification
3. The update **does not silently invalidate existing knowledge**
4. The update includes proper documentation and changelog entries

---

## Update Requirements (Mandatory)

When modifying version policy, the AI **must**:

1. **Preserve historical context**
   - Previous behavior must be retained and labeled `LEGACY`
   - No silent deletions

2. **Explicitly declare the new state**
   - New version number
   - Scope of change
   - Impacted syntax or behavior

3. **Annotate uncertainty**
   - If behavior is inferred from observation rather than documentation, it must be marked:
     ```
     STATUS: OBSERVED (NOT FORMALLY DOCUMENTED)
     ```

4. **Update the changelog**
   - Date
   - Reason for change
   - Source of truth (docs, compiler, release notes)
   - Risk notes (if any)

---

## Prohibited AI Actions (Even With Authority)

Even authorized AI agents may **not**:

- Guess future Pine Script behavior
- Remove safeguards to “simplify” rules
- Rewrite version policy wholesale without justification
- Mix speculative syntax with confirmed syntax
- Treat beta or experimental features as stable

---

## Conflict Resolution

If new Pine Script behavior conflicts with existing repository rules:

1. The AI must **add an override note**
2. The conflict must be documented explicitly
3. The old rule must remain until formally deprecated

Example:
```md
NOTE: As of Pine Script vX.Y, this rule no longer applies.
LEGACY behavior retained below for reference.
```

---

## Self‑Correction Clause

If an AI later determines that a prior update was incorrect:

- The correction **must not erase history**
- The incorrect section must be annotated:
  ```
  CORRECTION: Previous guidance was inaccurate due to <reason>.
  ```
- A new changelog entry is required

---

## Final Delegation Statement

This repository is **AI‑maintained, not AI‑guessed**.

AI agents are empowered to update version policy **only when certainty exceeds speculation**.

When certainty is insufficient:
- Document
- Annotate
- Defer

Never invent.

---

End of version policy.
