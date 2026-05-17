# Claude Skills — Strict Engineering Principles

Six opinionated [Claude Code](https://docs.claude.com/en/docs/agents-and-tools/claude-code) skills that enforce non-negotiable engineering rules: DRY, KISS, SOLID, strict typing, no-useless-tests, and latest-package-versions.

Each skill is a single `SKILL.md` file with YAML frontmatter (`name`, `description`) plus the rules Claude must follow when the skill is active.

---

## Skills in this repo

| Folder | Skill name | Purpose |
|---|---|---|
| `dry/` | `dry-principle` | Eliminate duplicated logic, repeated literals, copy-paste, parallel structures. |
| `kiss/` | `kiss-principle` | Block over-engineering, premature abstraction, clever code, speculative generality. |
| `solid/` | `solid-principles` | Enforce SRP, OCP, LSP, ISP, DIP on every class / module / interface. |
| `strict_types/` | `strict-types` | Total typing across the codebase. Zero `any`, zero escape hatches. |
| `no_useless_tests/` | `no-useless-tests` | Mandatory unit + integration + e2e pyramid, ≥ 80% coverage, no noise tests. |
| `packages_versions/` | `packages-versions-latest` | Latest stable versions mandatory for all deps, runtimes, base images. |

---

## Install

Pick one of three scopes.

### A. Project-only (recommended for per-repo control)

Copy or symlink the skill folders into `.claude/skills/` inside your project:

```bash
cd /path/to/your/project
mkdir -p .claude/skills

# Symlink (live updates from this repo)
ln -s "./dry"               .claude/skills/dry-principle
ln -s "./kiss"              .claude/skills/kiss-principle
ln -s "./solid"             .claude/skills/solid-principles
ln -s "./strict_types"      .claude/skills/strict-types
ln -s "./no_useless_tests"  .claude/skills/no-useless-tests
ln -s "./packages_versions" .claude/skills/packages-versions-latest
```

Or copy:

```bash
cp -r "./dry"               .claude/skills/dry-principle
cp -r "./kiss"              .claude/skills/kiss-principle
cp -r "./solid"             .claude/skills/solid-principles
cp -r "./strict_types"      .claude/skills/strict-types
cp -r "./no_useless_tests"  .claude/skills/no-useless-tests
cp -r "./packages_versions" .claude/skills/packages-versions-latest
```

Folder names under `.claude/skills/` should match each skill's `name:` field. Restart Claude Code after install so the new skills are indexed.

### B. Global (every project on your machine)

Install into `~/.claude/skills/`:

```bash
mkdir -p ~/.claude/skills
for pair in \
  "dry:dry-principle" \
  "kiss:kiss-principle" \
  "solid:solid-principles" \
  "strict_types:strict-types" \
  "no_useless_tests:no-useless-tests" \
  "packages_versions:packages-versions-latest"; do
  src="${pair%%:*}"
  dst="${pair##*:}"
  ln -sf "./$src" "$HOME/.claude/skills/$dst"
done
```

Restart Claude Code.

### C. Always-on enforcement via `CLAUDE.md`

Skills only fire when Claude decides to invoke them based on their `description`. To force enforcement on every change, add this to your project's `CLAUDE.md`:

```markdown
# Engineering rules — non-negotiable

Before writing or editing any code, Claude MUST invoke the following skills and follow them strictly:

- `dry-principle` — no duplicated logic, no copy-paste
- `kiss-principle` — simplest correct solution, no over-engineering
- `solid-principles` — SRP/OCP/LSP/ISP/DIP on every class/module
- `strict-types` — no `any`, no escape hatches, fully typed
- `no-useless-tests` — unit + integration + e2e, ≥80% coverage, no noise
- `packages-versions-latest` — latest stable for every dependency

Violations of any of the above block the change. Fix at source.
```

---

## Use

### Automatic (recommended)

When the skills are installed and `CLAUDE.md` references them, Claude reads the `description:` of each skill on every turn and invokes the relevant ones via the `Skill` tool. You do not need to type anything.

### Manual invocation

You can also force a skill from chat:

```
/dry-principle review the diff
/kiss-principle look at src/services/orders.ts and simplify
/solid-principles audit the new PaymentService
/strict-types scan the repo for any `any` or `@ts-ignore`
/no-useless-tests review the test suite for noise tests
/packages-versions-latest upgrade outdated deps
```

Or ask in natural language — the description fields match common phrasings ("review for DRY violations", "is this overengineered?", "are types strict?", "upgrade deps", etc.).

### Combine

Stack multiple skills in one request:

> "Write the new `OrderService`. Follow `solid-principles`, `strict-types`, and `dry-principle`. Add tests per `no-useless-tests`."

---

## What each skill enforces (quick reference)

- **dry-principle** — single source of truth for logic, constants, types, validation rules. Refactor on second occurrence with same semantics. Coincidental duplication kept separate.
- **kiss-principle** — shortest correct path. Boring beats clever. No speculative generality. Max nesting ≤ 3, functions ≤ 30 lines, no abstractions with 1 user.
- **solid-principles** — every class has one reason to change. Extensibility via polymorphism, not switch chains. Subtypes substitutable. Interfaces role-sized. Infrastructure injected, never instantiated in business code.
- **strict-types** — strict compiler mode, no `any`/`as any`/`@ts-ignore`/`# type: ignore`/`!!`/`dynamic`. External input enters as `unknown`, validated at the boundary. Branded IDs for sensitive primitives.
- **no-useless-tests** — full pyramid mandatory, ≥80% line + branch coverage (≥95% on critical paths). Every test must lock in a behavior and catch a real regression. Snapshot dumps, tautologies, framework tests, mocking-the-SUT all deleted.
- **packages-versions-latest** — latest stable resolved from registry on add. Lockfile committed. No `latest`/`main`/`HEAD` tags in production. Docker images pinned to digest. Vulnerability scans clean.

---

## Customize

The rules in each `SKILL.md` are deliberately strict. If a project needs an exception (e.g., legacy code can't pass strict types on day one), add an override in your project's `CLAUDE.md` rather than weakening the skill file:

```markdown
## Skill overrides for this project

- `strict-types`: existing files under `legacy/` are exempt; new files MUST comply.
- `no-useless-tests`: e2e suite is in progress — unit + integration required until 2026-Q3.
```

Skills are version-controlled — edit them only when the *standard itself* should change for everyone.

---

## Verifying install

After install + restart, ask Claude:

```
list the skills you currently have available
```

You should see all six by name. If not, check:

- Folder name under `.claude/skills/` (or `~/.claude/skills/`) matches the `name:` in `SKILL.md`.
- `SKILL.md` exists at the top level of each skill folder.
- YAML frontmatter is valid (no tabs, fenced by `---`).
- Claude Code was restarted after install.

---

## Files

```
.
├── README.md                            # this file
├── dry/SKILL.md                         # name: dry-principle
├── kiss/SKILL.md                        # name: kiss-principle
├── solid/SKILL.md                       # name: solid-principles
├── strict_types/SKILL.md                # name: strict-types
├── no_useless_tests/SKILL.md            # name: no-useless-tests
└── packages_versions/SKILL.md           # name: packages-versions-latest
```
