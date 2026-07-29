# Class 01: Python — orientation, setup, and learning structure

### Environment Setup: What to Install

- Avoid: Anaconda/Miniconda, Jupyter Notebooks, office laptops, pip/conda as primary tools
- Required installs:
  - Python 3.10+ (3.13 is latest stable; any 3.10+ is fine)
  - VS Code (free, industry standard from startups to Goldman Sachs)
  - UV (package and project manager, 10-200x faster than pip)
  - Terminal: Mac/Linux have one natively; Windows users need Git Bash or Windows Terminal
- Cursor or other AI editors are fine if already familiar

### Terminal and OS Guidance

- Development happens best on a Unix-based terminal
- Mac/Linux: terminal built in, run UV install command directly
- Windows: CMD and PowerShell not recommended for real dev work
  - Download Git Bash or Windows Terminal to run Linux-style commands
  - VS Code’s built-in terminal also works
  - WSL is an option but heavier; Git Bash is simpler
- Cloud/remote machines are almost always Linux: terminal fluency is essential

### Virtual Environments and UV Deep Dive

- Two roles of UV: package manager and project manager
- Package manager: uv add <package> installs into the current environment
- Virtual environments: isolated Python environments per project
  - Create: uv venv --python 3.12
  - Activate (Mac/Linux): source .venv/bin/activate
  - Activate (Windows): different command, pasted in shared link
  - Deactivate: deactivate
  - Packages installed outside are not accessible inside, and vice versa
- Project manager (uv init <project-name>): preferred modern approach
  - Creates pyproject.toml (name, version, Python version, dependencies)
  - Creates uv.lock (exact versions locked for reproducibility)
  - No need to manually activate venv; UV handles it inside the project folder
  - uv add -r requirements.txt supported for legacy workflows
  - uv sync / uv lock to sync or re-lock dependencies

### Python Fundamentals Covered

- Variables: no type declaration needed in Python (unlike Java/C++)
- f-strings: embed variables directly in print statements
- Functions: defined with def, use docstrings for AI-course context
- if __name__ == "__main__": to be covered next class
- import time / strftime: example of using standard library functions
- OOP concepts (classes, objects) flagged as prerequisites; crash course video to be shared
- Decorators noted as an advanced topic coming later

### APIs: Concept and Live Demo

- API analogy: calling a travel agent to get a flight price; code does the same to get external data
- Live demo using Frankfurter (free currency API):
  - Converted 100 USD to INR using requests library
  - Result: ~94.36 INR per USD (matched live rate)
  - Showed timeout handling, response parsing, and error handling
- LLMs (OpenAI etc.) are also called via API in the same pattern; covered next class

https://chatgpt.com/share/6a3f49e2-2130-83ee-81f2-d43b6a8a5cf0