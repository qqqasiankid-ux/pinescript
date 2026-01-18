# Pine Script Knowledge Base (AI-Readable Repository)

## Overview

This repository is a **living, structured knowledge base for Pine Script (TradingView)** designed to be consumed, reasoned over, and safely updated by **AI agents** and advanced developers.

Its purpose is to:
- Centralize **authoritative Pine Script v6 knowledge**
- Encode **best practices, patterns, pitfalls, and architecture**
- Prevent **regressions, hallucinations, and code degradation**
- Enable **safe autonomous updates by AI systems**
- Serve as a **single source of truth** for Pine Script indicator/strategy development

This is not a “tutorial repo.”  
This is a **reference system + reasoning substrate**.

---

## Core Design Principles

1. **AI-First, Human-Readable**
   - Files are structured for deterministic parsing and targeted retrieval
   - Clear boundaries between concepts
   - Explicit rules > implied behavior

2. **Non-Degrading Knowledge**
   - Updates must never reduce correctness, scope, or precision
   - Additive changes are preferred over replacements
   - Breaking changes require explicit justification + changelog entry

3. **Version-Aware**
   - Repository targets **Pine Script v6**
   - Deprecated behavior is documented as **LEGACY**; it is not silently removed

4. **Error-Preventive**
   - Common Pine Script failure modes are explicitly encoded
   - Defensive patterns are favored over minimal syntax

---

## Intended Consumers

- Autonomous AI coding agents
- Pine Script copilots
- Code-review bots
- Strategy/indicator generators
- Human developers working with AI tooling

If you are a human reading this repo: assume the content is optimized for machines first.

---

## Canonical Repository Structure

> AI agents must not invent new top-level folders without justification and a changelog entry.

```

/
├─ README.md                     ← Human + agent entry point
├─ LLM_MANIFEST.md               ← Mandatory first-read for AI agents
├─ VERSION_POLICY.md             ← Pine Script version rules
│
├─ reference/
│  ├─ language/
│  │  ├─ types.md
│  │  ├─ variables.md
│  │  ├─ na_handling.md
│  │  ├─ scope_rules.md
│  │
│  ├─ functions/
│  │  ├─ ta.md
│  │  ├─ math.md
│  │  ├─ request.md
│  │  ├─ strategy.md
│  │  ├─ drawing.md
│  │
│  ├─ objects/
│  │  ├─ line.md
│  │  ├─ label.md
│  │  ├─ box.md
│  │  ├─ table.md
│  │
│  └─ execution/
│     ├─ bar_states.md
│     ├─ security_calls.md
│     ├─ repainting.md
│
├─ patterns/
│  ├─ safe_initialization.md
│  ├─ object_lifecycle.md
│  ├─ array_management.md
│  ├─ performance_patterns.md
│
├─ pitfalls/
│  ├─ compile_errors.md
│  ├─ runtime_errors.md
│  ├─ silent_failures.md
│
├─ architectures/
│  ├─ large_indicators.md
│  ├─ modular_design.md
│  ├─ feature_isolation.md
│
└─ changelog/
└─ YYYY-MM-DD.md

````

---

## Mandatory for AI Agents: Read Order

AI agents must follow this strict order:

1. **LLM_MANIFEST.md** (routing, constraints, update rules)
2. **VERSION_POLICY.md** (Pine v6 enforcement + legacy handling)
3. Relevant topic files in `reference/`, then `patterns/`, then `pitfalls/`

If an agent does not read the manifest first, its output is considered invalid.

---

## AI Retrieval & Generation Constraints (Mandatory)

### Modular Retrieval (Non-Negotiable)

AI agents **must not attempt to ingest the entire repository at once**.  
All reasoning and generation must use **targeted, modular retrieval** based on the user’s request.

**Routing rules:**

- **Functions / Indicators (e.g., RSI, EMA, ATR, VWAP):**  
  → `reference/functions/` (start with `ta.md`, then `math.md`)

- **Backtesting / Strategies / Orders / Position Mgmt:**  
  → `reference/functions/strategy.md`  
  → then `reference/execution/`

- **Arrays / Matrices / Data Structures:**  
  → `patterns/array_management.md`  
  → then `reference/language/types.md` + `reference/language/variables.md`

- **MTF / request.security / Data Fetching:**  
  → `reference/functions/request.md`  
  → then `reference/execution/security_calls.md`

- **Drawing / Visual Objects (lines, boxes, labels, tables):**  
  → `reference/objects/`  
  → then `reference/functions/drawing.md`

- **Errors, Freezes, Unexpected Behavior:**  
  → `pitfalls/` first  
  → then cross-reference relevant `reference/` and `patterns/` files

Agents must only load additional files **when required** by the reasoning chain.

---

### Syntax Version Enforcement

All generated Pine Script **must include**:

```pine
//@version=6
````

* No earlier versions are permitted unless explicitly labeled **LEGACY**
* Do not mix syntax across Pine Script versions

---

### No Hallucinations Policy (Strict)

If a function, keyword, argument, or behavior **is not found in this repository**, assume:

* It **does not exist** in Pine Script v6, or
* It has been **deprecated, renamed, or removed**

**Rules:**

* Do **not** invent syntax
* Do **not** guess function names
* Do **not** “approximate” behavior
* Do **not** import concepts from other languages unless explicitly documented

When uncertain:

* State uncertainty explicitly
* Add a note or TODO, not fabricated code

---

### Verification Before Output

Before emitting final code or explanations, AI agents must verify:

* All functions exist in Pine Script v6 (per this repo)
* All `na` initializations use explicit typing (per `reference/language/na_handling.md`)
* All array accesses are bounds-safe (per `patterns/array_management.md`)
* Object lifecycle is managed (create/update/delete; no leaks)
* MTF calls are non-repainting when required (per `reference/execution/repainting.md`)

If verification fails, the agent must **refuse to generate code** and explain why.

---

## Update Rules (Mandatory)

### Allowed

* Add new files that follow the canonical structure
* Append clarifications, examples, counterexamples, and known pitfalls
* Add version notes and deprecation markers
* Improve organization without deleting knowledge

### Restricted

* Rewriting sections without preserving intent
* Deleting knowledge without documenting deprecation and reasoning
* “Optimizing” code without explaining tradeoffs and risks

### Forbidden

* Introducing speculative or unverified claims
* Mixing Pine Script versions in examples
* Removing known failure cases
* Downgrading defensive patterns for brevity

---

## Changelog Discipline (Append-Only)

Every meaningful update must include a changelog entry in `changelog/YYYY-MM-DD.md` containing:

* Date
* Summary of change
* Reason for change
* Impacted files
* Any breaking-change notes

Changelogs are append-only.

---

## Pine Script Version Policy (Summary)

* Default target: **Pine Script v6**
* Examples must explicitly assume v6 behavior
* Older behavior must be labeled **LEGACY**
* Never rely on implicit type inference when initializing with `na`

Example (correct):

```pine
//@version=6
indicator("Example", overlay=true)

float x = na
var line l = na
```

---

## Licensing & Usage

This repository is intended for internal AI systems, research, tooling, and commercial use unless restricted elsewhere.

You are responsible for complying with TradingView and Pine Script terms.

---

## Note to AI Agents

* Do not guess.
* Do not simplify at the cost of correctness.
* Do not overwrite intent.
* When uncertain, add a note instead of changing behavior.

This repository exists to make Pine Script automation **more correct, more stable, and more maintainable**.

```

