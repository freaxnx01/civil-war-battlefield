[//]: # (Source of truth: .ai/base-instructions.md + .ai/stacks/gdscript-godot.md — update those, then regenerate by re-running /sync-ai-instructions)

# GitHub Copilot Instructions

Follow all conventions below when generating or completing code.

# AI Agent Base Instructions

Canonical, **stack-agnostic** reference for all AI coding agents. Applies to every project regardless of language or framework. Stack-specific overlays live in `.ai/stacks/<stack>.md` and are loaded alongside this file. A project loads **base + exactly one stack overlay**. Tool-specific files (`CLAUDE.md`, `.github/copilot-instructions.md`, `SKILL.md`) derive from base + the chosen stack.

> **Workflow role:** If a `WORKFLOW-ROLE.md` exists at the repo root, read it before continuing — it describes this repo's place in the personal dev workflow (implementer / consumer / workflow infrastructure). See `ai-instructions/workflows/personal-dev-workflow.md` for the workflow doc itself.
>
> **Project context:** If a `PROJECT-OVERVIEW.md` exists at the repo root, read it before continuing — it describes this repo's product/project context (name, purpose, stakeholders, vision, core customer need, key features, architecture in one paragraph). Per-feature PRDs live under `docs/specs/` or `designs/`; ADRs under `docs/adr/`.
>
> **Agent notes:** If an `AGENT-NOTES.md` exists at the repo root, read it before continuing — it holds project-specific agent-facing context that doesn't fit in the regenerated CLAUDE.md: operational gotchas, project-specific commands, repo-local workflow conventions (branch naming, PR conventions, etc.).

---

## Working Method (before any code)

Meta-rules for *how* to approach a task. Framing adapted from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills).

- **State assumptions explicitly.** If multiple interpretations exist, present them — don't pick silently.
- **Ask when unclear.** Don't hide confusion behind plausible-looking code.
- **Push back when a simpler approach exists.** Minimum code that solves the problem; nothing speculative (no unrequested flexibility, configurability, or error handling for impossible cases).
- **Surgical edits.** Every changed line must trace to the request. Don't "improve" adjacent code, comments, or formatting. Match existing style. Remove orphans *your* change created — leave pre-existing dead code alone (mention it instead).
- **Goal-driven execution.** Restate the task as a verifiable success criterion before starting. For multi-step work, write a brief numbered plan with a `verify:` check per step, then loop until each check passes.

---

## Clean Code Principles

Apply to all generated and modified code, regardless of language:

- **Small methods/functions** — each does one thing at one level of abstraction; aim for ≤20 lines
- **Guard clauses** — validate and return/throw early at the top; avoid nested `if/else` pyramids
- **Command-Query Separation** — a function either performs an action (command, returns nothing) or returns data (query), never both
- **No flag arguments** — avoid boolean parameters that switch behaviour; split into two clearly named functions instead
- **Meaningful names** — names reveal intent; no abbreviations (`cnt`, `mgr`, `svc`) except universally understood ones (`id`, `url`, `dto`)
- **One level of abstraction per function** — don't mix high-level orchestration with low-level detail; extract helpers
- **Fail fast** — detect invalid state as early as possible and throw specific errors; don't let bad data travel deep into the call stack
- **DRY** — if the same logic exists in two places, extract it; but prefer duplication over the wrong abstraction — wait until the pattern is clear before generalising
- **No dead code** — delete unreachable branches, unused parameters, and vestigial methods; git has history
- **No commented-out code blocks** — delete them, git has history

---

## Testing — TDD, Tests First, No Shortcuts

Applies to every language and framework:

1. Write the failing test first
2. Write the minimum implementation to make it pass
3. Refactor
4. **Never modify a test to make it green** — fix the implementation
5. **Never hardcode return values, mock results, or stub logic** to satisfy a test
6. **Never silently swallow exceptions** to make a test green
7. **After implementation, run the full test suite** — not just the new test
8. **If a test fails after 3 attempts, STOP** and explain what's going wrong instead of continuing to iterate
9. Test naming: `MethodName_StateUnderTest_ExpectedBehavior` (or the idiomatic equivalent for the target language)
10. E2E tests must be independent and idempotent — seed and clean up their own data

