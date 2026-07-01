# UV

`uv` is an ultrafast Python package + project manager, written in Rust. One tool replacing pip + venv + conda + pyenv + most of Poetry.

# The problem it solves

Python has two distinct jobs that today need different tools:

1. **Package management** — installing libraries (`pip`).
2. **Project management** — declaring deps, locking versions, isolating envs, pinning the Python version (`venv` + `requirements.txt` + sometimes `conda` + `pyenv`).

Each tool has sharp edges, no shared mental model, and no real reproducibility story. uv collapses them into one.

# Why uv stands out

- **10–100× faster than pip** — `numpy + pandas + scikit-learn` installs in ~250 ms vs visible seconds.
- **Python-independent** — install via `brew` or `curl` without any Python on the machine. Then uv installs Python for you.
- **Global cache** — a library downloaded once is reused across every project. Repeat installs are near-instant.
- **One tool, one mental model** — no juggling pip / venv / pyenv / poetry.

# Installation

```bash
brew install uv                                    # macOS / Linux
pip install uv                                     # if pip already exists
curl -LsSf https://astral.sh/uv/install.sh | sh    # standalone, no Python needed
```

# The old way (what uv replaces)

```bash
python -m venv .venv
source .venv/bin/activate
pip install pandas numpy scikit-learn
pip freeze > requirements.txt
```

Three commands, two tools, no lockfile, no Python-version pinning, no reproducibility.

## The `-m` flag

Shape of the command: `<interpreter> -m <module> <args>`

- **`-m` runs a module as a script** — uses whichever interpreter you invoked.
- **Many CLI tools are just modules** — `venv`, `pip`, `http.server`, `json.tool`.
- **Pins which Python** — `python3.12 -m pip install x` installs into 3.12's site-packages; bare `pip` hides that choice.
- Try: `python3 -m http.server 8000`, `python3 -m json.tool < file.json`.

# Core commands

## Project lifecycle (the main path)

```bash
uv init uv_project           # scaffold project
cd uv_project
uv add numpy pandas scikit-learn
uv run main.py               # no `source .venv/bin/activate` needed
```

- **`uv init <name>`** — scaffolds `pyproject.toml`, `.python-version`, `main.py`, `.gitignore`, `uv.lock`. Comparable to Poetry's layout.
- **`uv add <lib>`** — installs a dep AND writes it into `pyproject.toml` + `uv.lock`.
- **`uv remove <lib>`** — uninstalls cleanly and updates both files.
- **`uv sync`** — installs exactly what `pyproject.toml` + `uv.lock` declare. The "clone repo, recreate env" command.
- **`uv lock`** — recomputes the full dep tree (transitive deps included) and refreshes `uv.lock`.
- **`uv run <script>`** — runs your script inside the project's venv. No activation needed.

## Python version management (built-in, no pyenv)

- **`uv python list`** — show installable Python versions.
- **`uv python install 3.12`** — install a Python version.
- `.python-version` file pins the version per project.

## pip-compat shim (when you don't want a full project)

```bash
uv venv                                     # drop-in for python -m venv
source .venv/bin/activate
uv pip install pandas numpy scikit-learn
uv pip freeze > requirements.txt
```

- **`uv venv`** — faster `python -m venv`.
- **`uv pip install / freeze`** — same flags as pip, ~10× faster.

## Ephemeral tools

- **`uv tool run jupyterlab`** — run a CLI tool in a one-off env, no global install.

## Workspaces

- `uv init <sub>` inside a project adds a workspace member — multi-package monorepos work out of the box.

# Mental model shifts

- **Project, not venv.** Think "which project directory am I in?", not "which venv is activated." `uv run` picks the right venv from `pyproject.toml`.
- **Declared, not installed.** `pyproject.toml` declares deps; `uv.lock` pins exact versions including transitive ones. Together they replace `requirements.txt` and make envs truly reproducible.
- **Cached, not redownloaded.** Same library across 10 projects = 1 download.
- **Python is just another dep.** uv installs and pins Python itself per project.

# Gotcha: two competing venvs

If a `.venv` from `python -m venv` is still activated when you run `uv run` inside a uv project:

```
warning: VIRTUAL_ENV=... does not match the project environment path `.venv`
and will be ignored; use `--active` to target the active environment instead
```

uv ignored the shell-activated venv and used the project's own. That's intentional — it's how uv guarantees reproducibility regardless of what your shell happens to have activated. Fix: `deactivate`, delete the stray `.venv`, let uv manage its own.
