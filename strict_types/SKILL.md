---
name: strict-types
description: Enforce strict, exhaustive typing across the entire application. Zero tolerance for `any`, implicit `any`, `unknown` left unrefined, untyped JSON, untyped function params/returns, untyped DB rows, untyped HTTP bodies, or escape hatches like `as any`, `@ts-ignore`, `# type: ignore`, `eval`, `Object`, `Function`. Use when writing or editing any code, defining APIs, DTOs, schemas, or function signatures. Invoke before any code-write or code-edit action in any typed language (TypeScript, Python, Go, Rust, Java, Kotlin, C#, Swift, Dart, etc.).
---

# Strict Types — Mandatory Total Typing

Every value MUST have precise, exact, narrow type. `any` and equivalent escape hatches FORBIDDEN.

## Mandatory Rules

### 1. Forbidden Constructs

Across all typed languages — never use:

- **TypeScript:** `any`, implicit `any`, `as any`, `as unknown as X` (when used to bypass), `Object`, `{}` (as "permissive" type), `Function`, `// @ts-ignore`, `// @ts-nocheck`, `// @ts-expect-error` without tracked issue, `noImplicitAny: false`, `strict: false`.
- **Python:** missing annotations on public functions/methods, `Any`, `# type: ignore` without tracked issue, `cast(Any, ...)`, dynamic attribute access without `Protocol` or `TypedDict`, `**kwargs` without typing, untyped `dict`/`list` for structured data.
- **Go:** `interface{}` / `any` for structured data, untyped maps for known shapes, untyped JSON decoding into `map[string]interface{}` for known schemas.
- **Java/Kotlin:** raw types (e.g., `List` without generic), `Object` as parameter for structured data, `!!` non-null assertions used to silence compiler, `@Suppress("UNCHECKED_CAST")`.
- **C#:** `dynamic`, `object` for structured data, `#nullable disable`, suppressing nullable warnings.
- **Rust:** `unsafe` outside FFI/perf-critical justified blocks, `Box<dyn Any>` for structured data.
- **Swift:** `Any`, `AnyObject` for structured data, force-unwrap `!` to bypass typing.

### 2. Required Constructs

- All public function/method signatures fully typed (parameters AND return types).
- All exported/public symbols fully typed.
- DTOs / request bodies / response bodies / event payloads → defined as schemas / classes / structs with precise field types.
- HTTP / queue / DB I/O boundaries validated and narrowed at the boundary (zod, valibot, pydantic, dataclasses, structs, etc.).
- Enum / union for closed sets of values — not strings.
- Discriminated unions for variant data — with exhaustive matching.
- Generics used to preserve type relationships across calls.
- Null/undefined modeled explicitly (`T | null`, `Optional[T]`, `Option<T>`, `T?`).
- Branded / nominal types for IDs and sensitive primitives where ecosystem supports it (`UserId`, `OrderId` — not raw `string`).

### 3. Compiler / Checker Configuration

- TypeScript: `"strict": true`, `"noImplicitAny": true`, `"strictNullChecks": true`, `"noUncheckedIndexedAccess": true`, `"exactOptionalPropertyTypes": true`.
- Python: `mypy --strict` or `pyright` in strict mode passes; `ruff` annotation rules enabled.
- Go: `go vet`, `staticcheck`, `errcheck` clean.
- Kotlin/Java: nullability annotations enforced; `-Werror` for unchecked / raw warnings.
- Rust: `#![deny(warnings)]` or at minimum no `dead_code`/`unused` warnings; `clippy::pedantic` reviewed.
- C#: `<Nullable>enable</Nullable>` + warnings as errors for nullability.

### 4. Boundary Handling

External input is `unknown` until validated. Never treat as target type without runtime check.

- HTTP body: validate with zod / pydantic / class-validator / DTO + validator — DO NOT cast.
- DB row: use schema-bound query layer (e.g., kysely, drizzle, sqlc, Diesel, Exposed) — DO NOT cast `any`.
- JSON parse: result is `unknown` → narrow with schema validation.
- File I/O / env vars: validated at startup; produce typed config object.

## Workflow

When writing a function:
1. Choose precise input types. No `any`. No `object`. Use named type or schema.
2. Choose precise return type. No implicit return inference for public API.
3. If accepting external/dynamic data, accept `unknown` and validate; do not accept `any`.

When importing untyped third-party code:
1. Search for official `@types/*` or community stubs at latest stable.
2. If none exist, write minimal `.d.ts` / `.pyi` / stub for the surface you actually use.
3. Do NOT widen the rest of your code to `any` to consume it.

When converting JSON / network data:
1. Define schema (zod / pydantic / struct + tag).
2. Parse → typed object.
3. Use typed object only.

When a tricky type situation arises:
1. Prefer refactor over `any`.
2. If truly necessary, use `unknown` with narrowing guard.
3. If still impossible, add `@ts-expect-error` / `# type: ignore[code]` with tracked issue ID and expiry date — never silent ignores.

## Red Flags

- `: any` anywhere in code → STOP. Replace with real type.
- `as any` / `as unknown as X` cast chain → STOP. Validate or model correctly.
- `@ts-ignore` / `# type: ignore` without explanation and ticket → STOP. Remove.
- Functions returning `Promise<any>` / `Task<any>` → STOP. Type the success value.
- Catch blocks with untyped error and unguarded `.message` access → narrow the error.
- `JSON.parse(...)` result used directly without validation → forbidden.
- Untyped `dict[str, Any]` / `map[string]interface{}` for structured payloads → forbidden.
- "I'll type it later" → never. Type it now.
- Disabling strict mode "temporarily" → forbidden. Fix the type, not the config.

## Verification

After every code change:
- Type checker passes in strictest mode (no relaxed flags).
- Linter rules for typing (e.g., `@typescript-eslint/no-explicit-any`, `no-unsafe-*`) clean.
- No new `any`, `as any`, or equivalent introduced anywhere.
- No new `@ts-ignore` / `# type: ignore` / `!!` / `dynamic` introduced.
- All new public APIs fully annotated.
- All new I/O boundaries validated against schema.
- CI fails on any reintroduction of forbidden constructs (lint rule enabled).

Code is strict-types-compliant ONLY when every value has precise type and no escape hatch is in use.
