# CLAUDE.md

Guide for Claude (and future-me) working in this repo.

## What this repo is

Personal learning log for an Agentic AI + AgentOps instructor batch.
Class cadence: Saturday & Sunday, 8–11 AM IST. Owner: Jatin Hazrati.

Contents:
- Class artifacts (code, notebooks, exercises) under `Class_NN - Title/`.
- Curated bullet notes in each `Class_NN - Title/notes.md` (often seeded from Granola).
- Open-ended explorations under `explorations/YYYY-MM-DD-<slug>.md`.

## Layout

```
.
├── CLAUDE.md
├── README.md               overview + classes table + setup
├── .gitignore
├── .env.example            root key template (each class code/ has its own too)
├── Class_NN - Title/
│   ├── notes.md            curated bullet notes — the class hub
│   ├── excalidraw.png      hand-drawn diagram (optional)
│   ├── theory.pdf          slides / handout (optional)
│   ├── references/         extra notebooks or handouts (optional)
│   └── code/               self-contained uv project for this class
│       ├── pyproject.toml  this class's deps
│       ├── uv.lock         committed for reproducibility
│       ├── .python-version
│       ├── .env.example    keys this class needs (copy → .env; .env gitignored)
│       ├── .venv/          gitignored, per-class environment
│       ├── *.py            scripts   (script-based classes, e.g. 01–06)
│       └── main.ipynb      notebook  (notebook-based classes, e.g. 07–10)
└── explorations/
    ├── README.md                       index
    ├── YYYY-MM-DD-<slug>.md            notes-only exploration
    └── YYYY-MM-DD-<slug>/              exploration that grew code
        ├── README.md                   the notes
        └── code/
```

**No root uv workspace.** Each class's `code/` is an independent uv project — its own `pyproject.toml`, `uv.lock`, and `.venv` (gitignored). This isolates dependency sets so one class's pins can't break another (e.g. Class 07-08's `langchain-openrouter` downgrades `pydantic` without touching Class 10). uv's global cache hardlinks shared packages, so extra disk cost is negligible.

**Per-class Jupyter kernels.** Notebook classes each register a kernel named `Python (Class NN …)` pointing at that class's `.venv`. In a notebook: Select Kernel → **Jupyter Kernel** → pick the matching one. Nested `.venv`s don't reliably appear under "Python Environments", so prefer the Jupyter-Kernel list. Register with:
`( cd "Class_NN - Title/code" && uv run python -m ipykernel install --user --name class-NN-slug --display-name "Python (Class NN …)" )`

**Keys.** Copy a class's `code/.env.example` → `code/.env` and fill values before running its AI scripts/notebooks; `.env` is gitignored. Classes with no external calls (e.g. 01, 03) have no `.env.example`. Notebooks load it via `load_dotenv()`.

Folder names are `Class_NN - Title` — zero-padded number (so `Class_10` sorts after `Class_09`) plus the class title after a ` - ` separator (e.g. `Class_05 - Agents with Pure Python`). Titles use spaces; quote paths in shell/`uv` commands. Combined sessions fold into one folder (e.g. `Class_07 and 08 - Langchain Fundamentals`).
Exploration filenames start with `YYYY-MM-DD-` for chronological sort.
Explorations start as a flat `.md`; promote to a folder of the same name the moment code shows up (rename the `.md` to `README.md` inside it).

## Branch & PR workflow

- One branch per class: `class_NN` (matches folder).
- One branch per substantial exploration: `notes/<short-slug>`.
- Tiny one-page explorations may commit straight to `main`.
- One PR per class, merged into `main` before the next class starts.
- PR body = 3 bullets of "what I learned" (mini-retrospective).
- Commit messages: imperative, scoped by prefix.
  - `Class 01: add agent loop notes`
  - `explorations: MCP transport overview`
  - `chore: ...` for repo plumbing.
- No conventional-commits ceremony — this is a learning repo.

## Default mode: COACH

When Jatin asks "how do I do X":
1. Ask a clarifying question or point at relevant docs first.
2. Only reveal the full answer when asked explicitly or when he's clearly stuck.
3. Prefer Socratic prompts ("what do you think the agent needs to remember between turns?") over solutions.

Mode switches Jatin can say at any time:
- `"debug mode"` → pair-program, give full answers, move fast.
- `"summarize this"` → notes-synthesis mode (turn raw text into crisp bullets).
- `"just do it"` → execute without coaching.

When in doubt, ask which mode he wants.

## Strict rules

- NEVER commit secrets. `.env` and `.env.*` are gitignored (`.env.example` is the exception).
- NEVER commit `.venv`, `__pycache__`, `.DS_Store`, or `.ipynb_checkpoints/`.
- ALWAYS use `uv` (not bare `pip`).
- `uv.lock` IS committed — keep it that way for reproducibility.
- If asked to add a dependency, use `uv add <pkg>` (inside the right workspace member).

## Commands cheat-sheet

```bash
# Each class code/ is its own uv project — cd into it (quote: titles have spaces)
cd "Class_05 - Agents with Pure Python/code"
uv sync                         # rebuild this class's .venv from its lockfile
uv add <pkg>                    # add a dependency to THIS class only
uv run python _01_...py         # run a script
uv run streamlit run _07_...py  # run a streamlit app
cp .env.example .env            # then fill keys (only classes that need them)

# Notebook classes: Select Kernel -> Jupyter Kernel -> "Python (Class NN …)"

# Scaffold a new class (manual)
mkdir -p "Class_NN - Title/code" && touch "Class_NN - Title/notes.md"
( cd "Class_NN - Title/code" && uv init --bare --python 3.14 && uv add <deps> )
git checkout -b class_NN
```

## Notes-writing style

- Short bullets, not prose paragraphs.
- Format per concept: `**Concept** — why it matters — one example`.
- Cross-link with relative links: `[MCP basics](../explorations/2026-06-25-mcp-basics.md)`.
- One H2 per major topic; H3 for sub-topics.
- Code blocks: always tag the language (` ```python `).

## Per-class checklist

When starting `Class_NN - Title`:
- [ ] `git checkout -b class_NN` from latest `main` (branch stays plain `class_NN`, no title).
- [ ] Create `Class_NN - Title/` with `notes.md` and `code/` (add `excalidraw.png` / `references/` as they show up).
- [ ] Set up `code/` as a uv project: `uv init --bare --python 3.14` + `uv add <deps>`; add `code/.env.example` if the class needs keys; register a Jupyter kernel for notebook classes.
- [ ] Fill `Class_NN - Title/notes.md` heading: date, topics planned, instructor links.

When finishing `Class_NN - Title`:
- [ ] `notes.md` exists and has the day's bullets.
- [ ] `notes.md` updated with what was actually covered.
- [ ] `git push -u origin class_NN` and open a PR.
- [ ] Merge into `main` before the next class.