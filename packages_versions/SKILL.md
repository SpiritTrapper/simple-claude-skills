---
name: packages-versions-latest
description: Enforce mandatory use of LATEST stable versions for all packages, dependencies, runtimes, language toolchains, and base images. Use when adding new dependencies, updating manifests (package.json, requirements.txt, pyproject.toml, go.mod, Cargo.toml, pom.xml, build.gradle, composer.json, Gemfile, Dockerfile FROM lines, .nvmrc, .python-version, etc.), reviewing dependency PRs, or onboarding a repo. Invoke whenever a dependency or version pin is added or changed.
---

# Packages & Versions — Latest Mandatory

Latest stable versions MANDATORY. Outdated dependencies forbidden unless documented blocker exists.

## Mandatory Rules

### 1. Adding New Dependency

When adding any new package:
1. Look up LATEST stable version from authoritative registry (npm, PyPI, Maven Central, crates.io, Go proxy, RubyGems, Packagist, Docker Hub, etc.).
2. Pin to that latest stable version.
3. Never copy version from old example, blog post, or memory — verify against registry.
4. Never use pre-release (`alpha`, `beta`, `rc`, `next`, `canary`) unless user explicitly requests it.

### 2. Existing Dependencies

When touching a project:
1. Check current versions against latest stable.
2. If a dependency is more than ONE major version behind latest, flag for upgrade.
3. If a dependency is more than ONE minor version behind on current major, schedule upgrade.
4. Security patches: upgrade immediately, no exceptions.

### 3. Pinning Style

- Lockfiles (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `poetry.lock`, `Cargo.lock`, `Gemfile.lock`, `go.sum`, `composer.lock`) committed.
- Manifest files use resolver-appropriate "latest minor" range matching ecosystem norms (`^x.y.z` for npm, `~=x.y.z` or `>=x.y` for Python, exact for Go modules, etc.) — but resolve to latest available at install time.
- No floating `*`, `latest`, `main`, `HEAD` tags in production manifests.

### 4. Toolchain & Runtime

Also LATEST stable:
- Node.js, Python, Go, Rust toolchain, JDK, Ruby, PHP, .NET runtime (latest LTS where ecosystem ships LTS).
- Package managers themselves (npm, pnpm, yarn, pip, poetry, uv, cargo, gradle, maven, composer, bundler).
- Linters, formatters, type checkers, test runners.
- Docker base images: pin to latest stable digest of latest minor tag (e.g., `node:22-alpine@sha256:...`).
- CI runner images / actions versions.

### 5. Verification Step (MUST RUN)

Before committing dependency changes:
- For npm: `npm outdated` clean (or all listed items have documented blockers).
- For Python: `pip list --outdated` clean; `uv pip list --outdated` clean.
- For Go: `go list -u -m all` shows no important updates pending.
- For Rust: `cargo outdated` clean.
- Run vulnerability scan: `npm audit`, `pip-audit`, `cargo audit`, `osv-scanner`, GitHub Dependabot — must show zero high/critical.

## Exceptions

Dependency MAY stay below latest ONLY when:
- Latest has documented breaking change incompatible with current code, AND
- Upgrade ticket exists with owner + deadline, AND
- Code comment / `UPGRADES.md` / `DEPENDENCIES.md` entry explains why and links ticket.

Without all three, "we'll upgrade later" not allowed — upgrade now.

## Workflow

When user says "add library X":
1. Resolve "X latest" from registry.
2. State resolved version explicitly to user before installing.
3. Install with that exact version.
4. Verify install succeeded and tests still pass.

When opening a repo:
1. Check `package.json` / equivalent and call out any majors-behind.
2. Offer to upgrade in follow-up.

When upgrading:
1. One ecosystem per PR (don't mix npm + python + docker in same PR).
2. Bump in lockfile, run full test suite, run typecheck, run linter, run e2e.
3. Update relevant code for breaking changes in same PR.
4. Document removed APIs in PR description.

## Red Flags

- "Let's use version we used in the other project" → STOP. Look up latest now.
- "This older version is fine, it works" → not a justification. Check latest.
- "Latest is too new, let me pick something stable" → latest STABLE is the rule. Don't downgrade out of fear.
- Copying version number from tutorial / blog → forbidden. Always resolve from registry.
- Pre-release/beta/RC in production → forbidden unless user explicitly requests and accepts risk.
- Unpinned `latest` tag in Docker `FROM` → forbidden. Pin to digest.
- Skipping lockfile update → forbidden.

## Verification

After every dependency change verify:
- Latest stable resolved for every added/updated package.
- Lockfile updated and committed.
- `outdated` / equivalent shows no surprises.
- Vulnerability scan clean.
- Tests + typecheck + lint pass on upgraded versions.
- Toolchain/runtime version pinned to latest stable (LTS where applicable).

Compliant ONLY when every dependency at latest stable or has documented blocker with owner and deadline.
