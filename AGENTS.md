# AORTA

Agent context for the AORTA repository (ROCm / PyTorch debugging and workload-triage toolkit).

## Project overview

- Language: Python 3.10+
- Layout: `src/aorta/` contains the package; `recipes/`, `docs/`, `scripts/`, `tests/` are at the repo root.
- Packaging: `pyproject.toml` with `setuptools_scm` (version from `vX.Y.Z` git tags), `setuptools` backend, source under `src/`.
- CLI entry point: `aorta` (registered as a project script in `pyproject.toml`).

## Setup commands

Install in editable mode with dev tooling:

```bash
uv venv
source .venv/bin/activate
uv pip install -e ".[dev]"
```

Plain `pip` also works:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[dev]"
```

GPU workloads also require PyTorch installed for the target ROCm version. AORTA intentionally does not depend on PyTorch in its package metadata.

## Run / test / lint commands

```bash
# Verify the CLI is importable and usable
aorta --help

# Run tests (CPU-safe subset; GPU/ROCm tests are marker-gated)
pytest tests/ -m "not slow and not gpu and not rocm"

# Run full test suite when on a ROCm node
pytest tests/

# Format code
black src/ tests/ scripts/
isort src/ tests/ scripts/

# Lint / static analysis
ruff check src/ tests/ scripts/
mypy src/aorta

# Install git hooks
pre-commit install
pre-commit run --all-files
```

## Key conventions

- `black` line length: 100.
- `isort` profile: `black`, line length 100.
- `ruff` selects `E`, `F`, `W`, `I`, `N`, `UP`, `B`, `C4`; ignores `E501`.
- `mypy` targets Python 3.10 with `ignore_missing_imports = true`.
- Excluded from formatting/lint: `scripts/sanitizers/download_sanitizer_artifacts.py` (vendored verbatim from upstream).
- Workloads register via the `aorta.workloads` entry-point group in `pyproject.toml`.

## Important gotchas

- The version is derived from git tags by `setuptools_scm`. An unpacked sdist without `.git` falls back to `0.0.0+unknown`.
- `llm_determinism` and `race` workloads require a distributed environment; launch them under `torchrun`.
- `aorta probe` / `aorta triage` are deprecated aliases for the unified `aorta sweep` front door.
- `aorta sweep run --recipe <yaml>` is the preferred entry point for matrix runs.
- The `_subprocess` workload is internal and cannot be invoked directly with `aorta run --workload _subprocess`.

## Useful shortcuts

```bash
# Dry-run a recipe
aorta sweep run --recipe recipes/llm-determinism/example-llm-determinism.yaml --dry-run

# Capture an environment snapshot
aorta env probe -o env.json

# List registered workloads / mitigations / environments
aorta sweep list-mitigations
aorta sweep list-environments
```
