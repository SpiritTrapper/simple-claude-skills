---
name: no-useless-tests
description: Enforce mandatory full test pyramid (unit + integration + e2e), minimum 80% code coverage, and zero tolerance for useless/noise tests. Use when writing features, fixing bugs, adding modules, or touching code. Tests must verify real behavior of real units, not implementation details, not framework code, not trivialities.
---

# No Useless Tests — Mandatory Coverage with Real Value

Tests mandatory. Useless tests forbidden. Every test must justify existence by proving real behavior of real unit.

## Mandatory Rules

### 1. Test Pyramid Required

Every feature MUST have all three layers:

- **Unit tests** — isolated logic, pure functions, single classes/modules. Mocked dependencies. Fast. Hundreds-to-thousands count.
- **Integration tests** — multiple units composed, real DB, real cache, real queues (test containers), real internal services. Tens-to-hundreds count.
- **E2E tests** — full system through public entry point (HTTP, CLI, UI). Real network, real persistence. Critical user journeys only. Few-to-dozens count.

No layer may be skipped. Feature with only unit tests incomplete. Feature with only e2e tests incomplete.

### 2. Coverage Floor: ≥ 80%

- Line coverage ≥ 80% per module.
- Branch coverage ≥ 80% per module.
- Critical paths (auth, payment, data mutation, security checks): ≥ 95%.
- Coverage measured by CI. PR blocked if coverage drops.
- Coverage is floor, not goal. 80% with garbage tests still fails the "no useless tests" rule.

### 3. No Useless Tests

Test is USELESS and MUST be deleted or rewritten if it:

- Tests the framework / library, not your code (e.g., asserts `Array.push` works).
- Tests private implementation details rather than observable behavior.
- Mocks the system under test (the thing being tested).
- Always passes regardless of code change (no real assertion, or asserts a tautology).
- Re-asserts same path with no new branch or boundary.
- Tests trivial getters/setters with no logic.
- Snapshot tests nobody reads — replace with explicit assertions on important fields.
- Tests dead code paths unreachable by real inputs.
- Asserts on log messages or comments instead of behavior.
- Duplicates existing test with renamed variables.
- Flaky and "retried" without root cause — fix or delete.

### 4. Each Test Has Reason to Exist

Every test MUST answer:
- **What behavior does it lock in?** (specific input → specific output, or specific effect).
- **What regression would it catch?** (concrete change that would flip it from green to red).
- **Why this layer?** (unit if logic-only, integration if composition matters, e2e if user-visible).

If cannot answer all three for a test, delete it.

## What to Test

- Public APIs / contracts of modules
- Business rules and domain logic
- Edge cases: empty input, max input, boundary values, null/undefined/zero
- Error paths: every thrown / returned error hit
- State transitions
- Permission / authorization decisions
- Data validation
- Concurrency / race conditions where relevant
- Critical user journeys end-to-end

## What NOT to Test

- Language built-ins, framework internals
- Third-party libraries (trust them or replace them)
- Private functions directly — test through public API
- Trivial pass-through code (`return x`)
- Generated code, types, DTOs without runtime logic
- Configuration constants

## Workflow

When adding a feature:
1. Write failing unit tests for each unit of new logic.
2. Write failing integration tests for composition with DB/queue/cache/services.
3. Write failing e2e test for user-facing journey.
4. Implement until all green.
5. Run coverage. If <80%, add tests for missed branches OR delete unreachable code.
6. Review every new test against "useless tests" checklist. Delete or rewrite any that fail.

When fixing a bug:
1. Write failing test reproducing the bug at appropriate layer.
2. Fix the bug.
3. Test goes green. Test stays in suite as regression guard.

When deleting code:
1. Delete related tests.
2. Do not leave orphan tests asserting on removed behavior.

## Red Flags

- "Coverage dropped, add a quick test" → STOP. Add real test, not coverage filler.
- "Mock the function I'm testing so it returns the right value" → not a test. Delete.
- "Snapshot tests cover everything" → false. Snapshots lock in noise. Use explicit assertions for important state.
- "It's flaky, just retry it" → flaky tests are useless tests. Fix the race or the dependency.
- "It tests the test setup" → meta-tests useless.
- "E2E too slow, skip" → forbidden. Keep e2e small but present.
- "Integration too hard, only unit" → forbidden. Composition bugs hide between units.

## Verification

After every code change:
- Coverage report shows ≥ 80% line and branch coverage for changed module.
- New code has unit + integration coverage; if user-visible, also e2e.
- Every new test reviewed against useless-test checklist.
- No skipped (`xit`, `it.skip`, `@Disabled`) tests committed without tracked ticket and removal date.
- All tests pass deterministically (no retries, no random failures).

Test suite compliant ONLY when every test catches real regression and pyramid is whole.
