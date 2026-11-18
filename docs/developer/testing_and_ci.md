---
title: Testing and CI
---

# Testing and CI

## Running the tests locally

The tests live in `tests/` and run with pytest inside the uv environment:

```bash
uv sync --extra dev
uv run pytest
```

Tests marked `integration` start a temporary local Prefect server, so they take
a little longer. To skip them:

```bash
uv run pytest -m "not integration"
```

## Continuous integration

| Workflow | Runs on | What it does |
| --- | --- | --- |
| `tests.yml` | Every pull request, and every push to `master` (via `release.yml`) | Installs the locked dependencies (`uv sync --locked --extra dev`) and runs `pytest` |
| `pr-labels.yml` | Every pull request | Requires a release label; see [Releasing](releasing.md) |
| `release.yml` | Every push to `master` | Runs the tests, then cuts a release if one is due; see [Releasing](releasing.md) |
| `docs.yml` | Every push to `master` | Builds and deploys this documentation site |

`uv sync --locked` fails if `uv.lock` is out of date with `pyproject.toml`.
After changing dependencies, run `uv lock` and commit the updated lockfile.
