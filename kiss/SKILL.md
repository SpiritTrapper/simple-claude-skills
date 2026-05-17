---
name: kiss-principle
description: Enforce strict KISS (Keep It Simple, Stupid) compliance in all code. Use when writing, editing, or reviewing code to prevent over-engineering, premature abstraction, unnecessary patterns, excessive indirection, clever code, and speculative generality. Invoke before any code-write and code-edit action.
---

# KISS Principle — Strict Enforcement

Simplicity is mandatory. Simplest solution that fully solves problem wins. Clever code loses.

## Mandatory Rules

1. **Solve actual problem.** Not hypothetical future problem. Not generic case. Concrete current requirement.
2. **Shortest correct path.** Fewest lines, fewest files, fewest abstractions, fewest moving parts that fully satisfy spec.
3. **Boring beats clever.** 5-line plain loop beats 1-line ternary chain with reduce + map + flat. Readability wins.
4. **No speculative generality.** Don't build for "what if we need X later". Add X when X actually needed.
5. **No premature abstraction.** Don't create interfaces, factories, or base classes for single implementation.
6. **Flat over nested.** Prefer linear flow. Early returns over deep if-else trees. Max 2 levels of nesting in functions.
7. **Direct over indirect.** No middleman classes, no pass-through wrappers, no "manager-of-managers".
8. **Standard tools first.** Use language stdlib / framework primitives before pulling new libraries or rolling custom ones.

## Detection Triggers (STOP and simplify when seen)

- Function longer than ~30 lines doing multiple things → split or simplify
- Class with 1 implementation behind interface → drop the interface
- Factory/Builder/Strategy/Visitor for trivial logic → replace with plain function
- Configuration object with 10+ optional flags → too many concerns, split feature
- Generic `<T, U, V>` types when concrete type would do → use concrete
- Inheritance chain >2 deep → flatten or compose
- File with 4+ abstraction layers (controller → service → use-case → repository → gateway → adapter) for CRUD → collapse layers
- Custom DSL/framework wrapping 5-line stdlib call → drop the wrapper
- Premature caching, premature parallelism, premature batching → remove until proven needed

## Allowed Complexity

Complexity allowed ONLY when:
- Performance profiling proves simple version too slow
- Multiple real consumers exist for the abstraction (not hypothetical)
- Problem domain inherently complex (parser, compiler, scheduler)
- Well-known pattern is simplest correct expression of the problem

## Workflow

Before writing code:
1. Ask: what is minimum that makes this work correctly?
2. Write that.
3. If tempted to add flag, option, or extension point — DON'T. Add on real demand.

After writing code:
1. Read diff as if never seen.
2. If any part requires comment to explain WHAT it does → rewrite simpler.
3. If junior dev wouldn't understand in 30 seconds → rewrite simpler.
4. Count abstractions. Remove any with only one user.

## Red Flags

- "We might need this later" → YAGNI. Delete.
- "This makes it more flexible" → flexibility costs. Pay only when buying real reuse.
- "It's a known pattern" → patterns ≠ correctness. Use only when problem matches pattern exactly.
- "It's more elegant this way" → elegance ≠ simplicity. If others can't read it, it's not elegant.
- "It's only one extra layer" → layers compound. Each adds cognitive cost.
- One-line clever functional chain nobody can debug → expand to readable steps.

## Verification

After every code change, verify:
- No new abstractions with only one implementer
- No new config flags without immediate real use case
- No new wrapper / facade / proxy without justified boundary
- No new generic types when concrete fit
- Cyclomatic complexity per function low (target ≤ 7)
- Function length bounded (target ≤ 30 lines)
- Nesting depth bounded (target ≤ 3)

Code is KISS-compliant ONLY when every line earns its place and removing anything breaks the feature.