Framework-specific test project layout, mocking library choice, and assertion library live in the stack overlay.

---

## UI Development Workflow (Mandatory Phase Order)

**Never skip phases. Never write component code before wireframe approval.**

| Phase | Command | Gate |
|---|---|---|
| 1 — Brainstorm | `/ui:brainstorm` | ASCII wireframe approved |
| 2 — Flow       | `/ui:flow`       | Mermaid diagrams approved |
| 3 — Build      | `/ui:build`      | Shell → logic → interactions → polish |
| 4 — Review     | `/ui:review`     | Checklist passes |

These commands ship from the global operator console (`agent-workflow`), installed once into `~/.claude/commands/ui/` — they are **not** synced per-project. They are stack-neutral: UI component library preferences (e.g. MudBlazor, shadcn/ui, Material, Flutter widgets) are read from the active stack overlay when one is present, otherwise inferred from the existing codebase.

### What to check before writing UI code

- [ ] Does a similar component already exist in a shared folder?
- [ ] Has the ASCII wireframe been approved?
- [ ] Has the Mermaid flow been approved?
- [ ] Are you building the shell first (no business logic yet)?
- [ ] Does the component need a unit/component test?

---

## Localization (i18n) & Regional Formatting

User-facing apps support **`de` and `en`** (CI/dev tooling exempt). Regional formatting follows the **OS region**, not the UI language; `de` with an unknown region falls back to **`de-CH`**. Render via the platform localization API, never `string.Format` / `toString()`.

