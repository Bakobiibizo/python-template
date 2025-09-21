# Python Project Template

A production-ready Python project template with:

- Semantic versioning, changelog generation, and annotated tags.
- Unified dev CLI via `uv run dev` (lint, format, mypy, pytest, build, release helpers).
- Pre-commit hooks (Ruff, format, mypy, pytest) and Conventional Commits enforcement.
- Branching flow centered on `release-candidate` for stacking features prior to release.

Placeholders used in this template:
- `{{project_name}}` – project/repo name (e.g., on GitHub/PyPI)
- `{{package_name}}` – Python package import name (e.g., `src/{{package_name}}`)

## Getting Started (Initialize from Template)

1) Clone this repo, then run the setup script to personalize:

```bash
python tools/setup_template.py \
  --project-name my-cool-thing \
  --package-name my_cool_thing \
  --init-git \
  --remote-url https://github.com/you/my-cool-thing
```

- Or run without flags and answer prompts:

```bash
python tools/setup_template.py
```

This will:
- Replace placeholders in files and rename directories like `src/{{package_name}}` → `src/my_cool_thing`.
- Optionally initialize git, set remote `origin`, and push the default branch.

## Dev CLI

Run helper tasks via `uv` (requires Python 3.12+ and `uv`):

- `uv run dev check` – ruff check (fix locally) + ruff format + mypy + pytest -q
- `uv run dev lint` – ruff check (adds `--fix` locally)
- `uv run dev format` – ruff format
- `uv run dev typecheck` – mypy
- `uv run dev test` – pytest -q
- `uv run dev build` – uv build
- `uv run dev version current|bump <major|minor|patch>` – bump version, update CHANGELOG, tag

Release helpers:
- `uv run dev release rc` – create/push `release-candidate`
- `uv run dev branch-create [<tag>/<branch-name>]` – branch off `release-candidate` (prompts if omitted)
- `uv run dev branch-rebase` – rebase current branch onto latest `release-candidate`
- `uv run dev branch-finalize` – merge current branch into `release-candidate` and push
- `uv run dev release-pr` – open PR from `release-candidate` → `main` using latest changelog

## Conventional Commits

Format:

```
<type>(<scope>): <subject>

<body>
```

Types: feat, fix, docs, refactor, perf, test, build, ci, chore, revert

## Branching Model

- `main` – protected; releases are merged here via PR from `release-candidate`.
- `release-candidate` – staging branch to aggregate accepted feature branches.
- Feature branches – `feat/*`, `fix/*`, `docs/*`, etc.; created from `release-candidate`.

Typical flow:

1. `uv run dev branch-create feat/my-feature`
2. Commit with Conventional Commits.
3. `uv run dev branch-rebase` (keep up-to-date with RC)
4. `uv run dev branch-finalize`
5. `uv run dev release-pr` (RC → main)

## Versioning & Changelog

- SemVer in `pyproject.toml` `[project].version`.
- Bump and tag:

```bash
uv run dev version bump patch
# then
git push && git push --tags
```

## Requirements

- Python 3.12+
- `uv` package manager
- `pre-commit`
- Optional: GitHub CLI `gh` for protection/PR helpers

## License

This template is provided under the MIT license. See `LICENSE` for details.

