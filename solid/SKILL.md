---
name: solid-principles
description: Enforce strict SOLID compliance (SRP, OCP, LSP, ISP, DIP) in all object-oriented and modular code. Use when designing classes, modules, services, interfaces, or refactoring existing structures. Invoke before any code-write or code-edit action that defines or modifies a class, module boundary, or interface.
---

# SOLID Principles — Strict Enforcement

Every class, module, and interface MUST comply with all five SOLID principles. Violations not deferred — fixed in the same change.

## S — Single Responsibility Principle

Class/module has ONE reason to change. ONE actor it serves.

### Mandatory Rules
- One class = one cohesive responsibility (e.g., `UserRepository` persists users; does not also send emails).
- If you describe a class with "and" or "also" — split it.
- God classes (200+ lines, many unrelated methods) — split.
- Change in business policy A and change in business policy B must not touch the same class.

### Triggers to split
- Class name contains `Manager`, `Helper`, `Util`, `Processor` without further qualification → likely doing too much.
- Methods operate on disjoint subsets of fields → split along the seams.
- Mixed concerns: persistence + HTTP + validation + business rules in one place → split into layers.

## O — Open / Closed Principle

Software entities OPEN for extension, CLOSED for modification.

### Mandatory Rules
- Adding new variant of behavior MUST NOT require editing existing stable code.
- Use polymorphism, strategy, plugin registration, or composition — not growing `switch`/`if` chains.
- Stable abstractions (core domain interfaces) edited only on real domain change, never to bolt on a new case.

### Triggers
- New `case` added to existing `switch` for every new type → refactor to polymorphism / registry.
- `if (type === 'A') ... else if (type === 'B') ...` ladders → replace with table-driven dispatch.
- Patching base class to support subclass-specific feature → push feature into subclass or compose.

## L — Liskov Substitution Principle

Subtypes MUST be substitutable for base types without breaking correctness.

### Mandatory Rules
- Subclass cannot strengthen preconditions.
- Subclass cannot weaken postconditions.
- Subclass cannot change invariants of base type.
- Subclass cannot throw new unchecked exceptions caller wasn't prepared for.
- No `if (instanceof Subtype) throw` — Liskov violation in disguise.

### Triggers
- Method overrides that throw `UnsupportedOperationException` / `NotImplementedError` → hierarchy wrong; remodel via composition or split interfaces.
- Subclass requires more setup than parent → preconditions strengthened, violates LSP.
- Subclass return values have narrower contract → postconditions weakened.
- Square-extends-Rectangle style mistakes → use separate types and shared interface for shared behavior, not inheritance.

## I — Interface Segregation Principle

Clients MUST NOT be forced to depend on methods they do not use.

### Mandatory Rules
- Prefer many small, role-specific interfaces over one large interface.
- Consumer depends only on methods it actually calls.
- No "fat" service interfaces with 20 methods serving 5 unrelated consumers.

### Triggers
- Implementer of an interface has half its methods as `throw new NotImplemented` → split the interface.
- Mock setups in tests require stubbing methods the test never invokes → split the interface.
- One interface consumed by N callers, each using a different subset → split per role.

## D — Dependency Inversion Principle

High-level modules MUST NOT depend on low-level modules. Both depend on abstractions. Abstractions MUST NOT depend on details.

### Mandatory Rules
- Business logic depends on interfaces (or types), not concrete infrastructure.
- Concrete dependencies (DB, HTTP client, file system, clock, RNG, third-party SDK) injected, not instantiated inside business logic.
- Composition root wires concretes to interfaces — domain code stays infrastructure-free.

### Triggers
- `new PostgresClient()` inside a domain/service class → invert via injected interface.
- Domain code `import`s an HTTP library or ORM directly → push that behind a port.
- Static singletons used inside business logic → inject through the constructor / function args.
- Tests need to monkey-patch globals to run → DIP missing; refactor.

## Workflow

When designing a class/module:
1. State its single responsibility in one sentence. If you need "and" — split.
2. Define which axes of change it must absorb without modification (OCP).
3. If part of hierarchy: verify substitutability (LSP).
4. Define minimal interfaces per consumer role (ISP).
5. Wire dependencies through abstractions; concretes injected at composition root (DIP).

When editing existing code:
1. If extending behavior would require editing switch / type-test → refactor to polymorphism first, then add.
2. If adding method to interface for one consumer → split the interface first.
3. If forced to instantiate infrastructure in a domain class → inject it.

## Red Flags

- "I'll just add another `if` for the new type" → OCP violation. Refactor to dispatch.
- "Class is small now, it'll be fine" → still split if responsibilities differ.
- "Subclass throws NotImplemented for that method" → LSP violation. Redesign.
- "We need this method on the interface for one caller" → ISP violation. Split.
- "It's easier to `new` it here than thread a dependency" → DIP violation. Thread the dependency.

## Verification

After every change, verify:
- Each touched class describes its responsibility in one short sentence without "and".
- No new `if/switch` chains added for type discrimination where polymorphism applies.
- No subclass weakens or strengthens contracts.
- No consumer forced to know methods it does not call.
- No domain/business class instantiates infrastructure directly.

Code is SOLID-compliant ONLY when every principle holds simultaneously.
