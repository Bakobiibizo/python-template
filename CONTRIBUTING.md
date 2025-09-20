# Contributing to {{project_name}}

Thank you for your interest in contributing! This document defines our contribution process so changes land quickly and safely.

## Quick Start

- Create a branch from `release-candidate` for your change:
  - `uv run dev branch-create feat/<short-topic>`
  - Or interactively: `uv run dev branch-create` and enter e.g. `feat/my-change`
- Make changes and commit using Conventional Commits (see below).
- Keep your branch up to date with `release-candidate` as it evolves:
  - `uv run dev branch-rebase`
- Finalize your branch into the `release-candidate` branch:
  - `uv run dev branch-finalize`
- Open the release PR from `release-candidate` to `main`:
  - `uv run dev release-pr`
- A maintainer will review and merge via PR. Releases are cut from `main` using `uv run dev version bump <part>`.

## Conventional Commits

Use the following format for commit messages:

```
<type>(<scope>): <subject>

<body>
```

- Types: `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- Subject: short imperative summary (lowercase, no period)
- Scope (optional): area touched, e.g., `scripts`, `docs`, `table`
- Body (optional): motivation, context, breaking changes

Examples:
- `feat(components): add modal component`
- `fix(django): use correct CSRF header name`
- `docs: clarify contribution flow`

Commit messages are validated by pre-commit (Commitizen).

## Branching Model

- `main` is protected and only changed via Pull Requests.
- `release-candidate` aggregates accepted feature branches prior to release.
- Feature branches: `feat/*`, `fix/*`, `docs/*`, etc.; created from `release-candidate`.

Flow:
1. Create a feature branch from `release-candidate`:
   - `uv run dev branch-create feat/<short-topic>`
2. Push commits (Conventional Commits enforced).
3. Rebase your branch on latest `release-candidate` as needed:
   - `uv run dev branch-rebase`
4. Finalize your branch into `release-candidate`:
   - `uv run dev branch-finalize`
5. Open a PR from `release-candidate` to `main`:
   - `uv run dev release-pr`
6. A maintainer reviews and merges the PR to `main`.

### Commands reference

- `uv run dev branch-create [<tag>/<branch-name>]`
  - Creates a branch from `release-candidate`; prompts if no name is provided.
  - Allowed tags: `feat`, `fix`, `docs`, `chore`, `refactor`, `perf`, `test`, `build`, `ci`.
- `uv run dev branch-rebase`
  - Rebases the current branch on top of the latest `release-candidate`.
  - On push rejection, use `git push --force-with-lease`.
- `uv run dev branch-finalize`
  - Merges the current branch into `release-candidate` (no-fast-forward) and pushes RC.
- `uv run dev release-pr`
  - Opens a PR from `release-candidate` to `main` using the latest `CHANGELOG.md` section for the body.

## Versioning & Changelog

- Semantic Versioning (SemVer) per project: `MAJOR.MINOR.PATCH`.
- Current version lives in `pyproject.toml` under `[project].version`.
- Bumping versions creates tags and prepends a changelog section based on Conventional Commits since the last tag:
  - `uv run dev version bump patch`
  - `uv run dev version bump minor`
  - `uv run dev version bump major`
- Push the release commit and tag:
  - `git push && git push --tags`

## Tooling

- Python: 3.12+
- Package manager: `uv`
- Linting/Formatting: `ruff`
- Type checking: `mypy`
- Tests: `pytest`
- Hooks: `pre-commit` (runs `uv run dev check`) and Commitizen (`commit-msg`)

Run checks locally:

```
uv run dev check
```

## Code Style

- Follow Ruff and Black-compatible formatting (enforced by `ruff format`).
- Prefer standard library and small, focused functions.
- Add type hints everywhere (`-> None` for no-return functions).

## Documentation

- Update or add docs under `docs/` when user-facing behavior changes.
- If adding components or patterns, include examples and accessibility guidance.

## Accessibility

- Ensure components are usable with keyboard only.
- Provide ARIA roles/names and manage focus where applicable.
- Announce dynamic updates via `aria-live` when needed.

## Security

- Validate server-side inputs for all state-changing requests.
- Be mindful of CSRF in adapters; include CSRF headers for HTMX requests.

## Questions

Open a discussion or issue if anything is unclear. Thank you for contributing!
