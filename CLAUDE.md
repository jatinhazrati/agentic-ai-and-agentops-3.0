# CLAUDE.md

Guide for Claude (and future-me) working in this repo.

## What this repo is

Personal learning log for an Agentic AI + AgentOps instructor batch.
Class cadence: Saturday & Sunday, 8–11 AM IST. Owner: Jatin Hazrati.

Contents:
- Class artifacts (code, notebooks, exercises) under `Class_NN/`.
- Curated bullet notes in each `Class_NN/notes.md` (often seeded from Granola).
- Open-ended explorations under `explorations/YYYY-MM-DD-<slug>.md`.

## Layout

```
.
├── CLAUDE.md
├── README.md
├── .gitignore
├── .env.example
├── pyproject.toml        (root uv workspace)
├── uv.lock
├── Class_NN - Title/
│   ├── README.md         class hub
│   ├── notes.md          curated bullet notes
│   ├── code/             scratch scripts
│   ├── notebook.ipynb    demos
│   └── <project>/        uv workspace member (mini-projects)
└── explorations/
    ├── README.md                       index
    ├── YYYY-MM-DD-<slug>.md            notes-only exploration
    └── YYYY-MM-DD-<slug>/              exploration that grew code
        ├── README.md                   the notes
        └── code/
```

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
# Sync the whole workspace
uv sync

# Run a script in a workspace member (quote the path — titles have spaces)
uv run --project "Class_01 - Packages and Getting Started/my-first-project" python main.py

# Open a notebook
uv run jupyter lab "Class_01 - Packages and Getting Started/notebook.ipynb"

# Scaffold a new class folder (manual) — mind the quotes, titles have spaces
mkdir -p "Class_NN - Title/code"
cp "Class_01 - Packages and Getting Started/README.md" "Class_NN - Title/README.md"   # then edit
touch "Class_NN - Title/notes.md"
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
- [ ] Create `Class_NN - Title/` with `README.md`, `notes.md`, `code/`.
- [ ] Fill `Class_NN - Title/README.md` heading: date, topics planned, instructor links.

When finishing `Class_NN - Title`:
- [ ] `notes.md` exists and has the day's bullets.
- [ ] `Class_NN - Title/README.md` updated with what was actually covered.
- [ ] `git push -u origin class_NN` and open a PR.
- [ ] PR body: 3 bullets of "what I learned."
- [ ] Merge into `main` before the next class.

## Pointers

- Specs live in `docs/superpowers/specs/`.
- Plans live in `docs/superpowers/plans/`.
- This bootstrap was designed in `docs/superpowers/specs/2026-06-28-repo-bootstrap-design.md`.
