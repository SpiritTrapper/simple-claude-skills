---
name: dry-principle
description: Enforce strict DRY (Don't Repeat Yourself) compliance in all code. Use when writing, editing, or reviewing code to detect and eliminate duplicated logic, repeated literals, copy-paste patterns, parallel structures, and redundant abstractions. Invoke before any code-write and code-edit action.
---

# DRY Principle — Strict Enforcement

Every piece of knowledge MUST have a single, unambiguous, authoritative representation within the system.

## Mandatory Rules

1. **No duplicated logic.** If the same logic appears in 2+ places — extract immediately. No "I'll refactor later".
2. **No copy-paste.** Copying a block of code is a violation. If you need it elsewhere, extract first, then reuse.
3. **No repeated literals.** Magic numbers, repeated strings, repeated config values → constants/enums/config files.
4. **No parallel structures.** If two switch/if-chains track the same domain, collapse to one source of truth (map, registry, polymorphism).
5. **Single source of truth.** Types, schemas, validation rules, error codes, route paths — defined once, imported everywhere.
6. **Shared utilities live in shared modules.** Never re-implement formatting, parsing, date math, HTTP wrappers, error handling per file.
7. **Reuse over rewrite.** Before writing a new helper, grep the codebase. Use existing utility if exists.

## Detection Triggers (STOP and refactor when seen)

- Same 3+ lines repeated in 2+ files
- Same string literal in 3+ places
- Same magic number in 2+ places (except `0`, `1`, `-1` for trivial use)
- Same type/interface declared in 2+ files
- Same validation rule expressed differently in 2+ layers
- Same error message string scattered across handlers
- Same conditional structure with different data — extract data-driven dispatch
- Same SQL/query shape with minor variants — parameterize

## Exceptions (Rule of Three Pragmatism)

DRY is NOT premature abstraction. Allowed:
- Two occurrences that LOOK similar but represent different concepts — keep separate (coincidental duplication ≠ semantic duplication).
- Test fixtures may repeat for clarity if abstraction obscures intent.
- Generated code / boilerplate where extraction breaks tooling.

If unsure: duplicate twice, extract on third occurrence with confirmed shared semantics.

## Workflow

Before writing new code:
1. Grep for similar names, signatures, behavior.
2. Check shared utility folders (`utils/`, `lib/`, `shared/`, `common/`).
3. If similar exists → reuse or extend it.
4. If similar exists but not quite fitting → refactor existing to cover both cases, don't fork.

After writing code, scan diff for:
- Repeated literals → extract to constants
- Repeated blocks → extract to function/method
- Repeated patterns → extract to abstraction (strategy, template method, generic)

## Red Flags

- "I'll just copy this and tweak it" → STOP. Extract first.
- "It's only twice" → still extract if semantics are identical.
- "It's faster to duplicate" → false economy. Duplicate code creates bug surface area.
- "The other version is in a different layer" → cross-layer duplication is worst kind. Share through contract module.

## Verification

After every code change, verify:
- No new repeated string literals across files
- No new repeated logic blocks (5+ lines)
- No new parallel type/schema declarations
- All new constants live in constants module if used in 2+ files

Code is DRY-compliant ONLY when every piece of knowledge has exactly one home.
