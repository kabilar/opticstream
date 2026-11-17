# OpticStream

OpticStream provides Prefect-based workflows and utilities for processing large
microscopy and imaging datasets, including LSM strips and PS-OCT tiles.

The CLI is exposed as the `opticstream` command, with a short alias `ops`.

```bash
opticstream --help      # or: ops --help
opticstream deploy ...  # or: ops deploy ...
```

## Documentation

Full documentation is available at
[opticstream.lincbrain.org](https://opticstream.lincbrain.org/).

To build and browse the documentation locally, see the
[documentation workflow](docs/developer/documentation_workflow.md) guide.

## Quick start

### 1. Install and activate an environment (uv recommended)

We recommend using [`uv`](https://docs.astral.sh/uv/) to manage the
environment for OpticStream. Python 3.11 or newer is required.

```bash
git clone https://github.com/lincbrain/opticstream.git
cd opticstream

# create and sync a uv-managed environment from pyproject.toml / uv.lock
uv sync               # add `--extra dev` for test, lint, and docs tooling

# activate the virtual environment that uv created (optional)
source .venv/bin/activate  # or prefix commands with `uv run`
```

You can also use a traditional virtual environment and `pip install -e .` if
you prefer. See the
[Installation guide](https://opticstream.lincbrain.org/getting-started/installation/)
for full details on uv, pip, and pipx based installs.

### 2. Explore the CLI

```bash
opticstream --help      # or: ops --help
opticstream deploy --help
```

### 3. Follow the full quickstart

For a complete walkthrough (including configuring scan blocks and running an
example LSM strip processing flow), see the
[Quickstart](https://opticstream.lincbrain.org/getting-started/quickstart/).

## Note about git-based dependencies

OpticStream depends on several packages that are installed directly from git
rather than from PyPI. Each is pinned to a specific commit in
`[project.dependencies]` in `pyproject.toml`:

- `linc-convert` (from `lincbrain/linc-convert`)
- `nifti-zarr` (from `calvinchai/nifti-zarr-py`)
- `psoct-toolbox` (from `calvinchai/psoct-toolbox`)

For local development against your own clones of these packages, you can
uncomment and edit the corresponding `[tool.uv.sources]` entries in
`pyproject.toml` to point at local editable paths. Do not commit
machine-specific paths (such as `path = "/home/username/..."`), since they will
not exist on other systems.

**Maintainers:** review and update these references whenever:

- You need changes from newer commits of any of these packages.
- Any of these repositories is moved, renamed, or transferred to a different
  GitHub namespace.
- The project is prepared for, or published to, PyPI. PyPI does not accept
  direct git references, so these dependencies must first be available from
  PyPI.

After changing a reference, run `uv lock` to update `uv.lock`.