Full rules: [`localization.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/localization.md)

---

## Versioning (SemVer)

All projects follow [Semantic Versioning 2.0.0](https://semver.org/): `MAJOR.MINOR.PATCH` — `MAJOR` = breaking, `MINOR` = new feature (backwards-compatible), `PATCH` = bug fix.

Conventional Commits mapping: `BREAKING CHANGE:` footer or `!` after type → MAJOR; `feat` → MINOR; `fix`, `perf` → PATCH; `chore`, `docs`, `ci`, `test`, `refactor` → no bump.

- Git tags follow `v<MAJOR>.<MINOR>.<PATCH>` (e.g. `v1.3.0`) — tag on `main` after merge
- Pre-release: `v1.0.0-alpha.1`, `v1.0.0-beta.2`, `v1.0.0-rc.1`
- **git-cliff** is the changelog and release notes tool — configured via `cliff.toml`
- Where the version is declared in the project (build file, manifest, etc.) is defined by the stack overlay — but it must be declared in **exactly one place**

---

## Changelog

All projects maintain a `CHANGELOG.md` in the repo root following [Keep a Changelog](https://keepachangelog.com) conventions. **Sections per release:** `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.

- `[Unreleased]` section accumulates changes until a release is cut
- Auto-generation: **git-cliff** with `cliff.toml` configured for Conventional Commits
- CI integration: `orhun/git-cliff-action` in GitHub Actions generates release notes into GitHub Releases
- CI can validate that `[Unreleased]` is not empty before allowing a release branch

Example: [`.ai/references/base/changelog-example.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/changelog-example.md)

---

## 12-Factor App Compliance

Projects follow the [12-Factor App](https://www.12factor.net/) methodology: one repo per service, all deps declared, env-var config, attached backing services, separate build/release/run stages, stateless processes, port binding, scale via replicas not threads, fast disposability, dev/prod parity, logs to stdout, admin processes as one-offs.

Stack-specific enforcement details (logging library, migrations, etc.) live in the stack overlay.

Full per-factor table: [`.ai/references/base/12-factor.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/12-factor.md)

---

## Branching Strategy (GitHub Flow + protection rules)

```text
main              ← always deployable, protected
  └── feature/<issue-id>-short-description
  └── fix/<issue-id>-short-description
  └── chore/<short-description>
  └── release/<version>   ← only if needed for staged releases
```

- `main` requires: passing CI, at least 1 PR review, no direct push
- Branch from `main`, PR back to `main`
- Delete branch after merge
- Rebase or squash merge — no merge commits on `main`

All changes go through a PR, including docs-only ones. There is no trivial-edit exception: a direct push to a protected `main` lands before the required checks report, so they become a postmortem instead of a gate, and it leaves open PRs' branches stale.

---

## Git Worktrees

### Worktree directory

- Use **project-local** worktrees under `.worktrees/` at the repo root (hidden directory)
- `.worktrees/` must be listed in `.gitignore` — add and commit it before creating the first worktree in a repo
- Use a **random, short branch name** when the user does not specify one (e.g. `wt/<8-hex-chars>`); do not prompt for a branch name

Agent tooling that automates worktree creation should discover these rules from `CLAUDE.md` / `AGENTS.md` (e.g. a `worktree.*director` grep) and honour them without asking.

---

## Commit Messages (Conventional Commits)

```text
<type>(<scope>): <short summary>

[optional body]

[optional footer: Closes #<issue>]
```

**Types:** `feat`, `fix`, `test`, `refactor`, `chore`, `docs`, `ci`, `perf`
**Scope:** module or layer name, e.g. `orders`, `auth`, `infra`, `ui`

```text
feat(orders): add order cancellation endpoint

Implements POST /api/v1/orders/{id}/cancel.
Validates order is in Pending state before cancelling.

Closes #42
```

- Subject line: imperative mood, ≤72 chars, no period
- Body: explain *why*, not *what*
- Breaking changes: add `BREAKING CHANGE:` footer (or `!` after the type)

---

## Pull Request Conventions

### PR Title

Follow Conventional Commits format: `feat(orders): add cancellation endpoint`

### PR Description Template

Body sections: **Summary** · **Changes** · **Testing** (unit, component/integration, E2E, local) · **Checklist** (tests pass, no new vulnerable deps, no secrets, migrations included if schema changed, API/OpenAPI spec still valid).

Template: [`.ai/references/base/pr-description-template.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/pr-description-template.md)

### Review Guidelines

- PRs should be small and focused — one concern per PR
- Reviewers check: architecture adherence, test quality, security, no shortcuts that make tests green
- Auto-assign reviewers via `CODEOWNERS`

---

## CI/CD (generic outline)

Pipeline stages: `build` → `test` → `security-scan` → `container-build` → `push`

- Build and test run on every PR
- Vulnerable-dependency scan fails the build on HIGH/CRITICAL
- Container image built and pushed only on `main` after tests pass
- E2E tests run against the built image before it is marked as a release candidate

Concrete CI configuration (GitHub Actions YAML, commands, package scanners) lives in the stack overlay.

---

## Scripting

**PowerShell — customer-delivered scripts target Windows PowerShell 5.1.** Anything a customer runs (`build.ps1`, install/deploy scripts, release artifacts) must run on 5.1 unless the project documents a PS 7+ floor; `pwsh` is not installed there.

- **Never** use `??`, `??=`, ternary `? :`, `?.`, `&&` / `||` chains — *parse* errors on 5.1, so the script dies before its first line — nor `ForEach-Object -Parallel`, `Sort-Object -Stable`, `-SslProtocol`
- `$IsWindows` / `$IsLinux` / `$IsMacOS` **do not exist** on 5.1 — they are `$null`, so the branch is silently skipped. Use `$env:OS -eq 'Windows_NT'`
- Pass `-Depth` to `ConvertTo-Json` (defaults to 2, truncates silently) and `-UseBasicParsing` to the web cmdlets (a patched host prompts and hangs)
- Start with `#requires -Version 5.1`, pin encoding, verify with PSScriptAnalyzer
- **Exempt:** dev-loop tooling (`justfile` recipes) may require `pwsh`

Full rules: [`powershell-5.1.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/powershell-5.1.md)

---

## Documentation Structure

Repo-root `docs/` contains:

- `design/<feature-name>/` — UI wireframes (`wireframe.md`) & Mermaid flows (`flow.md`) per feature
- `adr/` — Architecture Decision Records
- `ai-notes/` — AI agent working notes

Rules:

- `README.md` and `CHANGELOG.md` live in the repo root
- UI design artifacts are saved per feature during the UI workflow phases
- AI agents write working notes to `docs/ai-notes/`, not `.ai/`
- `.ai/` is reserved for agent instructions and skill files only

Layout: [`.ai/references/base/documentation-structure.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/base/documentation-structure.md)

---

## Security (baseline)

- Transport security enforced (HTTPS + HSTS)
- No secrets in source files or per-environment config files — environment variables or a secrets manager only
- Validate all inputs at system boundaries before any domain logic
- Run a vulnerable-dependency scan in CI — fail the build on HIGH/CRITICAL findings
- Standard security response headers on every HTTP response

Language- and framework-specific enforcement (specific scanners, validation libraries, header mechanisms) lives in the stack overlay.

---

## Agent Guardrails

- Do not install additional packages without asking first
- Do not change the project's target runtime or framework version
- Do not modify build/project files unless the task requires it
- Do not introduce new architectural patterns unless explicitly asked
- Do not touch files outside the scope of the current task
- Keep changes minimal and focused — do not refactor unrelated code unless asked
- Never skip git hooks (`--no-verify`) unless the user explicitly asks
- Never commit secrets or credential files

Stack-specific guardrails (e.g. "do not add NuGet packages") live in the stack overlay.

---

## Project Scaffold Checklist (baseline)

Init-time checklist (every project, regardless of stack) — including baseline, .NET, and WebAPI layers — lives at [`.ai/references/scaffold-checklists.md`](https://github.com/freaxnx01/ai-instructions/blob/main/.ai/references/scaffold-checklists.md). Stack-specific additions are in the same file under their respective sections.

[//]: # (Stack overlay — loaded together with .ai/base-instructions.md for Godot/GDScript projects)

# Godot / GDScript Stack Overlay

Applies on top of `.ai/base-instructions.md` for **Godot Engine games** written in
GDScript. Use it for repos like `civil-war-battlefield` (Godot 4.x tactics game) —
a `project.godot` at the repo root, `.gd` scripts paired with `.tscn` scenes, and a
deliverable that's an exported game binary/web build, not a service.

This overlay is calibrated against a single repo so far — treat it as a solid
starting default, not exhaustive. Widen it (test conventions, export pipeline,
addon usage) as more Godot projects accumulate.

---

## Tech Stack

Godot Engine **4.x** (pin the exact version in `project.godot`'s
`config/features`) · GDScript (typed) · Godot's built-in scene/node system —
no external ECS or ORM · export via Godot's built-in export presets
(`export_presets.cfg`) · `GUT` (Godot Unit Test) for scripted tests, when tests
exist · GitHub Actions with a Godot headless export step for CI builds.

---

## Project Structure

```text
project.godot           ← engine config, autoloads, input map
scenes/                 ← one subfolder per feature/screen, .tscn + paired .gd
  <feature>/
    <feature>.tscn
    <feature>.gd
scripts/                ← non-scene singletons/managers (autoloads point here)
units/ · entities/       ← game-object scenes, one .tscn + .gd pair each
addons/                 ← third-party plugins (vendored, not hand-edited)
assets/                 ← sprites, audio, fonts
export_presets.cfg      ← export targets (committed, no secrets)
```

- **Every scene (`.tscn`) that has behaviour gets a paired script (`.gd`) of the
  same base name**, attached at the scene root — don't scatter logic across
  unrelated scripts.
- Prefer **composition via scenes** (instance a `Unit.tscn` inside `Battlefield.tscn`)
  over deep inheritance chains of scripts.
- Autoloads (singletons registered in Project Settings → Autoload) are for
  genuinely global state (game manager, event bus) — don't autoload something
  that only one scene needs.

---

## GDScript Conventions

- **Static typing everywhere it's expressible**: `var health: int = 100`,
  `func take_damage(amount: int) -> void:`. Untyped `var`/`func` is a gap to
  close, not the default.
- `class_name` on any script meant to be referenced by type elsewhere
  (`class_name Unit extends CharacterBody2D`); scripts used only as a single
  scene's root script don't need one.
- Signals over polling: a node that needs to react to another node's state
  change connects to a `signal`, it doesn't `_process()`-poll a property.
- `@onready var` for node references resolved at `_ready()`; never assume a
  child node exists before `_ready()` has run.
- Constants (`const`) for magic numbers/strings that recur (`const MAX_UNITS = 20`),
  not inline literals scattered across scripts.
- Group related exported tunables under `@export_group` so the Inspector stays
  navigable as a script grows.

---

## Input & Game Loop

- Input actions are defined in the Input Map (`project.godot`), never
  hardcoded key checks (`Input.is_action_pressed("move_up")`, not
  `Input.is_key_pressed(KEY_W)`) — this is what makes remapping and
  controller support possible later without touching gameplay code.
- Physics-affecting logic goes in `_physics_process(delta)`; visual-only /
  input-polling logic goes in `_process(delta)`. Don't move a
  `CharacterBody2D`/`RigidBody2D` from `_process`.
- AI/controller scripts (see `ai_controller.gd`-style patterns) drive the same
  input/action surface a player would — don't give AI a separate privileged
  code path into unit state.

---

## Testing

Base TDD rules (tests first, never modify a test to make it green, full suite
after implementation) apply where testable logic exists. In practice:

- **Pure logic** (damage calculation, pathing cost, turn resolution) that
  doesn't need the scene tree is the highest-value thing to unit test — extract
  it into a plain `RefCounted`/static-method class so it's testable without
  instancing a scene.
- Use **GUT** (`gut/`) for scripted tests when a project has enough pure logic
  to justify it; don't add it speculatively to a project that's still mostly
  scene wiring.
- Manual playtest is the primary gate for scene/input/game-feel behaviour that
  doesn't reduce to a pure function — this is a deliberate deviation from
  base's TDD-first mandate, same rationale as the `browser-game` stack.

Manual verification checklist before every push/release:

- [ ] Project opens in the Godot editor with no script errors in the Output panel
- [ ] The game runs (F5) and the core loop responds to input
- [ ] No warnings about missing node references / unconnected signals for
      code paths touched by the change
- [ ] Exported build (if the change touches export config) launches standalone

---

## Versioning (stack binding)

Base SemVer/Conventional-Commits/`git-cliff` rules apply. The git tag `vX.Y.Z`
on `main` is the single source of truth. If the project displays a version
in-game (menu/HUD), mirror it the same way `browser-game`'s `version.js` does —
a hand-bumped display constant that must equal the latest tag, never a second
independent version source.

---

## Build & Export

- Export presets (`export_presets.cfg`) are committed — they define target
  platforms (Windows/Linux/macOS/Web/etc.) and are not secret.
- CI export runs Godot **headless**: `godot --headless --export-release
  "<preset>" <output-path>` (or `--export-debug` for dev builds), using the
  matching Godot version pinned for the project.
- Don't commit exported binaries/build artifacts — `.gitignore` covers
  `.godot/` (editor cache/import data), export output directories, and
  platform-specific build junk.

---

## Agent Guardrails (this stack)

In addition to the base guardrails:

- Do not upgrade the Godot engine version (`config/features` in
  `project.godot`) without asking — it can silently change scene/script
  compatibility.
- Do not add a third-party addon under `addons/` without asking; treat vendored
  addon code as read-only (fix forks upstream, don't hand-patch in-tree).
- Do not switch a scene's root node type without confirming — it can break
  every script that assumes the old base class's API.
- Do not hardcode input handling (`Input.is_key_pressed`) — go through the
  Input Map action system.
- Keep AI/controller code going through the same action surface as player
  input; don't give it a shortcut into internal state.

### Never generate (this stack)

- Untyped `var`/`func` declarations where a type is knowable
- Direct key/button checks bypassing the Input Map action system
- Physics-body movement from `_process()` instead of `_physics_process()`
- A scene (`.tscn`) with real behaviour and no paired script, or vice versa
- A second, hand-rolled version display disconnected from the git tag
- Exported build artifacts or `.godot/` cache committed to git
