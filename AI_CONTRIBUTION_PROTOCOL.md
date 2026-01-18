# AI_CONTRIBUTION_PROTOCOL.md
## How AI Agents Add or Update Knowledge in This Repository

### Goal

AI agents should expand this repository with accurate Pine Script v6 knowledge while preventing regressions, hallucinations, and silent breaking changes.

---

## What You Are Allowed To Do

- Populate stub files using verified information
- Add new topic files that match the canonical structure
- Append clarifications, examples, and pitfalls
- Add **LEGACY** sections instead of deleting outdated info
- Improve organization without removing intent

---

## What You Must Not Do

- Invent Pine Script syntax or functions
- Guess behavior that cannot be verified
- Delete knowledge without deprecation notes
- Rewrite entire documents without preserving history
- Add new top‑level folders without justification

---

## Required Workflow

1. Determine user intent and relevant file targets
2. Retrieve only the minimal required files
3. Add or update content
4. Mark uncertainty explicitly:
   - STATUS: VERIFIED / OBSERVED / SPECULATIVE (SPECULATIVE is forbidden for code)
5. Add a changelog entry documenting:
   - What changed
   - Why it changed
   - What files were impacted
   - Any risks or breaking changes

---

## Content Quality Standard

Every new section must include:

- Clear scope
- Canonical rules (bullet form)
- At least one working Pine v6 example
- At least one “pitfall” section describing common failures

---

## If You Cannot Verify

Do not generate code.
Instead:
- Add a TODO note
- Explain what source is needed to verify

---

End of contribution protocol.